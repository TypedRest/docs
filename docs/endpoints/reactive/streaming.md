# Streaming endpoint

Endpoint for a stream of entities using a persistent HTTP connection. The entities the server writes into the open response body are split on a configurable separator, `\n` by default. If the server speaks the Server-Sent Events protocol instead, use the [SSE endpoint](sse.md).

!!! note
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.  
    For TypeScript, reactive endpoints are part of the main `typedrest` package and provide `AsyncIterable`s via `stream()` instead of observables.

| Method     | Input | Result        | HTTP Verb | Description                    |
| ---------- | ----- | ------------- | --------- | ------------------------------ |
| Get stream | -     | Entity stream | `GET`     | Provides a stream of entities. |

## Usage

=== "C#"

    ```csharp
    var events = new StreamingEndpoint<Event>(client, "events");

    // Subscribe to the stream
    IObservable<Event> stream = events.GetObservable();
    stream.Subscribe(x => Console.WriteLine($"Event: {x.Type} - {x.Message}"));
    ```

=== "Java"

    ```java
    StreamingEndpoint<Event> events = new StreamingEndpointImpl<>(client, "events", Event.class);

    // Subscribe to the stream
    Observable<Event> stream = events.getObservable();
    stream.subscribe(x -> System.out.println("Event: " + x.getType() + " - " + x.getMessage()));
    ```

=== "Kotlin"

    ```kotlin
    val events = StreamingEndpointImpl(client, "events", Event::class.java)

    // Subscribe to the stream
    val stream = events.getObservable()
    stream.subscribe { x -> println("Event: ${x.type} - ${x.message}") }
    ```

=== "TypeScript"

    ```typescript
    const events = new StreamingEndpoint<Event>(client, "events");

    // Consume the stream
    for await (const x of events.stream()) {
        console.log(`Event: ${x.type} - ${x.message}`);
    }
    ```

    Pass a different separator as the third constructor argument if the server does not delimit entities with `\n`:

    ```typescript
    const events = new StreamingEndpoint<Event>(client, "events", "\r\n");
    ```
