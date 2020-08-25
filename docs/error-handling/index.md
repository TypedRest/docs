# Exception mapping

TypedRest maps non-success HTTP Status (4xx and 5xx) to exceptions/errors. The following mappings are applied by default:

| HTTP Status Code                 | .NET Exception                | JavaScript Error      |
| -------------------------------- | ----------------------------- | --------------------- |
| **400 Bad Request**              | `InvalidDataException`        | `BadRequestError`     |
| **401 Unauthorized**             | `AuthenticationException`     | `AuthenticationError` |
| **403 Forbidden**                | `UnauthorizedAccessException` | `AuthorizationError`  |
| **404 NotFound** or **410 Gone** | `KeyNotFoundException`        | `NotFoundError`       |
| **408 Request Timeout**          | `TimeoutException`            | `TimeoutError`        |
| **409 Conflict**                 | `InvalidOperationException`   | `ConflictError`       |
| **412 Precondition Failed**      | `InvalidOperationException`   | `ConcurrencyError`    |
| **416 Range Not Satisfiable**    | `InvalidOperationException`   | `RangeError`          |
| other                            | `HttpRequestException`        | `HttpError`           |
