# Testing endpoints

To verify that your TypedRest endpoint configuration issues the correct HTTP requests and correctly deserializes responses, use a mock HTTP server. This lets you write fast, deterministic tests without a running API server.

Most of what an endpoint does comes from TypedRest and is tested there. What is specific to your client is its structure: does each child endpoint point to the right URI and use the right endpoint type? One test per child endpoint is usually enough. You don't need to test every method of every endpoint.

## Verifying requests

=== "C#"

    Install [RichardSzalay.MockHttp](https://github.com/richardszalay/mockhttp) to intercept `HttpClient` calls:

    ```
    dotnet add package RichardSzalay.MockHttp
    ```

    Inject a `MockHttpMessageHandler` into `HttpClient` and pass it to your entry endpoint:

    ```csharp
    var mock = new MockHttpMessageHandler();
    var client = new EntryEndpoint(new HttpClient(mock), new Uri("http://localhost/"));
    var endpoint = new ElementEndpoint<Contact>(client, "contacts/1");

    mock.Expect(HttpMethod.Get, "http://localhost/contacts/1")
        .Respond("application/json", """{"name":"Smith"}""");

    var contact = await endpoint.ReadAsync();
    Assert.Equal("Smith", contact.Name);

    mock.VerifyNoOutstandingExpectation();
    ```

    To verify a request body, use `.WithContent()`:

    ```csharp
    mock.Expect(HttpMethod.Put, "http://localhost/contacts/1")
        .WithContent("""{"name":"Smith"}""")
        .Respond("application/json", """{"name":"Smith"}""");

    await endpoint.SetAsync(new Contact { Name = "Smith" });
    ```

=== "Java"

    Install [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) and point your entry endpoint at the mock server's URL:

    ```java
    MockWebServer server = new MockWebServer();

    EntryEndpoint client = new EntryEndpoint(server.url("/").uri(), (HttpCredentials) null, new JacksonJsonSerializer());
    ElementEndpoint<Contact> endpoint = new ElementEndpointImpl<>(client, "contacts/1", Contact.class);

    server.enqueue(new MockResponse()
        .setBody("{\"name\":\"Smith\"}")
        .addHeader("Content-Type", "application/json"));

    Contact contact = endpoint.read();
    assertEquals("Smith", contact.getName());

    RecordedRequest request = server.takeRequest();
    assertEquals("GET", request.getMethod());
    assertEquals("/contacts/1", request.getPath());

    server.close();
    ```

    To verify a request body:

    ```java
    server.enqueue(new MockResponse()
        .setBody("{\"name\":\"Smith\"}")
        .addHeader("Content-Type", "application/json"));

    endpoint.set(new Contact("Smith"));

    RecordedRequest request = server.takeRequest();
    assertEquals("PUT", request.getMethod());
    assertEquals("{\"name\":\"Smith\"}", request.getBody().readUtf8());
    ```

=== "Kotlin"

    Install [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) and point your entry endpoint at the mock server's URL:

    ```kotlin
    val server = MockWebServer()

    val client = EntryEndpoint(server.url("/").toUri())
    val endpoint = ElementEndpointImpl(client, "contacts/1", Contact::class.java)

    server.enqueue(MockResponse()
        .setBody("""{"name":"Smith"}""")
        .addHeader("Content-Type", "application/json"))

    val contact = endpoint.read()
    assertEquals("Smith", contact.name)

    val request = server.takeRequest()
    assertEquals("GET", request.method)
    assertEquals("/contacts/1", request.path)

    server.close()
    ```

    To verify a request body:

    ```kotlin
    server.enqueue(MockResponse()
        .setBody("""{"name":"Smith"}""")
        .addHeader("Content-Type", "application/json"))

    endpoint.set(Contact("Smith"))

    val request = server.takeRequest()
    assertEquals("PUT", request.method)
    assertEquals("""{"name":"Smith"}""", request.body.readUtf8())
    ```

=== "TypeScript"

    Install [jest-fetch-mock](https://github.com/jefflau/jest-fetch-mock) to intercept `fetch` calls:

    ```bash
    npm install --save-dev jest-fetch-mock
    ```

    Enable mocking in your test setup and use `mockOnceIf` to queue responses:

    ```typescript
    import fetchMock from 'jest-fetch-mock';

    fetchMock.enableMocks();

    beforeEach(() => fetchMock.resetMocks());

    test('read contact', async () => {
        const client = new EntryEndpoint(new URL('http://localhost/'));
        const endpoint = new ElementEndpoint<Contact>(client, 'contacts/1');

        fetchMock.mockOnceIf(
            'http://localhost/contacts/1',
            '{"name":"Smith"}',
            { headers: { 'Content-Type': 'application/json' } }
        );

        const contact = await endpoint.read();
        expect(contact.name).toBe('Smith');
    });
    ```

    To verify a request body:

    ```typescript
    test('set contact', async () => {
        fetchMock.mockOnceIf(
            req => req.method === 'PUT' && req.url === 'http://localhost/contacts/1',
            async req => {
                expect(await req.text()).toBe('{"name":"Smith"}');
                return { body: '{"name":"Smith"}', headers: { 'Content-Type': 'application/json' } };
            }
        );

        await endpoint.set({ name: 'Smith' });
    });
    ```

## Testing error responses

A mock server is also the easiest way to produce error responses that are hard to cause with a real API. Respond with an error status and check that TypedRest [maps it](../error-handling/index.md) to the expected exception:

=== "C#"

    ```csharp
    mock.Expect(HttpMethod.Get, "http://localhost/contacts/1")
        .Respond(HttpStatusCode.NotFound);

    await Assert.ThrowsAsync<KeyNotFoundException>(() => endpoint.ReadAsync());
    ```

=== "Java"

    ```java
    server.enqueue(new MockResponse().setResponseCode(404));

    assertThrows(NotFoundException.class, endpoint::read);
    ```

=== "Kotlin"

    ```kotlin
    server.enqueue(MockResponse().setResponseCode(404))

    assertThrows<NotFoundException> { endpoint.read() }
    ```

=== "TypeScript"

    ```typescript
    import { NotFoundError } from 'typedrest/errors';

    test('contact not found', async () => {
        fetchMock.mockOnceIf('http://localhost/contacts/1', '', { status: 404 });

        await expect(endpoint.read()).rejects.toBeInstanceOf(NotFoundError);
    });
    ```

The same approach tests how your application reacts to `401`, `403` or `500` responses, or to a `412 Precondition Failed` after a write with an outdated ETag (see [Optimistic concurrency](../tutorials/concurrency.md)).
