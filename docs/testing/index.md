# Testing

A TypedRest client sits between your application and an API, so there are three things worth testing:

[Testing endpoints](endpoints.md)
: Verify that your TypedRest endpoint wrappers issue the correct HTTP requests, correctly parse responses and turn error responses into the right exceptions. Uses a mock HTTP server to intercept calls at the network layer. Catches wrong URIs, methods, bodies or error handling.

[Mocking endpoints](mocking.md)
: Test service or business logic that depends on TypedRest endpoints without making any HTTP calls. TypedRest endpoints implement interfaces, so they integrate naturally with standard mocking frameworks. Catches bugs in your own logic.

[Integration testing](integration.md)
: Run the service in the test process and point the client at it. Catches mismatches between client and service, such as differing routes, unexpected status codes or properties serialized differently.

The [Testing your client](../tutorials/testing.md) tutorial applies all three to one client.
