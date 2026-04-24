# Polling endpoint

Endpoint for a resource that can be polled for state changes.

!!! note
    Reactive endpoints are not available for TypeScript.  
    For .NET, use the [TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/) NuGet package.  
    For Java/Kotlin, use the [typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive) Maven artifact.

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
    var status = new PollingEndpoint<Status>(client, "status");

    // Poll for state changes
    IObservable<Status> stream = status.GetObservable(TimeSpan.FromSeconds(5));
    stream.Subscribe(x => Console.WriteLine($"Status: {x.State}"));

    // Use as regular element endpoint
    Status current = await status.ReadAsync();
    await status.SetAsync(new Status { State = "active" });
    ```

=== "Java"

    ```java
    PollingEndpoint<Status> status = new PollingEndpointImpl<>(client, "status", Status.class);

    // Poll for state changes
    Observable<Status> stream = status.getObservable(Duration.ofSeconds(5));
    stream.subscribe(x -> System.out.println("Status: " + x.getState()));

    // Use as regular element endpoint
    Status current = status.read();
    status.set(new Status("active"));
    ```

=== "Kotlin"

    ```kotlin
    val status = PollingEndpointImpl(client, "status", Status::class.java)

    // Poll for state changes
    val stream = status.getObservable(Duration.ofSeconds(5))
    stream.subscribe { x -> println("Status: ${x.state}") }

    // Use as regular element endpoint
    val current = status.read()
    status.set(Status("active"))
    ```
