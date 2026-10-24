# Live updates

Sometimes reading a resource once is not enough: you want to know when it changes. TypedRest has four [reactive endpoint](../endpoints/index.md) types for this. Each one turns repeated or long-running HTTP requests into a stream of entities, but they put different demands on the server.

This tutorial uses each of them for a different part of the address book: an import job you wait for, a change log you follow, and events the server pushes.

!!! note "Packages"
    For .NET, add the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package. For Java/Kotlin, add the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact. In TypeScript, reactive endpoints are part of the main `typedrest` package. .NET and Java/Kotlin provide streams as observables, TypeScript as async iterables; see [Reactive endpoints](../endpoints/reactive/index.md).

## Wait for a job to finish: polling

Say the API imports contacts from a file in the background. `POST /imports` starts an import, and `GET /imports/42` returns its current state:

```json
{ "state": "running", "progress": 40 }
```

The server needs nothing special for this. A [polling endpoint](../endpoints/reactive/polling.md) reads the resource repeatedly and emits each new state, until the end condition you pass is met:

=== "C#"

    ```csharp
    var job = new PollingEndpoint<ImportJob>(client, "./imports/42", endCondition: x => x.State == "completed")
    {
        PollingInterval = TimeSpan.FromSeconds(2)
    };

    // Awaiting an observable returns its last element (requires using System.Reactive.Linq)
    ImportJob final = await job.GetObservable()
        .Do(x => Console.WriteLine($"{x.Progress}% done"));
    ```

=== "Java"

    ```java
    PollingEndpointImpl<ImportJob> job = new PollingEndpointImpl<>(client, "./imports/42", ImportJob.class,
        x -> x.getState().equals("completed"));
    job.setPollingInterval(Duration.ofSeconds(2));

    job.getObservable()
        .blockingForEach(x -> System.out.println(x.getProgress() + "% done"));
    ```

=== "Kotlin"

    ```kotlin
    val job = PollingEndpointImpl(client, "./imports/42", ImportJob::class.java) { it.state == "completed" }
        .apply { pollingInterval = Duration.ofSeconds(2) }

    job.getObservable()
        .blockingForEach { println("${it.progress}% done") }
    ```

=== "TypeScript"

    ```typescript
    const job = new PollingEndpoint<ImportJob>(client, "./imports/42", x => x.state === "completed");
    job.pollingInterval = 2000;

    for await (const x of job.stream()) {
        console.log(`${x.progress}% done`);
    }
    ```

