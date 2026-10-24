# Optimistic concurrency

Two people open the same contact. Alice changes the email address and saves. A moment later Bob, still looking at the old version, fixes a typo in the name and saves too. Bob's request replaces the whole contact, so Alice's new email address is silently lost.

HTTP solves this "lost update" problem with entity tags (ETags), and TypedRest uses them automatically if the server provides them. This tutorial shows what TypedRest does, how to handle conflicts, and what the server needs to do.

## How it works

1. When an [element endpoint](../endpoints/generic/element.md) reads an entity, the server includes an `ETag` header identifying that version of it, such as `"v7"`. The endpoint keeps the response along with the ETag.
2. When the same endpoint later sets (`PUT`) or deletes the entity, it sends `If-Match: "v7"`, meaning "only do this if the entity is still at version v7".
3. If someone else changed the entity in the meantime, the server answers `412 Precondition Failed` instead of applying the change, and TypedRest throws an exception.

The same ETag also saves bandwidth: reading the entity again with the same endpoint sends `If-None-Match: "v7"`, and if nothing changed the server answers `304 Not Modified` without a body. TypedRest then returns the entity it kept from before.

```
GET /contacts/1337                                →  200 OK, ETag: "v7"
PUT /contacts/1337, If-Match: "v7"  (Alice)       →  204 No Content
PUT /contacts/1337, If-Match: "v7"  (Bob)         →  412 Precondition Failed
```

## Detecting conflicts

The exception for `412 Precondition Failed` is:

| .NET                        | Java/Kotlin         | TypeScript         |
| --------------------------- | ------------------- | ------------------ |
| `InvalidOperationException` | `ConflictException` | `ConcurrencyError` |

The example below plays both Alice and Bob, using two endpoint objects for the same contact:

=== "C#"

    ```csharp
    var alice = client.Contacts["1337"];
    var bob = client.Contacts["1337"];

    var aliceCopy = await alice.ReadAsync(); // ETag "v7"
    var bobCopy = await bob.ReadAsync();     // ETag "v7"

    aliceCopy.Email = "smith@example.com";
    await alice.SetAsync(aliceCopy);         // If-Match: "v7", succeeds

    bobCopy.Name = "Smith";
    try
    {
        await bob.SetAsync(bobCopy);         // If-Match: "v7", fails
    }
    catch (InvalidOperationException)
    {
        Console.WriteLine("Someone else changed this contact. Please review their changes.");
    }
    ```

=== "Java"

    ```java
    ContactEndpoint alice = client.getContacts().get("1337");
    ContactEndpoint bob = client.getContacts().get("1337");

    Contact aliceCopy = alice.read(); // ETag "v7"
    Contact bobCopy = bob.read();     // ETag "v7"

    aliceCopy.setEmail("smith@example.com");
    alice.set(aliceCopy);             // If-Match: "v7", succeeds

    bobCopy.setName("Smith");
    try {
        bob.set(bobCopy);             // If-Match: "v7", fails
    } catch (ConflictException ex) {
        System.out.println("Someone else changed this contact. Please review their changes.");
    }
    ```

=== "Kotlin"

    ```kotlin
    val alice = client.contacts["1337"]
    val bob = client.contacts["1337"]

    val aliceCopy = alice.read() // ETag "v7"
    val bobCopy = bob.read()     // ETag "v7"

    alice.set(aliceCopy.copy(email = "smith@example.com")) // If-Match: "v7", succeeds

    try {
        bob.set(bobCopy.copy(name = "Smith"))              // If-Match: "v7", fails
    } catch (ex: ConflictException) {
        println("Someone else changed this contact. Please review their changes.")
    }
    ```

=== "TypeScript"

    ```typescript
    import { ConcurrencyError } from "typedrest/errors";

    const alice = client.contacts.get("1337");
    const bob = client.contacts.get("1337");

    const aliceCopy = await alice.read(); // ETag "v7"
    const bobCopy = await bob.read();     // ETag "v7"

    aliceCopy.email = "smith@example.com";
    await alice.set(aliceCopy);           // If-Match: "v7", succeeds

    bobCopy.name = "Smith";
    try {
        await bob.set(bobCopy);           // If-Match: "v7", fails
    } catch (err) {
        if (err instanceof ConcurrencyError) console.log("Someone else changed this contact. Please review their changes.");
        else throw err;
    }
    ```

