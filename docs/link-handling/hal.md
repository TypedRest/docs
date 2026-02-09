# HAL (Hypertext Application Language)

TypedRest supports extracting links from response bodies using the [HAL specification](https://datatracker.ietf.org/doc/html/draft-kelly-json-hal). HAL provides a consistent format for embedding links and embedded resources within JSON responses.

## HAL format

HAL adds a `_links` object to JSON responses containing links organized by relation type:

```json
{
    "id": 123,
    "name": "John Doe",
    "_links": {
        "self": { "href": "/users/123" },
        "orders": { "href":  "/users/123/orders" },
        "search": { "href":  "/users/{id}", "templated": true }
    }
}
```

## Content type

TypedRest recognizes HAL responses by the `application/hal+json` content type:

When this content type is present, TypedRest automatically parses the `_links` object and makes the links available through the standard link resolution methods.

## Using HAL links

After receiving a HAL response, you can resolve links just like with HTTP Link headers:

=== "C#"

    ```csharp
    await endpoint.ReadAsync();

    // Resolve a single link
    Uri ordersUri = endpoint.Link("orders");

    // Resolve a templated link
    Uri userUri = endpoint.LinkTemplate("search", new { id = "456" });
    ```

=== "Java"

    ```java
    endpoint.read();

    // Resolve a single link
    URI ordersUri = endpoint.link("orders");

    // Resolve a templated link
    URI userUri = endpoint.linkTemplate("search", Map.of("id", "456"));
    ```

=== "Kotlin"

    ```kotlin
    endpoint.read()

    // Resolve a single link
    val ordersUri = endpoint.link("orders")

    // Resolve a templated link
    val userUri = endpoint.linkTemplate("search", mapOf("id" to "456"))
    ```

=== "TypeScript"

    ```typescript
    await endpoint.read();

    // Resolve a single link
    const ordersUri = endpoint.link("orders");

    // Resolve a templated link
    const userUri = endpoint.linkTemplate("search", { id: "456" });
    ```

## Multiple links

HAL supports multiple links for the same relation type using an array:

```json
{
    "_links": {
        "item": [
            { "href": "/items/1", "title": "First Item" },
            { "href": "/items/2", "title": "Second Item" }
        ]
    }
}
```

Retrieve all links with `GetLinks`:

=== "C#"

    ```csharp
    var items = endpoint.GetLinks("item");
    foreach (var (uri, title) in items)
    {
        Console.WriteLine($"{title}: {uri}");
    }
    ```

=== "Java"

    ```java
    List<Link> items = endpoint.getLinks("item");
    for (Link link : items) {
        System.out.println(link.getTitle() + ": " + link.getUri());
    }
    ```

=== "Kotlin"

    ```kotlin
    val items = endpoint.getLinks("item")
    for (link in items) {
        println("${link.title}: ${link.uri}")
    }
    ```

=== "TypeScript"

    ```typescript
    const items = endpoint.getLinks("item");
    for (const { uri, title } of items) {
        console.log(`${title}: ${uri}`);
    }
    ```

## Templated links

HAL links can be marked as [templates](uri-templates.md) with the `templated` property:

```json
{
    "_links": {
        "find": {
            "href": "/users{?name,email}",
            "templated": true
        }
    }
}
```

=== "C#"

    ```csharp
    var findUri = endpoint.LinkTemplate("find", new { name = "John", email = "john@example.com" });
    // Result: /users?name=John&email=john%40example.com
    ```

=== "Java"

    ```java
    URI findUri = endpoint.linkTemplate("find", Map.of("name", "John", "email", "john@example.com"));
    // Result: /users?name=John&email=john%40example.com
    ```

=== "Kotlin"

    ```kotlin
    val findUri = endpoint.linkTemplate("find", mapOf("name" to "John", "email" to "john@example.com"))
    // Result: /users?name=John&email=john%40example.com
    ```

=== "TypeScript"

    ```typescript
    const findUri = endpoint.linkTemplate("find", { name: "John", email: "john@example.com" });
    // Result: /users? name=John&email=john%40example.com
    ```
