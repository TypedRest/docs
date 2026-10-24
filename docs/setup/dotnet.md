# .NET

## Getting started

### 1. Install the dependency

Add the [TypedRest](https://www.nuget.org/packages/TypedRest/) NuGet package to your project:

```bash
dotnet add package TypedRest
```

### 2. Create an entry endpoint

Create an `EntryEndpoint` pointing at your API:

```csharp
using TypedRest.Endpoints;

var client = new EntryEndpoint(new Uri("https://example.com/api/"));
```

### 3. Define your client class

Extend `EntryEndpoint` and expose your API's resources as properties:

```csharp
class MyClient(Uri uri) : EntryEndpoint(uri)
{
    public CollectionEndpoint<Contact> Contacts => new(this, relativeUri: "./contacts");
}
```

### 4. Use the client

Use your client to interact with the API:

```csharp
var client = new MyClient(new Uri("https://example.com/api/"));

// Read all contacts
List<Contact> contacts = await client.Contacts.ReadAllAsync();

// Create a new contact
ContactEndpoint newContact = await client.Contacts.CreateAsync(new Contact { Name = "Smith" });

// Read a specific contact
Contact contact = await newContact.ReadAsync();
```

### 5. Next steps

- Read the [Introduction](../introduction.md) for the full conceptual walkthrough
- Explore all available [Endpoints](../endpoints/index.md) types
- Check out the [API documentation](https://dotnet.typedrest.net/) for detailed reference

## Additional packages

[TypedRest.Reactive](https://www.nuget.org/packages/TypedRest.Reactive/)  
Adds support for streaming with [ReactiveX (Rx)](http://reactivex.io/) to TypedRest.

[TypedRest.OAuth](https://www.nuget.org/packages/TypedRest.OAuth/)  
Provides an [HttpClient DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler) for [OAuth 2.0](https://oauth.net/2/) / [OpenID Connect](https://openid.net/connect/) authentication; see [below](#oauth-20).  
This can also be used independently of the other TypedRest packages.

[TypedRest.CommandLine](https://www.nuget.org/packages/TypedRest.CommandLine/)  
Build command-line interfaces for TypedRest clients.

## Dependency Injection

If you wish to use TypedRest in an ASP.NET Core web service (or any other kind of application that uses `IServiceCollection`-based dependency injection) you can use the `.AddTypedRest()` extension method. It registers your entry endpoint as a [typed HTTP client](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests#typed-clients), so its `HttpClient` is created and configured by `IHttpClientFactory`:

```csharp
// Register the client class itself
services.AddTypedRest<MyClient>(new Uri("https://example.com/api/"));

// Register the client class behind an interface
services.AddTypedRest<IMyClient, MyClient>(new Uri("https://example.com/api/"));
```

The client class must have a constructor that takes an `HttpClient`. `AddTypedRest()` sets the `HttpClient`'s `BaseAddress` to the URI you pass, which the entry endpoint then uses as its own URI:

```csharp
class MyClient : EntryEndpoint, IMyClient
{
    public MyClient(Uri uri) : base(uri) {} // For use without dependency injection

    public MyClient(HttpClient httpClient) : base(httpClient) {} // For AddTypedRest()

    // ...
}
```

Typed clients are registered as transient, and `IHttpClientFactory` recycles the underlying connections periodically so that DNS changes are picked up. Do not keep a client in a singleton for the lifetime of the application, as that bypasses this recycling. If a singleton needs the client, inject `IServiceScopeFactory` or `IHttpClientFactory` and create a client when needed.

`AddTypedRest()` returns an `IHttpClientBuilder`, so all of the usual HTTP client configuration (timeouts, message handlers, resilience) is available. TypedRest adds two authentication methods on top:

`.AddBasicAuth()`
: Sends HTTP Basic credentials. Takes either a delegate that fills in a `NetworkCredential`, e.g. from configuration, or an `ICredentials` provider that is queried for the client's URI.

`.AddOAuthHandler()`
: Adds the [OAuth handler](#oauth-20) from TypedRest.OAuth. Takes a delegate that configures `OAuthOptions`, a delegate that builds them from the `IServiceProvider`, or the identity server URI, client ID and client secret directly. Options are created separately for each client, so several clients in one application can authenticate against different identity servers. Options set with a configuring delegate or passed directly are registered as named options under the name of the client's `HttpClient`.

```csharp
services.AddTypedRest<IMyClient, MyClient>(new Uri("https://example.com/api/"))
    .AddOAuthHandler(options => configuration.GetSection("MyClient:OAuth").Bind(options));
```

## OAuth 2.0

The [TypedRest.OAuth](https://www.nuget.org/packages/TypedRest.OAuth/) package provides `OAuthHandler`, an `HttpClient` message handler that authenticates requests using the OAuth 2.0 [client credentials flow](https://oauth.net/2/grant-types/client-credentials/), the usual flow for service-to-service calls. It does not depend on the rest of TypedRest, so you can use it with any `HttpClient`.

```csharp
var handler = new OAuthHandler(
    new OAuthOptions
    {
        Uri = new Uri("https://identity.example.com/"),
        ClientId = "my-client",
        ClientSecret = clientSecret,
        Scope = "my-scope"
    },
    innerHandler: new HttpClientHandler());

var client = new MyClient(new HttpClient(handler) { BaseAddress = new Uri("https://example.com/api/") });
```

| Option                | Description                                                     | Default  |
| --------------------- | --------------------------------------------------------------- | -------- |
| `Uri` (required)      | The identity server to request tokens from                      |          |
| `ClientId` (required) | The client identifier to present to the identity server        |          |
| `ClientSecret` (required) | The client secret to present to the identity server       |          |
| `Scope`               | The scope to request a token for                                |          |
| `Audience`            | The audience to request a token for                             |          |
| `TokenLifetimeBuffer` | How long before a token expires to request a new one            | 1 minute |

On the first request, the handler finds the token endpoint using [OpenID Connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) on `Uri`, requests a token and sends it as a `Bearer` header. It reuses the token until `TokenLifetimeBuffer` before it expires.

If the server still responds with `401 Unauthorized`, and its `WWW-Authenticate` header reports `error="invalid_token"` or gives no details, the handler requests a new token and retries the request once. Requests with streamed content, such as uploads, are not retried, because their body cannot be sent a second time.

With dependency injection, use [`.AddOAuthHandler()`](#dependency-injection) instead of building the handler yourself.

## See also

- [API documentation](https://dotnet.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/TypedRest-DotNet)
- [Sample project](https://github.com/TypedRest/Sample-DotNet)
