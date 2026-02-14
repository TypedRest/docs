# Function endpoint

RPC endpoint that takes an entity as input and returns another entity as output when invoked.

| Method | Input  | Result | HTTP Verb | Description           |
| ------ | ------ | ------ | --------- | --------------------- |
| Invoke | Entity | Entity      | `POST`    | Invokes the function. |

## Usage

=== "C#"

    ```csharp
    var calculator = new FunctionEndpoint<Calculation, Result>(client, "calculate");

    // Invoke with input and get output
    Result result = await calculator.InvokeAsync(new Calculation { X = 10, Y = 5, Op = "add" });
    ```

=== "Java"

    ```java
    FunctionEndpoint<Calculation, Result> calculator = new FunctionEndpoint<>(client, "calculate", Calculation.class, Result.class);

    // Invoke with input and get output
    Result result = calculator.invoke(new Calculation(10, 5, "add"));
    ```

=== "Kotlin"

    ```kotlin
    val calculator = FunctionEndpoint(client, "calculate", Calculation::class.java, Result::class.java)

    // Invoke with input and get output
    val result = calculator.invoke(Calculation(10, 5, "add"))
    ```

=== "TypeScript"

    ```typescript
    const calculator = new FunctionEndpoint<Calculation, Result>(client, "calculate");

    // Invoke with input and get output
    const result = await calculator.invoke({ x: 10, y: 5, op: "add" });
    ```
