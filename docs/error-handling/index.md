title: Overview

TypedRest maps non-success HTTP Status (4xx and 5xx) to exceptions. The following mappings are applied by default:

| HTTP Status Code                 | .NET Exception Type                                                                                                           |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| **400 Bad Request**              | [InvalidDataException](https://docs.microsoft.com/en-us/dotnet/api/system.io.invaliddataexception)                            |
| **401 Unauthorized**             | [AuthenticationException](https://github.com/TypedRest/TypedRest-DotNet/blob/master/src/TypedRest/AuthenticationException.cs) |
| **403 Forbidden**                | [UnauthorizedAccessException](https://docs.microsoft.com/en-us/dotnet/api/system.unauthorizedaccessexception)                 |
| **404 NotFound** or **410 Gone** | [KeyNotFoundException](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.keynotfoundexception)           |
| **408 Request Timeout**          | [TimeoutException](https://docs.microsoft.com/en-us/dotnet/api/system.timeoutexception)                                       |
| **409 Conflict**                 | [InvalidOperationException](https://docs.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)                     |
| **412 Precondition Failed**      | [InvalidOperationException](https://docs.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)                     |
| **416 Range Not Satisfiable**    | [InvalidOperationException](https://docs.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)                     |
| other                            | [HttpRequestException](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httprequestexception)                      |

This can be customized by providing your own implementation of the [error handler interface](https://github.com/TypedRest/TypedRest-DotNet/blob/master/src/TypedRest/Errors/IErrorHandler.cs).
