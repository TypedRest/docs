# Polling endpoint

Endpoint for a resource that can be polled for state changes.

!!! note
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.  
    For TypeScript, reactive endpoints are part of the main `typedrest` package and provide `AsyncIterable`s via `stream()` instead of observables.

| Method     | Input  | Result        | HTTP Verb | Description                                      |
| ---------- | ------ | ------------- | --------- | ------------------------------------------------ |
| Get stream | -      | Entity stream | `GET`     | Provides a stream of entity states.              |
| Exists     | -      | Boolean       | `HEAD`    | Determines whether the element currently exists. |
| Read       | -      | Entity        | `GET`     | Returns the entity.                              |
| Set        | Entity | Entity        | `PUT`     | Sets/replaces the entity.                        |
| Merge      | Entity | Entity        | `PATCH`   | Modifies the existing entity by merging changes. |
| Delete     | -      | -             | `DELETE`  | Deletes the element.                             |

Extends [Element endpoint](../generic/element.md)

## Usage

=== "C#"

    ```csharp
    var status = new PollingEndpoint<Status>(client, "status")
    {
        PollingInterval = TimeSpan.FromSeconds(5)
    };

    // Poll for state changes
    IObservable<Status> stream = status.GetObservable();
    stream.Subscribe(x => Console.WriteLine($"Status: {x.State}"));

    // Use as regular element endpoint
    Status current = await status.ReadAsync();
    await status.SetAsync(new Status { State = "active" });
    ```

=== "Java"

    ```java
    PollingEndpoint<Status> status = new PollingEndpointImpl<>(client, "status", Status.class, null); // no end condition
    status.setPollingInterval(Duration.ofSeconds(5));

    // Poll for state changes
    Observable<Status> stream = status.getObservable();
    stream.subscribe(x -> System.out.println("Status: " + x.getState()));

    // Use as regular element endpoint
    Status current = status.read();
    status.set(new Status("active"));
    ```

=== "Kotlin"

    ```kotlin
    val status = PollingEndpointImpl(client, "status", Status::class.java)
        .apply { pollingInterval = Duration.ofSeconds(5) }

    // Poll for state changes
    val stream = status.getObservable()
    stream.subscribe { x -> println("Status: ${x.state}") }

    // Use as regular element endpoint
    val current = status.read()
    status.set(Status("active"))
    ```

=== "TypeScript"

    ```typescript
    const status = new PollingEndpoint<Status>(client, "status");
    status.pollingInterval = 5000;

    // Use as regular element endpoint
    const current = await status.read();
    await status.set({ state: "active" });

    // Poll for state changes
    for await (const x of status.stream()) {
        console.log(`Status: ${x.state}`);
    }
    ```

    The loop keeps polling until you `break` out of it or the `AbortSignal` passed to `stream()` is triggered (see [Stopping a stream](index.md#stopping-a-stream)).

## End condition

Pass an end condition to the constructor to stop polling once the entity reaches its final state. The stream emits that final state and then completes:

=== "C#"

    ```csharp
    var job = new PollingEndpoint<Job>(client, "jobs/42", endCondition: x => x.State == "completed");
    ```

=== "Java"

    ```java
    PollingEndpoint<Job> job = new PollingEndpointImpl<>(client, "jobs/42", Job.class, x -> x.getState().equals("completed"));
    ```

=== "Kotlin"

    ```kotlin
    val job = PollingEndpointImpl(client, "jobs/42", Job::class.java) { it.state == "completed" }
    ```

=== "TypeScript"

    ```typescript
    const job = new PollingEndpoint<Job>(client, "jobs/42", x => x.state === "completed");
    ```

Without an end condition, the stream continues until you stop it.

## Change detection

States identical to the previous one are skipped, so the stream only delivers changes.

.NET and Java/Kotlin compare states with `Equals`/`equals`. Give the entity class value equality (a C# `record`, a Kotlin `data class`, or overridden `equals` in Java); otherwise every poll deserializes a new, unequal object and counts as a change. TypeScript compares the serialized JSON.

This also applies to DTOs from the [code generator](../../code-generation/index.md): generated Kotlin DTOs are `data class`es, but generated C# and Java DTOs are plain classes without value equality. Add `Equals`/`GetHashCode` in a partial class (C#), or poll with a hand-written entity class (Java).

## Polling interval

The endpoint waits 3 seconds between requests by default. Change this with `PollingInterval`/`pollingInterval` (a `TimeSpan` in .NET, a `Duration` in Java/Kotlin, milliseconds in TypeScript).

The server can slow clients down: if a response has a `Retry-After` header, the endpoint uses it as the new polling interval.