States identical to the previous one are skipped, so you only see changes. In .NET and Java/Kotlin, give `ImportJob` value equality for this to work; see [Change detection](../endpoints/reactive/polling.md#change-detection). The server can also slow clients down with a [`Retry-After` header](../endpoints/reactive/polling.md#polling-interval).

A polling endpoint is also an [element endpoint](../endpoints/generic/element.md), so you can still read, set or delete the job with it.

## Follow a growing collection: streaming collection

Say the API keeps a log of every change to the address book at `/contacts/changes`, where new entries are only ever added to the end. A [streaming collection endpoint](../endpoints/reactive/streaming-collection.md) reads the entries that exist and then keeps asking for new ones:

=== "C#"

    ```csharp
    var changes = new StreamingCollectionEndpoint<ContactChange>(client, "./contacts/changes");

    // Start with the last 10 entries, then follow new ones
    using var subscription = changes.GetObservable(startIndex: -10)
        .Subscribe(x => Console.WriteLine($"{x.Type}: {x.ContactId}"));
    ```

=== "Java"

    ```java
    StreamingCollectionEndpoint<ContactChange> changes = new StreamingCollectionEndpointImpl<>(client, "./contacts/changes", ContactChange.class);

    // Start with the last 10 entries, then follow new ones
    Disposable subscription = changes.getObservable(-10)
        .subscribe(x -> System.out.println(x.getType() + ": " + x.getContactId()));
    ```

=== "Kotlin"

    ```kotlin
    val changes = StreamingCollectionEndpointImpl(client, "./contacts/changes", ContactChange::class.java)

    // Start with the last 10 entries, then follow new ones
    val subscription = changes.getObservable(-10)
        .subscribe { println("${it.type}: ${it.contactId}") }
    ```

=== "TypeScript"

    ```typescript
    const changes = new StreamingCollectionEndpoint<ContactChange>(client, "./contacts/changes");

    // Start with the last 10 entries, then follow new ones
    for await (const x of changes.stream(-10)) {
        console.log(`${x.type}: ${x.contactId}`);
    }
    ```

A positive [start index](../endpoints/reactive/streaming-collection.md#start-index) skips that many entries instead, which lets a client that has already processed entries resume where it left off.

This uses the same [range requests](paging.md) as paging: the client keeps asking for the entries after the last one it has seen, and the server answers `416 Range Not Satisfiable` while there are none. [Server requirements](../endpoints/reactive/streaming-collection.md#server-requirements) describes the exchange, including why the server must leave the total out of `Content-Range`.

The endpoint is still a [collection endpoint](../endpoints/generic/collection.md), so you can also read and create entries with it.

## Receive pushed events: Server-Sent Events

Polling always lags behind by up to one interval and sends requests even when nothing happens. If the server can keep a connection open and push events as they occur, use [Server-Sent Events](https://developer.mozilla.org/docs/Web/API/Server-sent_events) (SSE), the standard format for this that browsers also understand.

Say `GET /contacts/1337/events` pushes an event whenever that contact changes. The server responds with `Content-Type: text/event-stream` and writes events into the open response as they happen:

```
id: 17
event: updated
data: {"type":"updated","contact":{"id":"1337","name":"Smith"}}

id: 18
event: deleted
data: {"type":"deleted"}

```

An [SSE endpoint](../endpoints/reactive/sse.md) deserializes the `data` of each event as an entity:

=== "C#"

    ```csharp
    var events = new SseStreamingEndpoint<ContactEvent>(client, "./contacts/1337/events");

    using var subscription = events.GetObservable()
        .Subscribe(x => Console.WriteLine($"{x.Type}: {x.Contact?.Name}"));
    ```

=== "Java"

    ```java
    StreamingEndpoint<ContactEvent> events = new SseStreamingEndpointImpl<>(client, "./contacts/1337/events", ContactEvent.class, null);

    Disposable subscription = events.getObservable()
        .subscribe(x -> System.out.println(x.getType()));
    ```

=== "Kotlin"

    ```kotlin
    val events = SseStreamingEndpointImpl(client, "./contacts/1337/events", ContactEvent::class.java)

    val subscription = events.getObservable()
        .subscribe { println("${it.type}: ${it.contact?.name}") }
    ```

=== "TypeScript"

    ```typescript
    const events = new SseStreamingEndpoint<ContactEvent>(client, "./contacts/1337/events");

    for await (const x of events.stream()) {
        console.log(`${x.type}: ${x.contact?.name}`);
    }
    ```

If the connection drops, the endpoint [reconnects](../endpoints/reactive/sse.md#reconnecting) on its own and sends the `id` of the last event it received as a `Last-Event-ID` header, so a server that keeps recent events can send the ones the client missed.

To receive only some kinds of events, pass an [event type](../endpoints/reactive/sse.md#event-types) to the constructor; events with a different `event:` field are then ignored.

If your server streams entities without the SSE format, for example as newline-delimited JSON, use a [streaming endpoint](../endpoints/reactive/streaming.md) instead. It splits the response body on a separator, but has no event types, IDs or reconnecting.

## Stopping a stream

In .NET and Java/Kotlin, dispose the subscription; the C# examples above do this with `using` when the variable goes out of scope. In TypeScript, `break` out of the loop. To stop a TypeScript stream from somewhere else, pass an `AbortSignal`; see [Stopping a stream](../endpoints/reactive/index.md#stopping-a-stream).

## Choosing an endpoint

As a rule of thumb: use polling if you cannot change the server, a streaming collection if the data is a list that only grows and clients need to catch up on what they missed, and SSE if updates must arrive immediately. [Choosing an endpoint](../endpoints/reactive/index.md#choosing-an-endpoint) compares all four reactive endpoints, including what each needs from your infrastructure.
