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
using TypedRest;

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
Provides an [HttpClient DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler) for [OAuth 2.0](https://oauth.net/2/) / [OpenID Connect](https://openid.net/connect/) authentication.  
This can also be used independently of the other TypedRest packages.

[TypedRest.CommandLine](https://www.nuget.org/packages/TypedRest.CommandLine/)  
Build command-line interfaces for TypedRest clients.

## Dependency Injection

If you wish to use TypedRest in an ASP.NET Core web service (or any other kind of service that uses `IServiceProvider`-based dependency injection) you can use the `.AddTypedRest<T>()` extension method. This registers the endpoint you specify for dependency injection and connects it with [HttpClientFactory](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).

## See also

- [API documentation](https://dotnet.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/TypedRest-DotNet)
- [Sample project](https://github.com/TypedRest/Sample-DotNet)
