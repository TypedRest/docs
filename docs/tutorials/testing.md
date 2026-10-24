# Testing your client

A TypedRest client sits between your application and an API, so there are three things worth testing: that the endpoints send the right requests, that your code uses the client correctly, and that client and service agree. [Testing](../testing/index.md) describes each of these strategies and the tools they use.

This tutorial applies them to the `AddressBookClient` from [Build your first client](first-client.md).

## Test the requests your endpoints send

What is specific to your client is its structure: does `Contacts["1337"].Poke` really go to `/contacts/1337/poke`? A mock HTTP server answers the requests and records them, so you can check them without a real API (see [Testing endpoints](../testing/endpoints.md) for setup):

=== "C#"

    With [RichardSzalay.MockHttp](https://github.com/richardszalay/mockhttp), pass the mock handler to the client through its `HttpClient`. This uses the `AddressBookClient(HttpClient)` constructor from [Dependency injection](dependency-injection.md#1-prepare-the-client):

    ```csharp
    public class AddressBookClientFacts
    {
        private readonly MockHttpMessageHandler _mock = new();
        private readonly AddressBookClient _client;

        public AddressBookClientFacts()
        {
            _client = new AddressBookClient(new HttpClient(_mock) { BaseAddress = new Uri("http://localhost/") });
        }

        [Fact]
        public async Task PokesContact()
        {
            _mock.Expect(HttpMethod.Post, "http://localhost/contacts/1337/poke")
                 .Respond(HttpStatusCode.NoContent);

            await _client.Contacts["1337"].Poke.InvokeAsync();

            _mock.VerifyNoOutstandingExpectation();
        }
    }
    ```

=== "Java"

    With [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver), point the client at the mock server's URL:

    ```java
    class AddressBookClientTest {
        private MockWebServer server;
        private AddressBookClient client;

        @BeforeEach
        void setUp() throws IOException {
            server = new MockWebServer();
            server.start();
            client = new AddressBookClient(server.url("/").uri());
        }

        @AfterEach
        void tearDown() throws IOException {
            server.close();
        }

        @Test
        void pokesContact() throws InterruptedException {
            server.enqueue(new MockResponse().setResponseCode(204));

            client.getContacts().get("1337").getPoke().invoke();

            RecordedRequest request = server.takeRequest();
            assertEquals("POST", request.getMethod());
            assertEquals("/contacts/1337/poke", request.getPath());
        }
    }
    ```

=== "Kotlin"

    With [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver), point the client at the mock server's URL:

    ```kotlin
    class AddressBookClientTest {
        private val server = MockWebServer()
        private lateinit var client: AddressBookClient

        @BeforeEach
        fun setUp() {
            server.start()
            client = AddressBookClient(server.url("/").toUri())
        }

        @AfterEach
        fun tearDown() = server.close()

        @Test
        fun pokesContact() {
            server.enqueue(MockResponse().setResponseCode(204))

            client.contacts["1337"].poke.invoke()

            val request = server.takeRequest()
            assertEquals("POST", request.method)
            assertEquals("/contacts/1337/poke", request.path)
        }
    }
    ```

=== "TypeScript"

    With [jest-fetch-mock](https://github.com/jefflau/jest-fetch-mock), queue a response and check the request it was used for:

    ```typescript
    import fetchMock from "jest-fetch-mock";

    fetchMock.enableMocks();
    beforeEach(() => fetchMock.resetMocks());

    test("pokes contact", async () => {
        const client = new AddressBookClient(new URL("http://localhost/"));
        fetchMock.mockResponseOnce("", { status: 204 });

        await client.contacts.get("1337").poke.invoke();

        const [url, init] = fetchMock.mock.calls[0];
        expect(url).toBe("http://localhost/contacts/1337/poke");
        expect(init?.method).toBe("POST");
    });
    ```

One test per child endpoint is usually enough.

### Test your error handling

A mock server can also [produce errors](../testing/endpoints.md#testing-error-responses) that are hard to cause with a real API. This test simulates a [conflicting update](concurrency.md): the read returns an ETag, and the server rejects the following write with `412 Precondition Failed`:

=== "C#"

    ```csharp
    [Fact]
    public async Task ReportsConflict()
    {
        _mock.Expect(HttpMethod.Get, "http://localhost/contacts/1337")
             .Respond(_ => new HttpResponseMessage(HttpStatusCode.OK)
             {
                 Content = new StringContent("""{"id":"1337","name":"Smith"}""", Encoding.UTF8, "application/json"),
                 Headers = { ETag = new EntityTagHeaderValue("\"v7\"") }
             });
        _mock.Expect(HttpMethod.Put, "http://localhost/contacts/1337")
             .WithHeaders("If-Match", "\"v7\"")
             .Respond(HttpStatusCode.PreconditionFailed);

        var contact = _client.Contacts["1337"];
        var entity = await contact.ReadAsync();
        await Assert.ThrowsAsync<InvalidOperationException>(() => contact.SetAsync(entity));

        _mock.VerifyNoOutstandingExpectation();
    }
    ```

=== "Java"

    ```java
    @Test
    void reportsConflict() throws InterruptedException {
        server.enqueue(new MockResponse()
            .setBody("{\"id\":\"1337\",\"name\":\"Smith\"}")
            .addHeader("Content-Type", "application/json")
            .addHeader("ETag", "\"v7\""));
        server.enqueue(new MockResponse().setResponseCode(412));

        ContactEndpoint contact = client.getContacts().get("1337");
        Contact entity = contact.read();
        assertThrows(ConflictException.class, () -> contact.set(entity));

        server.takeRequest(); // GET
        assertEquals("\"v7\"", server.takeRequest().getHeader("If-Match"));
    }
    ```

=== "Kotlin"

    ```kotlin
    @Test
    fun reportsConflict() {
        server.enqueue(MockResponse()
            .setBody("""{"id":"1337","name":"Smith"}""")
            .addHeader("Content-Type", "application/json")
            .addHeader("ETag", "\"v7\""))
        server.enqueue(MockResponse().setResponseCode(412))

        val contact = client.contacts["1337"]
        val entity = contact.read()
        assertThrows<ConflictException> { contact.set(entity) }

        server.takeRequest() // GET
        assertEquals("\"v7\"", server.takeRequest().getHeader("If-Match"))
    }
    ```

=== "TypeScript"

    ```typescript
    import { ConcurrencyError } from "typedrest/errors";

    test("reports conflict", async () => {
        const client = new AddressBookClient(new URL("http://localhost/"));
        fetchMock.mockResponseOnce('{"id":"1337","name":"Smith"}', {
            headers: { "Content-Type": "application/json", "ETag": '"v7"' }
        });
        fetchMock.mockResponseOnce("", { status: 412 });

        const contact = client.contacts.get("1337");
        const entity = await contact.read();
        await expect(contact.set(entity)).rejects.toBeInstanceOf(ConcurrencyError);

        const [, init] = fetchMock.mock.calls[1];
        expect(new Headers(init?.headers).get("If-Match")).toBe('"v7"');
    });
    ```

## Test code that uses the client

When testing your own logic, the HTTP requests are a distraction. Replace the client with a [mock](../testing/mocking.md) instead and check that your code calls the right endpoint. Take this service, which pokes a new team member:

=== "C#"

    ```csharp
    public class TeamService(IAddressBookClient addressBook)
    {
        public async Task WelcomeAsync(string contactId, CancellationToken cancellationToken = default)
            => await addressBook.Contacts[contactId].Poke.InvokeAsync(cancellationToken);
    }
    ```

=== "Java"

    ```java
    public class TeamService {
        private final AddressBookClient addressBook;

        public TeamService(AddressBookClient addressBook) {
            this.addressBook = addressBook;
        }

        public void welcome(String contactId) {
            addressBook.getContacts().get(contactId).getPoke().invoke();
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    class TeamService(private val addressBook: AddressBookClient) {
        fun welcome(contactId: String) {
            addressBook.contacts[contactId].poke.invoke()
        }
    }
    ```

=== "TypeScript"

    ```typescript
    export class TeamService {
        constructor(private readonly addressBook: AddressBookClient) {}

        async welcome(contactId: string) {
            await this.addressBook.contacts.get(contactId).poke.invoke();
        }
    }
    ```

The test should check that `welcome` pokes the right contact, without any HTTP involved:

=== "C#"

    Moq can only mock interfaces and virtual members, so it needs interfaces all the way down the chain, as described in [Mocking a custom client](../testing/mocking.md#mocking-a-custom-client). The `IAddressBookClient` from [Dependency injection](dependency-injection.md#1-prepare-the-client) still hands out the `ContactEndpoint` class, whose `Poke` property Moq cannot set up. Give `ContactEndpoint` an interface and use it in `IAddressBookClient`:

    ```csharp
    public interface IContactEndpoint : IElementEndpoint<Contact>
    {
        IElementEndpoint<Note> Note { get; }
        IActionEndpoint Poke { get; }
    }

    public class ContactEndpoint(IEndpoint referrer, Uri relativeUri)
        : ElementEndpoint<Contact>(referrer, relativeUri), IContactEndpoint
    {
        public IElementEndpoint<Note> Note => new ElementEndpoint<Note>(this, relativeUri: "./note");
        public IActionEndpoint Poke => new ActionEndpoint(this, relativeUri: "./poke");
    }

    public interface IAddressBookClient : IEndpoint
    {
        ICollectionEndpoint<Contact, IContactEndpoint> Contacts { get; }
    }
    ```

    `AddressBookClient.Contacts` keeps creating a `CollectionEndpoint<Contact, ContactEndpoint>`; only its declared type changes to `ICollectionEndpoint<Contact, IContactEndpoint>`. Moq can then set up and verify the whole chain in one expression:

    ```csharp
    [Fact]
    public async Task PokesNewTeamMember()
    {
        var addressBook = new Mock<IAddressBookClient>();
        addressBook.Setup(x => x.Contacts["1337"].Poke.InvokeAsync(It.IsAny<CancellationToken>()))
                   .Returns(Task.CompletedTask);

        await new TeamService(addressBook.Object).WelcomeAsync("1337");

        addressBook.Verify(x => x.Contacts["1337"].Poke.InvokeAsync(It.IsAny<CancellationToken>()));
    }
    ```

=== "Java"

    With [Mockito](https://site.mockito.org/), stub each level of the chain:

    ```java
    @Test
    void pokesNewTeamMember() {
        ActionEndpoint poke = mock(ActionEndpoint.class);
        ContactEndpoint contact = mock(ContactEndpoint.class);
        when(contact.getPoke()).thenReturn(poke);
        GenericCollectionEndpoint<Contact, ContactEndpoint> contacts = mock(GenericCollectionEndpoint.class);
        when(contacts.get("1337")).thenReturn(contact);
        AddressBookClient addressBook = mock(AddressBookClient.class);
        when(addressBook.getContacts()).thenReturn(contacts);

        new TeamService(addressBook).welcome("1337");

        verify(poke).invoke();
    }
    ```

=== "Kotlin"

    With [mockito-kotlin](https://github.com/mockito/mockito-kotlin), stub each level of the chain:

    ```kotlin
    @Test
    fun pokesNewTeamMember() {
        val poke = mock<ActionEndpoint>()
        val contact = mock<ContactEndpoint> { on { this.poke } doReturn poke }
        val contacts = mock<GenericCollectionEndpoint<Contact, ContactEndpoint>> { on { get("1337") } doReturn contact }
        val addressBook = mock<AddressBookClient> { on { this.contacts } doReturn contacts }

        TeamService(addressBook).welcome("1337")

        verify(poke).invoke()
    }
    ```

=== "TypeScript"

    Any object with the members your code uses can stand in for the client:

    ```typescript
    test("pokes new team member", async () => {
        const poke = { invoke: jest.fn().mockResolvedValue(undefined) };
        const contacts = { get: jest.fn().mockReturnValue({ poke }) };
        const addressBook = { contacts } as unknown as AddressBookClient;

        await new TeamService(addressBook).welcome("1337");

        expect(contacts.get).toHaveBeenCalledWith("1337");
        expect(poke.invoke).toHaveBeenCalled();
    });
    ```

## Test client and service together

If you write the service as well as the client, test the two together: run the service inside the test process, replace its own dependencies such as its database with mocks, and point the `AddressBookClient` at it. This catches a route that differs between client and service, a status code the client does not expect, or a property the two serialize differently.

[Integration testing](../testing/integration.md) shows how to host the service with `WebApplicationFactory` in ASP.NET Core or `@SpringBootTest` in Spring Boot. [Sample-DotNet](https://github.com/TypedRest/Sample-DotNet) and [Sample-Kotlin](https://github.com/TypedRest/Sample-Kotlin) both have complete sets of tests like this for the address book API.
