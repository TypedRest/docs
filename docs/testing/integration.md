# Integration testing

If you write the service as well as the client, test the two together. This catches the mistakes neither [endpoint tests](endpoints.md) nor [mocks](mocking.md) can: a route that differs between client and service, a status code the client does not expect, or a property the two serialize differently.

Run the service inside the test process and point the client at it. Replace the service's own dependencies, such as its database, with mocks, so the test only covers the HTTP layer.

=== "C#"

    Use [WebApplicationFactory](https://learn.microsoft.com/aspnet/core/test/integration-tests) from `Microsoft.AspNetCore.Mvc.Testing`, which hosts the service in memory and provides an `HttpClient` connected to it. Give your client a constructor that passes an `HttpClient` on to the entry endpoint:

    ```csharp
    public class MyClient : EntryEndpoint
    {
        public MyClient(Uri uri) : base(uri) {}

        public MyClient(HttpClient httpClient) : base(httpClient) {}

        // Child endpoints
    }
    ```

    The entry endpoint then uses the `HttpClient`'s `BaseAddress` as its URI:

    ```csharp
    public class ContactsApiFacts(WebApplicationFactory<Program> factory) : IClassFixture<WebApplicationFactory<Program>>
    {
        private readonly Mock<IContactsService> _service = new();

        private MyClient CreateClient()
            => new(factory
                .WithWebHostBuilder(x => x.ConfigureTestServices(services => services.AddSingleton(_service.Object)))
                .CreateClient());

        [Fact]
        public async Task ReadsContact()
        {
            _service.Setup(x => x.ReadAsync("1337")).ReturnsAsync(new Contact { Id = "1337", Name = "Smith" });

            var contact = await CreateClient().Contacts["1337"].ReadAsync();

            Assert.Equal("Smith", contact.Name);
        }
    }
    ```

    `IContactsService` stands for whatever the service's controllers depend on. If the service uses top-level statements, add `public partial class Program;` to it so the test can reference `Program`.

    [Sample-DotNet](https://github.com/TypedRest/Sample-DotNet) has a complete set of tests like this, using `TestServer` directly.

=== "Java"

    Use `@SpringBootTest` to start the service on a random port, and `@MockitoBean` to replace its dependencies:

    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    class ContactsApiTest {
        @LocalServerPort
        private int port;

        @MockitoBean
        private ContactsService service;

        @Test
        void readsContact() {
            Contact smith = new Contact("Smith");
            smith.setId("1337");
            when(service.read("1337")).thenReturn(smith);
            MyClient client = new MyClient(URI.create("http://localhost:" + port + "/"));

            Contact contact = client.getContacts().get("1337").read();

            assertEquals("Smith", contact.getName());
        }
    }
    ```

    `ContactsService` stands for whatever the service's controllers depend on.

=== "Kotlin"

    Use `@SpringBootTest` to start the service on a random port, and `@MockitoBean` to replace its dependencies:

    ```kotlin
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    class ContactsApiTest {
        @LocalServerPort
        private var port: Int = 0

        @MockitoBean
        private lateinit var service: ContactsService

        @Test
        fun readsContact() {
            whenever(service.read("1337")).thenReturn(Contact(id = "1337", name = "Smith"))
            val client = MyClient(URI("http://localhost:$port/"))

            val contact = client.contacts["1337"].read()

            assertEquals("Smith", contact.name)
        }
    }
    ```

    `ContactsService` stands for whatever the service's controllers depend on. [Sample-Kotlin](https://github.com/TypedRest/Sample-Kotlin) has a complete set of tests like this.

=== "TypeScript"

    If the service is written in another language, test the TypeScript client against a running instance of it, for example one started in your CI pipeline, or rely on [endpoint tests](endpoints.md) with a mock server.

Sharing the entity classes between client and service, as the sample projects do with a separate DTO project, rules out a whole class of mismatches before any test runs.
