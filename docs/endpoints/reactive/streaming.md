# Streaming endpoint

Endpoint for a stream of entities using a persistent HTTP connection.

| Method         | Input | Result        | HTTP Verb | Description                                |
| -------------- | ----- | ------------- | --------- | ------------------------------------------ |
| Get observable | -     | Entity stream | `GET`     | Provides an observable stream of entities. |

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
