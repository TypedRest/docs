# Consumer endpoint

RPC endpoint that takes an entity as input when invoked.

| Method | Input  | Result | HTTP Verb | Description                       |
| ------ | ------ | ------ | --------- | --------------------------------- |
| Invoke | Entity | -      | `POST`    | Sends the entity to the consumer. |

## Usage

=== "C#"

    ```csharp
    var logger = new ConsumerEndpoint<LogEntry>(client, "log");

    // Invoke with input
    await logger.InvokeAsync(new LogEntry { Message = "Hello, world!", Level = "Info" });
    ```

=== "Java"

    ```java
    ConsumerEndpoint<LogEntry> logger = new ConsumerEndpoint<>(client, "log", LogEntry.class);

    // Invoke with input
    logger.invoke(new LogEntry("Hello, world!", "Info"));
    ```

=== "Kotlin"

    ```kotlin
    val logger = ConsumerEndpoint(client, "log", LogEntry::class.java)

    // Invoke with input
    logger.invoke(LogEntry("Hello, world!", "Info"))
    ```

=== "TypeScript"

    ```typescript
    const logger = new ConsumerEndpoint<LogEntry>(client, "log");

    // Invoke with input
    await logger.invoke({ message: "Hello, world!", level: "Info" });
    ```
