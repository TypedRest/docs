# Producer endpoint

RPC endpoint that returns an entity as output when invoked.

| Method | Input | Result | HTTP Verb | Description                       |
| ------ | ----- | ------ | --------- | --------------------------------- |
| Invoke | -     | Entity | `POST`    | Gets an entity from the producer. |

## Usage

=== "C#"

    ```csharp
    var generator = new ProducerEndpoint<Token>(client, "generate-token");

    // Invoke and get output
    Token token = await generator.InvokeAsync();
    ```

=== "Java"

    ```java
    ProducerEndpoint<Token> generator = new ProducerEndpoint<>(client, "generate-token", Token.class);

    // Invoke and get output
    Token token = generator.invoke();
    ```

=== "Kotlin"

    ```kotlin
    val generator = ProducerEndpoint(client, "generate-token", Token::class.java)

    // Invoke and get output
    val token = generator.invoke()
    ```

=== "TypeScript"

    ```typescript
    const generator = new ProducerEndpoint<Token>(client, "generate-token");

    // Invoke and get output
    const token = await generator.invoke();
    ```
