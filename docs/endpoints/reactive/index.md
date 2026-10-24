# Reactive endpoints

Reactive endpoints turn repeated or long-running HTTP requests into a stream of entities, so you get notified of changes instead of reading a resource once.

- [Polling endpoint](polling.md) - poll a resource for state changes
- [Streaming Collection endpoint](streaming-collection.md) - follow a collection that is only ever appended to
- [SSE endpoint](sse.md) - receive events pushed by the server as Server-Sent Events
- [Streaming endpoint](streaming.md) - receive entities the server writes into a persistent connection

!!! note "Packages"
    For .NET, add the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package. For Java/Kotlin, add the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact. In TypeScript, reactive endpoints are part of the main `typedrest` package.

## Streams

.NET and Java/Kotlin provide streams as [ReactiveX](https://reactivex.io/) observables via `GetObservable()`/`getObservable()`. TypeScript provides them as async iterables via `stream()`.

Nothing is sent until you subscribe to the observable or start iterating.

## Stopping a stream

=== "C#"

    Dispose the subscription. This cancels the request or the wait between polls.

    ```csharp
    using var subscription = events.GetObservable()
        .Subscribe(x => Console.WriteLine(x.Type));

    // ...

    subscription.Dispose();
    ```

=== "Java"

    Dispose the subscription:

    ```java
    Disposable subscription = events.getObservable()
        .subscribe(x -> System.out.println(x.getType()));

    // ...

    subscription.dispose();
    ```

=== "Kotlin"

    Dispose the subscription:

    ```kotlin
    val subscription = events.getObservable()
        .subscribe { println(it.type) }

    // ...

    subscription.dispose()
    ```

=== "TypeScript"

    `break` out of the loop. To stop a stream from somewhere else, pass an `AbortSignal` to `stream()`. For a streaming collection, the signal is the second argument, after the start index:

    ```typescript
    const controller = new AbortController();
    setTimeout(() => controller.abort(), 60_000);

    for await (const x of events.stream(controller.signal)) {
        console.log(x.type);
    }
    ```

    Aborting ends the loop normally rather than throwing. A polling endpoint is the exception while a request is in flight: aborting it then throws the `AbortError` from `fetch`.

## Choosing an endpoint

|                         | Polling                         | Streaming collection           | SSE                            | Streaming                      |
| ----------------------- | ------------------------------- | ------------------------------ | ------------------------------ | ------------------------------ |
| **Delivers**            | changed states of one resource  | new elements of a collection   | events                         | entities                       |
| **Transport**           | repeated `GET`                  | repeated `GET` with `Range`    | one long-lived `GET`           | one long-lived `GET`           |
| **Server requirements** | none                            | range requests                 | SSE output                     | line-separated output          |
| **Delay**               | up to one polling interval      | up to one polling interval     | immediate                      | immediate                      |
| **Resuming**            | not needed                      | by start index                 | by `Last-Event-ID`             | not supported                  |
| **Good for**            | job status, slow-changing state | logs, feeds, message histories | notifications, live dashboards | simple feeds you don't control |

As a rule of thumb: use polling if you cannot change the server, a streaming collection if the data is a list that only grows and clients need to catch up on what they missed, and SSE if updates must arrive immediately.

Long-lived connections need some care in your infrastructure: proxies and load balancers may close idle connections or buffer responses. The SSE endpoint reconnects for you, but disable response buffering on the server side for streamed responses such as `text/event-stream`.

The [Live updates](../../tutorials/live-updates.md) tutorial uses polling, a streaming collection and SSE for different parts of one API.
