# Polling endpoint

Endpoint for a resource that can be polled for state changes.

!!! note
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.  
    For TypeScript, reactive endpoints are part of the main `typedrest` package and provide `AsyncIterable`s via `stream()` instead of observables.

| Method         | Input  | Result        | HTTP Verb | Description                                      |
| -------------- | ------ | ------------- | --------- | ------------------------------------------------ |
| Get observable | -      | Entity stream | `GET`     | Provides an observable stream of entity states.  |
| Exists         | -      | Boolean       | `HEAD`    | Determines whether the element currently exists. |
| Read           | -      | Entity        | `GET`     | Returns the entity.                              |
| Set            | Entity | Entity        | `PUT`     | Sets/replaces the entity.                        |
| Merge          | Entity | Entity        | `PATCH`   | Modifies the existing entity by merging changes. |
| Delete         | -      | -             | `DELETE`  | Deletes the element.                             |

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
    PollingEndpoint<Status> status = new PollingEndpointImpl<>(client, "status", Status.class);
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

    The loop keeps polling until you `break` out of it, until the `AbortSignal` passed to `stream()` is triggered, or until the entity reaches the state described by the end condition:

    ```typescript
    const status = new PollingEndpoint<Status>(client, "status", x => x.state === "completed");
    ```