In .NET and Java/Kotlin the same exception type is also used for `409 Conflict`. In Java/Kotlin you can tell them apart by the exception's `status`. In .NET, derive from `DefaultErrorHandler` and override `MapException` if you need a separate exception type for `412`, then pass your handler to the entry endpoint's constructor.

!!! warning "Keep the endpoint object"
    The ETag is stored in the endpoint object that did the read. This sends **no** `If-Match`:

    ```csharp
    var contact = await client.Contacts["1337"].ReadAsync();
    contact.Name = "Smith";
    await client.Contacts["1337"].SetAsync(contact); // a new endpoint object, which has never read anything
    ```

    Keep the endpoint in a variable for the whole read-modify-write cycle, as in the examples above.

    Every set or delete clears the stored response before sending its request. After catching the `412` above, `bob` therefore has no ETag left: calling `bob.SetAsync(bobCopy)` again would send no `If-Match` and overwrite Alice's change after all. Always read again before the next set, or use `update` as described below.

## Retrying automatically

Often the right reaction to a conflict is simply to start over: read the current version, apply the same change to it, and try to save again. The `update` method does exactly that, up to a maximum number of retries (3 by default):

=== "C#"

    ```csharp
    await client.Contacts["1337"].UpdateAsync(contact => contact.Name = "Smith");
    ```

    There are overloads taking a `Func<Contact, Contact>` and a maximum number of retries. If the server sends a `Retry-After` header with the `412`, TypedRest waits that long before retrying.

=== "Java"

    ```java
    client.getContacts().get("1337").update(contact -> {
        contact.setName("Smith");
        return contact;
    }, 3);
    ```

    If the server sends a `Retry-After` header with the `412`, TypedRest waits that long before retrying.

=== "Kotlin"

    ```kotlin
    client.contacts["1337"].update({ it.copy(name = "Smith") })
    ```

    If the server sends a `Retry-After` header with the `412`, TypedRest waits that long before retrying.

=== "TypeScript"

    ```typescript
    await client.contacts.get("1337").update(contact => { contact.name = "Smith"; });
    ```

    TypedRest for TypeScript retries immediately, without waiting for a `Retry-After` delay.

The change you pass in may run more than once, so it must only depend on the entity it receives, not on state that changes between calls.

If all retries fail, the last exception is thrown.

### JSON Patch (.NET only)

.NET also offers an `UpdateAsync` overload that takes a [JSON Patch](https://jsonpatch.com/) document. It sends only the change, as a `PATCH` request with the content type `application/json-patch+json`, so there is no stale copy that could overwrite someone else's change:

```csharp
await client.Contacts["1337"].UpdateAsync(patch => patch.Replace(x => x.Name, "Smith"));
```

If the server does not support JSON Patch and answers `404 Not Found` or `405 Method Not Allowed`, TypedRest falls back to the read-modify-write cycle.

## Merge is not protected

`merge` (`PATCH` with a partial entity) sends no `If-Match`. It does not need to, as long as the server only changes the fields present in the request. Use it for changes that should win no matter what else happened to the entity, and set or update for changes that depend on the version you read.

## What the server needs to do

TypedRest only sends conditional requests if the server cooperates:

- **Send an `ETag` header** on `GET` responses for the elements you want to protect. Any string that changes whenever the entity changes will do: a version number, a timestamp or a hash of the content.
- **Check `If-Match`** on `PUT` and `DELETE`. If the header is present and does not match the current ETag, respond with `412 Precondition Failed` and change nothing.
- **Check `If-None-Match`** on `GET` and respond with `304 Not Modified` if it matches. This is optional; it only saves bandwidth.
- **Do not send `Cache-Control: no-store`** on those responses. TypedRest for .NET and Java/Kotlin does not keep responses marked like this, so it would have no ETag to send.

In .NET and Java/Kotlin, a `Last-Modified` header works as a fallback when there is no `ETag`: TypedRest then sends `If-Unmodified-Since` and `If-Modified-Since` instead. Timestamps have a resolution of one second, so prefer ETags if changes can happen faster than that. TypedRest for TypeScript only supports ETags.

## Next steps

- [Testing your client](testing.md): simulate a `412` to test your conflict handling.
- [Command-line tool](command-line.md): the exit code for conflicts.
