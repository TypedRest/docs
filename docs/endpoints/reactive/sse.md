# SSE endpoint

Endpoint for a stream of entities using [Server-Sent Events](https://developer.mozilla.org/docs/Web/API/Server-sent_events) (SSE).

!!! note
    Reactive endpoints are not available for TypeScript.  
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.

| Method         | Input | Result        | HTTP Verb | Description                                |
| -------------- | ----- | ------------- | --------- | ------------------------------------------ |
| Get observable | -     | Entity stream | `GET`     | Provides an observable stream of entities. |

The request is sent with `Accept: text/event-stream`. The `data:` field of each event is deserialized as an entity. Unlike the [Streaming endpoint](streaming.md), which simply splits the response body on a separator, this understands the SSE wire format, so it can also make use of the event type, id and reconnection interval the server sends along.

## Usage

=== "C#"

    ```csharp
    var events = new SseStreamingEndpoint<Event>(client, "events");

    // Subscribe to the stream
    IObservable<Event> stream = events.GetObservable();
    stream.Subscribe(x => Console.WriteLine($"Event: {x.Type} - {x.Message}"));
    ```

=== "Java"

    ```java
    StreamingEndpoint<Event> events = new SseStreamingEndpointImpl<>(client, "events", Event.class);

    // Subscribe to the stream
    Observable<Event> stream = events.getObservable();
    stream.subscribe(x -> System.out.println("Event: " + x.getType() + " - " + x.getMessage()));
    ```

=== "Kotlin"

    ```kotlin
    val events = SseStreamingEndpointImpl(client, "events", Event::class.java)

    // Subscribe to the stream
    val stream = events.getObservable()
    stream.subscribe { x -> println("Event: ${x.type} - ${x.message}") }
    ```

## Event types

A server can label events with an `event:` field to send more than one kind of message over a single stream. Pass an event type to the constructor to receive only those events; all others are ignored. Leave it unset to receive every event, whatever its type.

=== "C#"

    ```csharp
    var updates = new SseStreamingEndpoint<Job>(client, "jobs/events", eventType: "update");
    ```

=== "Java"

    ```java
    StreamingEndpoint<Job> updates = new SseStreamingEndpointImpl<>(client, "jobs/events", Job.class, "update");
    ```

=== "Kotlin"

    ```kotlin
    val updates = SseStreamingEndpointImpl(client, "jobs/events", Job::class.java, eventType = "update")
    ```

## Reconnecting

Long-lived streams can get interrupted. By default the endpoint reconnects on its own after a dropped connection, a transient transport error or a `5xx` response, without the subscriber seeing anything.

Two SSE fields steer this. A `retry:` field tells the client how long to wait before reconnecting; until the server sends one, the endpoint waits three seconds. An `id:` field is remembered and sent back as a `Last-Event-ID` header on the next attempt, so a server that supports it can resume where the stream left off rather than starting over.

To end a stream for good, respond with `204 No Content`. The observable then completes instead of reconnecting.

Turn reconnecting off to have connection failures surface as errors on the observable instead:

=== "C#"

    ```csharp
    var events = new SseStreamingEndpoint<Event>(client, "events")
    {
        AutoReconnect = false
    };
    ```

=== "Java"

    ```java
    SseStreamingEndpointImpl<Event> events = new SseStreamingEndpointImpl<>(client, "events", Event.class);
    events.setAutoReconnect(false);
    ```

=== "Kotlin"

    ```kotlin
    val events = SseStreamingEndpointImpl(client, "events", Event::class.java)
        .apply { autoReconnect = false }
    ```
