# Streaming Collection endpoint

Endpoint for a collection of entities observable as an append-only stream using long-polling.

!!! note
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.  
    For TypeScript, reactive endpoints are part of the main `typedrest` package and provide `AsyncIterable`s via `stream()` instead of observables.


| Method         | Input       | Result        | HTTP Verb | Description                                                                    |
| -------------- | ----------- | ------------- | --------- | ------------------------------------------------------------------------------ |
| Get observable | -           | Entity stream | `GET`     | Provides an observable stream of elements.                                     |
| Get            | ID / Entity | Endpoint      | -         | Get an [Element endpoint](../generic/element.md) for a specific child element. |
| Read all       | -           | Entities      | `GET`     | Returns all entities in the collection.                                        |
| Read range     | Range       | Entities      | `GET`     | Returns all entities within a specific range of the collection.                |
| Create         | Entity      | -             | `POST`    | Adds an entity as a new element to the collection.                             |
| Create All     | Entities    | -             | `PATCH`   | Adds (or updates) multiple entities as elements in the collection.             |
| Set All        | Entities    | -             | `PUT`     | Replaces the entire content of the collection with new entities.               |

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
    Observable<Message> stream = messages.getObservable();
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

    Pass a start index to `stream()` to skip elements you have already seen, or a negative one to start with the last few:

    ```typescript
    // Start after the first 10 elements
    messages.stream(10);

    // Start with the last 5 elements
    messages.stream(-5);
    ```

    You can also use `GenericStreamingCollectionEndpoint` to use a custom type derived from `ElementEndpoint` for [elements](../generic/element.md).

    ```typescript
    const messages = new GenericStreamingCollectionEndpoint<Message, MessageEndpoint>(client, "messages", MessageEndpoint);
    ```
