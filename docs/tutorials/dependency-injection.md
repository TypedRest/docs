# Dependency injection

In a service or web application you usually don't create clients by hand. You register them with the dependency injection container once, configure the HTTP client there, and let the container pass the client to whatever needs it.

This tutorial registers the `AddressBookClient` from [Build your first client](first-client.md) in ASP.NET Core.

## ASP.NET Core

Everything in this section works for any .NET application that uses `IServiceCollection`, such as worker services or console apps built on the generic host.

### 1. Prepare the client

`AddTypedRest()` registers the client as a typed HTTP client: `IHttpClientFactory` creates and configures the `HttpClient` and passes it to the client's constructor, so the client needs a constructor that takes one (see [Dependency injection](../setup/dotnet.md#dependency-injection) in the .NET reference). An interface makes it easy to replace the client in tests:

```csharp
public interface IAddressBookClient : IEndpoint
{
    ICollectionEndpoint<Contact, ContactEndpoint> Contacts { get; }
}

public class AddressBookClient : EntryEndpoint, IAddressBookClient
{
    // For use without dependency injection
    public AddressBookClient(Uri uri) : base(uri) {}

    // For AddTypedRest(), which sets HttpClient.BaseAddress to the API's URI
    public AddressBookClient(HttpClient httpClient) : base(httpClient) {}

    public ICollectionEndpoint<Contact, ContactEndpoint> Contacts
        => new CollectionEndpoint<Contact, ContactEndpoint>(this, relativeUri: "./contacts");
}
```

To mock calls deeper in the client, such as `Contacts["1337"].Poke`, give `ContactEndpoint` an interface as well; see [Mocking a custom client](../testing/mocking.md#mocking-a-custom-client). If you [generate the client](from-openapi.md), it already has interfaces for everything, and you can add the constructor in a `partial` class.

### 2. Register the client

Put the API's URI in `appsettings.json`:

```json
{
  "AddressBook": {
    "Uri": "https://example.com/api/"
  }
}
```

and register the client in `Program.cs`:

```csharp
builder.Services.AddTypedRest<IAddressBookClient, AddressBookClient>(
    new Uri(builder.Configuration["AddressBook:Uri"]!));
```

### 3. Use the client

Ask for `IAddressBookClient` wherever you need it, for example in a minimal API handler:

```csharp
app.MapGet("/team", async (IAddressBookClient addressBook, CancellationToken cancellationToken) =>
{
    var contacts = await addressBook.Contacts.ReadAllAsync(cancellationToken);
    return contacts.Select(x => x.Name);
});
```

or in the constructor of a service:

```csharp
public class TeamService(IAddressBookClient addressBook)
{
    public async Task WelcomeAsync(string contactId, CancellationToken cancellationToken)
        => await addressBook.Contacts[contactId].Poke.InvokeAsync(cancellationToken);
}
```

The client is registered as transient. Do not keep it in a singleton for the lifetime of the application; the [reference](../setup/dotnet.md#dependency-injection) explains why and what to do instead.

### 4. Configure the HTTP client

`AddTypedRest()` returns an `IHttpClientBuilder`, so all of the HTTP client configuration that ASP.NET Core offers is available. For example, to set a timeout and add retries for transient errors with [Microsoft.Extensions.Http.Resilience](https://www.nuget.org/packages/Microsoft.Extensions.Http.Resilience/):

```csharp
var uri = new Uri(builder.Configuration["AddressBook:Uri"]!);

builder.Services.AddTypedRest<IAddressBookClient, AddressBookClient>(uri)
    .ConfigureHttpClient(x => x.Timeout = TimeSpan.FromSeconds(30))
    .AddStandardResilienceHandler(options => options.Retry.DisableForUnsafeHttpMethods());
```

`DisableForUnsafeHttpMethods()` turns off retries for `POST`, `PUT`, `PATCH`, `DELETE` and `CONNECT`, so only reads such as `GET` are repeated. Without it, a `POST` that reached the server but whose response got lost would be sent again and could create a contact twice.

### 5. Add authentication

For HTTP Basic authentication:

```csharp
builder.Services.AddTypedRest<IAddressBookClient, AddressBookClient>(uri)
    .AddBasicAuth(credentials => builder.Configuration.GetSection("AddressBook:Credentials").Bind(credentials));
```

For OAuth 2.0, add [TypedRest.OAuth](https://www.nuget.org/packages/TypedRest.OAuth/) and its [handler](../setup/dotnet.md#oauth-20), which requests tokens with the client credentials flow and renews them before they expire:

```csharp
builder.Services.AddTypedRest<IAddressBookClient, AddressBookClient>(uri)
    .AddOAuthHandler(options => builder.Configuration.GetSection("AddressBook:OAuth").Bind(options));
```

```json
{
  "AddressBook": {
    "Uri": "https://example.com/api/",
    "OAuth": {
      "Uri": "https://identity.example.com/",
      "ClientId": "address-book-sync",
      "Scope": "contacts"
    }
  }
}
```

Keep the client secret out of `appsettings.json`. Set `AddressBook:OAuth:ClientSecret` with [user secrets](https://learn.microsoft.com/aspnet/core/security/app-secrets) during development and with an environment variable (`AddressBook__OAuth__ClientSecret`) or a secret store in production.

Each registered client gets its own options, so several clients in one application can authenticate against different identity servers.

## Next steps

- [Testing your client](testing.md): replace the registered client with a mock, or test it against an in-process server.
