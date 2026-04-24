# Exception mapping

TypedRest maps non-success HTTP Status (4xx and 5xx) to exceptions/errors. The following mappings are applied by default:

| HTTP Status Code                 | .NET Exception                | Java/Kotlin Exception      | TypeScript Error      |
| -------------------------------- | ----------------------------- | -------------------------- | --------------------- |
| **400 Bad Request**              | `InvalidDataException`        | `BadRequestException`      | `BadRequestError`     |
| **401 Unauthorized**             | `AuthenticationException`     | `AuthenticationException`  | `AuthenticationError` |
| **403 Forbidden**                | `UnauthorizedAccessException` | `AuthorizationException`   | `AuthorizationError`  |
| **404 NotFound** or **410 Gone** | `KeyNotFoundException`        | `NotFoundException`        | `NotFoundError`       |
| **408 Request Timeout**          | `TimeoutException`            | `TimeoutException`         | `TimeoutError`        |
| **409 Conflict**                 | `InvalidOperationException`   | `ConflictException`        | `ConflictError`       |
| **412 Precondition Failed**      | `InvalidOperationException`   | `ConflictException`        | `ConcurrencyError`    |
| **416 Range Not Satisfiable**    | `InvalidOperationException`   | `ConflictException`        | `HttpError`           |
| other                            | `HttpRequestException`        | `HttpException`            | `HttpError`           |
