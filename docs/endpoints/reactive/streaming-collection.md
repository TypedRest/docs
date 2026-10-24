# Streaming Collection endpoint

Endpoint for a collection of entities observable as an append-only stream using long-polling.

!!! note
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.  
    For TypeScript, reactive endpoints are part of the main `typedrest` package and provide `AsyncIterable`s via `stream()` instead of observables.


| Method     | Input       | Result        | HTTP Verb | Description                                                                    |
| ---------- | ----------- | ------------- | --------- | ------------------------------------------------------------------------------ |
| Get stream | -           | Entity stream | `GET`     | Provides a stream of elements.                                                 |
| Get        | ID / Entity | Endpoint      | -         | Get an [Element endpoint](../generic/element.md) for a specific child element. |
| Read all   | -           | Entities      | `GET`     | Returns all entities in the collection.                                        |
| Read range | Range       | Entities      | `GET`     | Returns all entities within a specific range of the collection.                |
| Create     | Entity      | -             | `POST`    | Adds an entity as a new element to the collection.                             |
| Create All | Entities    | -             | `PATCH`   | Adds (or updates) multiple entities as elements in the collection.             |
| Set All    | Entities    | -             | `PUT`     | Replaces the entire content of the collection with new entities.               |

Extends [Collection endpoint](../generic/collection.md)

## Usage

=== "C#"

    ```csharp
    var messages = new StreamingCollectionEndpoint<Message>(client, "messages");

    // Subscribe to new messages
    IObservable<Message> stream = messages.GetObservable();
    stream.Subscribe(m => Console.WriteLine($"New message: {m.Text}"));

    // Use as regular collection endpoint
    List<Message> all = await messages.ReadAllAsync();
    await messages.CreateAsync(new Message { Text = "Hello!" });
    ```

    You can also configure `StreamingCollectionEndpoint` to use a custom type derived from `ElementEndpoint` for [elements](../generic/element.md).

    ```csharp
    var messages = new StreamingCollectionEndpoint<Message, MessageEndpoint>(client, "messages");
    ```

=== "Java"

    ```java
    StreamingCollectionEndpoint<Message> messages = new StreamingCollectionEndpointImpl<>(client, "messages", Message.class);

    // Subscribe to new messages
    Observable<Message> stream = messages.getObservable(0); // start index
    stream.subscribe(x -> System.out.println("New message: " + x.getText()));

    // Use as regular collection endpoint
    List<Message> all = messages.readAll();
    messages.create(new Message("Hello!"));
    ```

    You can also use `GenericStreamingCollectionEndpointImpl` to use a custom type derived from `ElementEndpoint` for [elements](../generic/element.md).

    ```java
    GenericStreamingCollectionEndpoint<Message, MessageEndpoint> messages = new GenericStreamingCollectionEndpointImpl<>(client, "messages", Message.class, (ref, uri) -> new MessageEndpoint(ref, uri));
    ```

=== "Kotlin"

    ```kotlin
    val messages = StreamingCollectionEndpointImpl(client, "messages", Message::class.java)

    // Subscribe to new messages
    val stream = messages.getObservable()
    stream.subscribe { x -> println("New message: ${x.text}") }

    // Use as regular collection endpoint
    val all = messages.readAll()
    messages.create(Message("Hello!"))
    ```

    You can also use `GenericStreamingCollectionEndpointImpl` to use a custom type derived from `ElementEndpoint` for [elements](../generic/element.md).

    ```kotlin
    val messages = GenericStreamingCollectionEndpointImpl(client, "messages", Message::class.java, ::MessageEndpoint)
    ```

=== "TypeScript"

    ```typescript
    const messages = new StreamingCollectionEndpoint<Message>(client, "messages");

    // Use as regular collection endpoint
    const all = await messages.readAll();
    await messages.create({ text: "Hello!" });

    // Consume new messages
    for await (const x of messages.stream()) {
        console.log(`New message: ${x.text}`);
    }
    ```

    You can also use `GenericStreamingCollectionEndpoint` to use a custom type derived from `ElementEndpoint` for [elements](../generic/element.md).

    ```typescript
    const messages = new GenericStreamingCollectionEndpoint<Message, MessageEndpoint>(client, "messages", MessageEndpoint);
    ```

## Start index

Pass a start index to skip elements you have already seen, which lets a client resume where it left off, or a negative one to start with the last few. Zero, the default, starts with the first element.

=== "C#"

    ```csharp
    // Start after the first 10 elements
    messages.GetObservable(startIndex: 10);

    // Start with the last 5 elements
    messages.GetObservable(startIndex: -5);
    ```

=== "Java"

    ```java
    // Start after the first 10 elements
    messages.getObservable(10);

    // Start with the last 5 elements
    messages.getObservable(-5);
    ```

=== "Kotlin"

    ```kotlin
    // Start after the first 10 elements
    messages.getObservable(10)

    // Start with the last 5 elements
    messages.getObservable(-5)
    ```

=== "TypeScript"

    ```typescript
    // Start after the first 10 elements
    messages.stream(10);

    // Start with the last 5 elements
    messages.stream(-5);
    ```

## Server requirements

The endpoint uses the same HTTP range requests as [reading a range](../generic/collection.md) of a regular collection, so the server has to support them:

1. The client requests `Range: elements=N-`, from the next position it has not seen to the end (or `Range: elements=-N` for the last `N` elements).
2. If there are elements from `N` on, the server responds with `206 Partial Content`, the elements, and `Content-Range: elements N-M/*`. The client waits for the polling interval and asks for the elements after `M`.
3. If there are none yet, the server responds with `416 Range Not Satisfiable`. The client waits for the polling interval and asks again.

!!! warning "Leave out the total"
    Use `*` for the total in `Content-Range`, as above. If the server sends a total (`elements 0-9/10`) and the response reaches it, the client takes that as the end of the collection and the stream completes.

The polling interval is 3 seconds by default. Change it with `PollingInterval`/`pollingInterval` (a `TimeSpan` in .NET, a `Duration` in Java/Kotlin, milliseconds in TypeScript). If a response has a `Retry-After` header, the endpoint uses it as the new polling interval.
