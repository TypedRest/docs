# Testing

TypedRest supports two complementary testing strategies:

[Testing endpoints](endpoints.md)
: Verify that your TypedRest endpoint wrappers issue the correct HTTP requests and correctly parse responses. Uses a mock HTTP server to intercept calls at the network layer.

[Mocking endpoints](mocking.md)
: Test service or business logic that depends on TypedRest endpoints without making any HTTP calls. TypedRest endpoints implement interfaces, so they integrate naturally with standard mocking frameworks.
