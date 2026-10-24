# Paging

Reading all ten thousand contacts with a single `GET /contacts` is slow and memory-hungry. Many APIs let clients fetch a collection in pages instead. TypedRest uses the standard HTTP mechanism for this, [range requests](https://developer.mozilla.org/docs/Web/HTTP/Range_requests), applied to elements of a collection instead of bytes of a file.

## The protocol

The client asks for a range of elements with a `Range` header. Positions start at 0 and both ends are inclusive:

```
GET /contacts
Range: elements=0-49
```

The server responds with `206 Partial Content`, the requested elements, and a `Content-Range` header saying which elements these are and how many there are in total:

```
HTTP/1.1 206 Partial Content
Content-Range: elements 0-49/1234
Content-Type: application/json

[ ...50 contacts... ]
```

If the start of the requested range lies beyond the end of the collection, the server responds with `416 Range Not Satisfiable`.

To tell clients that it supports this, the server includes `Accept-Ranges: elements` in its responses for the collection.

## Reading one page

[Collection endpoints](../endpoints/generic/collection.md) have a read range method that returns a `PartialResponse`. It contains the elements, the range they came from, and whether the end of the collection was reached:

=== "C#"

    ```csharp
    PartialResponse<Contact> page = await client.Contacts.ReadRangeAsync(new RangeItemHeaderValue(0, 49));

    foreach (var contact in page.Elements)
        Console.WriteLine(contact.Name);

    Console.WriteLine($"Elements {page.Range?.From}-{page.Range?.To} of {page.Range?.Length}");
    bool more = !page.EndReached;
    ```

=== "Java"

    ```java
    PartialResponse<Contact> page = client.getContacts().readRange(0L, 49L);

    for (Contact contact : page.getElements())
        System.out.println(contact.getName());

    HttpContentRangeHeader range = page.getRange();
    System.out.println("Elements " + range.getFrom() + "-" + range.getTo() + " of " + range.getLength());
    boolean more = !page.getEndReached();
    ```

=== "Kotlin"

    ```kotlin
    val page = client.contacts.readRange(0, 49)

    for (contact in page.elements)
        println(contact.name)

    println("Elements ${page.range?.from}-${page.range?.to} of ${page.range?.length}")
    val more = !page.endReached
    ```

=== "TypeScript"

    ```typescript
    const page = await client.contacts.readRange(0, 49);

    for (const contact of page.elements)
        console.log(contact.name);

    console.log(`Elements ${page.range?.from}-${page.range?.to} of ${page.range?.length}`);
    const more = !page.endReached;
    ```

`EndReached` is true when the last element of the page is the last element of the collection. It is also true when the server ignored the `Range` header and sent the whole collection with `200 OK`, so code that pages through a collection works with servers that do not support ranges too.

## Reading all pages

To process a large collection one page at a time, keep asking for the range after the last one you received. Wrapping this in a function that returns a lazy sequence keeps the paging out of the calling code:

=== "C#"

    ```csharp
    // In a static class
    public static async IAsyncEnumerable<TEntity> ReadAllPagedAsync<TEntity, TElementEndpoint>(
        this ICollectionEndpoint<TEntity, TElementEndpoint> collection,
        long pageSize = 50,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
        where TEntity : class
        where TElementEndpoint : IElementEndpoint<TEntity>
    {
        long from = 0;
        while (true)
        {
            var page = await collection.ReadRangeAsync(new RangeItemHeaderValue(from, from + pageSize - 1), cancellationToken);
            foreach (var entity in page.Elements)
                yield return entity;

            if (page.EndReached) yield break;
            from = page.Range!.To!.Value + 1;
        }
    }
    ```

    ```csharp
    await foreach (var contact in client.Contacts.ReadAllPagedAsync())
        Console.WriteLine(contact.Name);
    ```

=== "Java"

    ```java
    public static <T> void forEachPaged(GenericCollectionEndpoint<T, ?> collection, long pageSize, Consumer<T> action) {
        long from = 0;
        while (true) {
            PartialResponse<T> page = collection.readRange(from, from + pageSize - 1);
            page.getElements().forEach(action);

            if (page.getEndReached()) return;
            from = page.getRange().getTo() + 1;
        }
    }
    ```

    ```java
    forEachPaged(client.getContacts(), 50, contact -> System.out.println(contact.getName()));
    ```

=== "Kotlin"

    ```kotlin
    fun <T> GenericCollectionEndpoint<T, *>.readAllPaged(pageSize: Long = 50): Sequence<T> = sequence {
        var from = 0L
        while (true) {
            val page = readRange(from, from + pageSize - 1)
            yieldAll(page.elements)

            if (page.endReached) break
            from = page.range!!.to!! + 1
        }
    }
    ```

    ```kotlin
    for (contact in client.contacts.readAllPaged())
        println(contact.name)
    ```

=== "TypeScript"

    ```typescript
    import { GenericCollectionEndpoint, ElementEndpoint } from "typedrest/endpoints/generic";

    export async function* readAllPaged<T, E extends ElementEndpoint<T>>(collection: GenericCollectionEndpoint<T, E>, pageSize = 50) {
        let from = 0;
        while (true) {
            const page = await collection.readRange(from, from + pageSize - 1);
            yield* page.elements;

            if (page.endReached) return;
            from = page.range!.to! + 1;
        }
    }
    ```

    ```typescript
    for await (const contact of readAllPaged(client.contacts))
        console.log(contact.name);
    ```

The next page starts after the last element the server actually returned, not after the last one requested. Servers may return smaller pages than asked for, for example to enforce a maximum page size, and this way nothing is skipped.

Pages are fetched only as the loop needs them. Stopping the loop early stops fetching.

!!! warning "Collections that change while you page"
    Range requests address elements by position. If a contact is inserted or deleted before the current position while you are paging, later pages shift and you may see an element twice or miss one. This is fine for append-only collections and for display purposes. If you need a consistent snapshot, ask for the whole collection in one request.

## When the server does not send a total

A server may answer with `Content-Range: elements 0-49/*` if counting all elements is expensive. Then TypedRest cannot tell from the response whether this was the last page, and `EndReached` stays false. The loop above then requests one range past the end and gets `416 Range Not Satisfiable`. Treat that as the end of the collection:

| .NET                        | Java/Kotlin         | TypeScript                 |
| --------------------------- | ------------------- | -------------------------- |
| `InvalidOperationException` | `ConflictException` | `RangeNotSatisfiableError` |

=== "C#"

    ```csharp
    PartialResponse<TEntity> page;
    try
    {
        page = await collection.ReadRangeAsync(new RangeItemHeaderValue(from, from + pageSize - 1), cancellationToken);
    }
    catch (InvalidOperationException) when (from > 0)
    {
        yield break;
    }
    ```

=== "Java"

    ```java
    PartialResponse<T> page;
    try {
        page = collection.readRange(from, from + pageSize - 1);
    } catch (ConflictException ex) {
        if (from > 0 && ex.getStatus() == HttpStatusCode.RangeNotSatisfiable) return;
        throw ex;
    }
    ```

=== "Kotlin"

    ```kotlin
    val page = try {
        readRange(from, from + pageSize - 1)
    } catch (ex: ConflictException) {
        if (from > 0 && ex.status == HttpStatusCode.RangeNotSatisfiable) break
        throw ex
    }
    ```

=== "TypeScript"

    ```typescript
    import { RangeNotSatisfiableError } from "typedrest/errors";

    let page;
    try {
        page = await collection.readRange(from, from + pageSize - 1);
    } catch (err) {
        if (from > 0 && err instanceof RangeNotSatisfiableError) return;
        throw err;
    }
    ```

A `416` on the very first page means the collection is empty, which is why the examples only treat it as the end when `from > 0`. Handle that case too if empty collections are possible.

## The last few elements

Leave out the start of the range to get the last elements of the collection instead, for example the ten most recently added contacts:

=== "C#"

    ```csharp
    var latest = await client.Contacts.ReadRangeAsync(new RangeItemHeaderValue(null, 10));
    ```

=== "Java"

    ```java
    PartialResponse<Contact> latest = client.getContacts().readRange(null, 10L);
    ```

=== "Kotlin"

    ```kotlin
    val latest = client.contacts.readRange(null, 10)
    ```

=== "TypeScript"

    ```typescript
    const latest = await client.contacts.readRange(undefined, 10);
    ```

This sends `Range: elements=-10`. Likewise, leaving out the end (`elements=100-`) asks for everything from position 100 on.

## Checking for support

After any request to the collection, the read range allowed property (`ReadRangeAllowed` in .NET, `readRangeAllowed` in Java, Kotlin and TypeScript) tells you whether the server sent `Accept-Ranges: elements`. It is `null`/`undefined` before the first request.

If the server uses a different unit name in its `Range` headers, such as `items`, change the endpoint's range unit (`RangeUnit` in .NET, `rangeUnit` in Java, Kotlin and TypeScript) before sending requests.

## Next steps

- [Live updates](live-updates.md): streaming collection endpoints use the same range requests to follow a collection as it grows.
- [Command-line tool](command-line.md): `addressbook contacts 0-49` reads a range from the command line.
