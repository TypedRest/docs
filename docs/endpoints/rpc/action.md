# Action endpoint

RPC endpoint that is invoked with no input or output.

| Method | Input | Result | HTTP Verb | Description         |
| ------ | ----- | ------ | --------- | ------------------- |
| Invoke | -     | -      | `POST`    | Invokes the action. |

## Usage

=== "C#"

    ```csharp
    var refresh = new ActionEndpoint(client, "refresh");

    // Invoke the action
    await refresh.InvokeAsync();
    ```

=== "Java"

    ```java
    ActionEndpoint refresh = new ActionEndpoint(client, "refresh");

    // Invoke the action
    refresh.invoke();
    ```

=== "Kotlin"

    ```kotlin
    val refresh = ActionEndpoint(client, "refresh")

    // Invoke the action
    refresh.invoke()
    ```

=== "TypeScript"

    ```typescript
    const refresh = new ActionEndpoint(client, "refresh");

    // Invoke the action
    await refresh.invoke();
    ```
