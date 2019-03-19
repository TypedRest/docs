TypedRest helps you build type-safe, fluent-style REST API clients.

Common REST patterns such as collections are represented as classes, allowing you to write more idiomatic code. For example, TypedRest lets you turn this:
```csharp
var httpClient = new HttpClient {BaseAddress = new Uri("http://example.com/")};
var response = httpClient.GetAsync("/contacts/123");
var contact = await response.Content.ReadAsAsync<ContactDto>();
```

into this:
```csharp
var myService = new MyServiceClient(new Uri("http://example.com/"));
var contact = myService.Contacts["123"].ReadAsync();
```

Read more about the **[Motivation](motivation.md)** behind TypedRest,  
or get started now with **[.NET](https://github.com/TypedRest/TypedRest-DotNet)** or **[Java](https://github.com/TypedRest/TypedRest-Java)**!
