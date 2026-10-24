# Entry endpoint

Represent the top-level URI of an API. Used to address the various resources of the API.

The constructor of the entry endpoint requires you to specify the APIs root URL. It also provides optional parameters to override the default HTTP client, JSON serializer, [error handler](../error-handling/index.md) and [link handler](../link-handling/index.md).

## Usage

=== "C#"

    ```csharp
    // Create an entry endpoint
    var client = new EntryEndpoint(new Uri("http://example.com/api/"));

    // Create child endpoint
    var contacts = new CollectionEndpoint<Contact>(client, "contacts");
    ```

=== "Java"

    ```java
    // Create an entry endpoint
    EntryEndpoint client = new EntryEndpoint(URI.create("http://example.com/api/"));

    // Create child endpoint
    CollectionEndpoint<Contact> contacts = new CollectionEndpointImpl<>(client, "contacts", Contact.class);
    ```

=== "Kotlin"

    ```kotlin
    // Create an entry endpoint
    val client = EntryEndpoint(URI.create("http://example.com/api/"))

    // Create child endpoint
    val contacts = CollectionEndpointImpl(client, "contacts", Contact::class.java)
    ```

=== "TypeScript"

    ```typescript
    // Create an entry endpoint
    const client = new EntryEndpoint(new URL("http://example.com/api/"));

    // Create child endpoint
    const contacts = new CollectionEndpoint<Contact>(client, "contacts");
    ```

## HTTP client

Every endpoint sends its requests through the HTTP client of the entry endpoint it was created from. Configure that HTTP client, for example with a timeout or default headers, and the configuration applies to every request of the client.

=== "C#"

    Pass your own `HttpClient`, or change the one the entry endpoint created via its `HttpClient` property:

    ```csharp
    var client = new EntryEndpoint(new HttpClient { Timeout = TimeSpan.FromSeconds(30) }, new Uri("http://example.com/api/"));
    client.HttpClient.DefaultRequestHeaders.Add("X-Api-Key", apiKey);
    ```

    If you omit the URI, the `HttpClient`'s `BaseAddress` is used instead. In applications that use `IServiceCollection`, let `IHttpClientFactory` create the `HttpClient`; see [Dependency injection](../setup/dotnet.md#dependency-injection).

=== "Java"

    TypedRest for the JVM uses [OkHttp](https://square.github.io/okhttp/). Pass a configured `OkHttpClient`, and add headers with an interceptor:

    ```java
    OkHttpClient httpClient = new OkHttpClient.Builder()
        .connectTimeout(Duration.ofSeconds(30))
        .addInterceptor(chain -> chain.proceed(chain.request().newBuilder()
            .header("X-Api-Key", apiKey)
            .build()))
        .build();
    EntryEndpoint client = new EntryEndpoint(URI.create("http://example.com/api/"), httpClient, new JacksonJsonSerializer());
    ```

    If you use [streaming or SSE endpoints](reactive/index.md), avoid `callTimeout()`: it limits the entire call, including reading the response body, so it cuts off long-lived streams. `readTimeout()` (10 seconds by default) limits the time between two reads, so a stream that stays silent for longer is cut off as well.

=== "Kotlin"

    TypedRest for the JVM uses [OkHttp](https://square.github.io/okhttp/). Pass a configured `OkHttpClient`, and add headers with an interceptor:

    ```kotlin
    val httpClient = OkHttpClient.Builder()
        .connectTimeout(Duration.ofSeconds(30))
        .addInterceptor { chain ->
            chain.proceed(chain.request().newBuilder()
                .header("X-Api-Key", apiKey)
                .build())
        }
        .build()
    val client = EntryEndpoint(URI.create("http://example.com/api/"), httpClient)
    ```

    If you use [streaming or SSE endpoints](reactive/index.md), avoid `callTimeout()`: it limits the entire call, including reading the response body, so it cuts off long-lived streams. `readTimeout()` (10 seconds by default) limits the time between two reads, so a stream that stays silent for longer is cut off as well.

=== "TypeScript"

    Set default headers on the entry endpoint's `httpClient`:

    ```typescript
    const client = new EntryEndpoint(new URL("http://example.com/api/"));
    client.httpClient.defaultHeaders.set("X-Api-Key", apiKey);
    ```

    To change how requests are sent, pass your own implementation of `HttpClient` as the fifth constructor argument, for example a class derived from `FetchHttpClient`.

## Authentication

Since all requests go through the entry endpoint's HTTP client, authentication is configured there too.

### HTTP Basic authentication

=== "C#"

    ```csharp
    var client = new EntryEndpoint(new Uri("https://example.com/api/"), new NetworkCredential("alice", "secret"));
    ```

=== "Java"

    ```java
    EntryEndpoint client = new EntryEndpoint(URI.create("https://example.com/api/"), new HttpCredentials("alice", "secret"), new JacksonJsonSerializer());
    ```

=== "Kotlin"

    ```kotlin
    val client = EntryEndpoint(URI.create("https://example.com/api/"), HttpCredentials("alice", "secret"))
    ```

=== "TypeScript"

    The TypeScript entry endpoint has no credentials parameter. Set the header instead:

    ```typescript
    const client = new EntryEndpoint(new URL("https://example.com/api/"));
    client.httpClient.defaultHeaders.set("Authorization", `Basic ${btoa("alice:secret")}`);
    ```

In .NET, Java and Kotlin you can also put the credentials in the URI, as in `https://alice:secret@example.com/api/`. They are read from the URI and sent as a Basic `Authorization` header.

### API keys and bearer tokens

Send a fixed key or token as a [default header](#http-client), for example `Authorization: Bearer ...` or `X-Api-Key: ...`.

### OAuth 2.0

Tokens from an OAuth 2.0 identity server expire, so the client has to request them, reuse them while they are valid and renew them.

- In .NET, the [TypedRest.OAuth](../setup/dotnet.md#oauth-20) package does this for the client credentials flow.
- TypedRest for the JVM and for TypeScript have no built-in OAuth support. Add the token with an OkHttp interceptor or an `HttpClient` derived from `FetchHttpClient`; the [Authentication](../tutorials/authentication.md#oauth-20-tokens-that-expire) tutorial shows both.

A rejected request throws an `AuthenticationException`/`AuthenticationError` (`401`) or an authorization exception (`403`); see [Exception mapping](../error-handling/index.md).
