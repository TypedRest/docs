# Indexer endpoint

Endpoint that addresses child endpoints by ID.

| Method | Input | Result   | HTTP Verb | Description                                                         |
| ------ | ----- | -------- | --------- | ------------------------------------------------------------------- |
| Get    | ID    | Endpoint | -         | Get an [Element endpoint](element.md) for a specific child element. |

## Usage

=== "C#"

    ```csharp
    var users = new IndexerEndpoint<UserEndpoint>(client, "users");

    // Get a child endpoint by ID
    UserEndpoint user = users["123"];

    // Use the child endpoint
    User userData = await user.ReadAsync();
    ```

=== "Java"

    ```java
    IndexerEndpoint<UserEndpoint> users = new IndexerEndpointImpl<>(client, "users", (ref, uri) -> new UserEndpoint(ref, uri));

    // Get a child endpoint by ID
    UserEndpoint user = users.get("123");

    // Use the child endpoint
    User userData = user.read();
    ```

=== "Kotlin"

    ```kotlin
    val users = IndexerEndpointImpl(client, "users", ::UserEndpoint)

    // Get a child endpoint by ID
    val user = users["123"]

    // Use the child endpoint
    val userData = user.read()
    ```

=== "TypeScript"

    ```typescript
    const users = new IndexerEndpoint<UserEndpoint>(client, "users");

    // Get a child endpoint by ID
    const user = users.get("123");

    // Use the child endpoint
    const userData = await user.read();
    ```
