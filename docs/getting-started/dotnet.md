title: .NET

## Dependencies

Add one or more of the following NuGet packages to your project:

[TypedRest](https://www.nuget.org/packages/TypedRest/)  
The main TypedRest library.

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
