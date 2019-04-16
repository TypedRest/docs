TypedRest helps you build type-safe, fluent-style REST API clients.

Common REST patterns such as collections are represented as classes, allowing you to write more idiomatic code. For example, TypedRest lets you turn this:
```csharp
var httpClient = new HttpClient(new Uri("http://example.com/"));
var response = httpClient.GetAsync("/contacts/123/address");
var data = await response.Content.ReadAsAsync<AddressDto>();
```

into this:
```csharp
var myService = new MyServiceClient(new Uri("http://example.com/"));
var data = myService.Contacts["123"].Address.ReadAsync();
```

Read more about the **[Motivation](motivation.md)** behind TypedRest,  
or get started now with **[.NET](https://github.com/TypedRest/TypedRest-DotNet)** or **[Java](https://github.com/TypedRest/TypedRest-Java)**!
