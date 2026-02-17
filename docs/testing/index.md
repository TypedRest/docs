# Testing

## Introduction

When building custom endpoint classes with TypedRest, you'll want to write unit tests to ensure they interact with the HTTP layer correctly. The general testing approach is:

1. **Mock the HTTP layer** — Use a mock HTTP handler or server
2. **Create an `EntryEndpoint`** — Wire it to the mock HTTP infrastructure
3. **Instantiate your endpoint** — Create it as a child of the entry endpoint
4. **Set up expected responses** — Configure the mock to return appropriate HTTP responses
5. **Call the endpoint** — Invoke the method you're testing
6. **Assert requests and results** — Verify that the correct HTTP request was sent and the response was properly deserialized

## Test infrastructure

Each platform provides infrastructure for setting up tests with mocked HTTP:

=== "C#"

    TypedRest tests use xUnit with `[Fact]` attributes and an `EndpointTestBase` abstract class that provides mock HTTP infrastructure:

    ```csharp
    using System.Net.Http;
    using RichardSzalay.MockHttp;
    using TypedRest;
    using Xunit;

    public abstract class EndpointTestBase : IDisposable
    {
        public const string JsonMime = "application/json";
        protected readonly MockHttpMessageHandler Mock = new();
        protected readonly IEndpoint EntryEndpoint;

        protected EndpointTestBase(MediaTypeFormatter? serializer = null)
        {
            EntryEndpoint = new MockEntryEndpoint(Mock, serializer);
        }

        public void Dispose() => Mock.VerifyNoOutstandingExpectation();

        private class MockEntryEndpoint(HttpMessageHandler messageHandler, MediaTypeFormatter? serializer)
            : EntryEndpoint(new HttpClient(messageHandler), new Uri("http://localhost/"), serializer);
    }
    ```

    This uses the [RichardSzalay.MockHttp](https://www.nuget.org/packages/RichardSzalay.MockHttp/) package to mock HTTP requests and [FluentAssertions](https://fluentassertions.com/) for assertions.

=== "Java"

    TypedRest tests use JUnit with `@Test` annotations and an `AbstractEndpointTest` abstract class that provides mock HTTP infrastructure:

    ```java
    import net.typedrest.EntryEndpoint;
    import okhttp3.mockwebserver.MockWebServer;
    import org.junit.jupiter.api.AfterEach;

    abstract class AbstractEndpointTest {
        protected MockWebServer server = new MockWebServer();
        protected EntryEndpoint entryEndpoint = new EntryEndpoint(server.url("/").uri());

        @AfterEach
        void after() throws Exception {
            server.close();
        }
    }
    ```

    This uses [OkHttp's MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) to mock HTTP responses.

=== "Kotlin"

    TypedRest tests use Kotlin Test with `@Test` annotations and an `AbstractEndpointTest` abstract class that provides mock HTTP infrastructure:

    ```kotlin
    import net.typedrest.EntryEndpoint
    import okhttp3.mockwebserver.MockWebServer
    import kotlin.test.AfterTest

    abstract class AbstractEndpointTest {
        protected var server: MockWebServer = MockWebServer()
        protected var entryEndpoint: EntryEndpoint = EntryEndpoint(server.url("/").toUri())

        @AfterTest
        fun after() = server.close()
    }
    ```

    This uses [OkHttp's MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) to mock HTTP responses.

=== "TypeScript"

    TypedRest tests use [Jest](https://jestjs.io/) with `test()` and `expect()`, and [jest-fetch-mock](https://www.npmjs.com/package/jest-fetch-mock) to mock the global `fetch` function:

    ```typescript
    import fetchMock from 'jest-fetch-mock';
    import {EntryEndpoint, ElementEndpoint} from 'typedrest';

    fetchMock.enableMocks();

    let endpoint: ElementEndpoint<MockEntity>;

    beforeEach(() => {
        fetchMock.resetMocks();
        endpoint = new ElementEndpoint(new EntryEndpoint('http://localhost/'), 'endpoint');
    });
    ```

## Example: Testing ElementEndpoint

Here's a complete example of testing an `ElementEndpoint`'s `read` method:

=== "C#"

    ```csharp
    using System.Net.Http;
    using FluentAssertions;
    using RichardSzalay.MockHttp;
    using TypedRest;
    using Xunit;

    public class ElementEndpointTest : EndpointTestBase
    {
        private readonly ElementEndpoint<MockEntity> _endpoint;

        public ElementEndpointTest()
        {
            _endpoint = new ElementEndpoint<MockEntity>(EntryEndpoint, relativeUri: "./endpoint");
        }

        [Fact]
        public async Task TestRead()
        {
            Mock.Expect(HttpMethod.Get, "http://localhost/endpoint")
                .Respond(JsonMime, """{"id":5,"name":"test"}""");

            var result = await _endpoint.ReadAsync();

            result.Should().Be(new MockEntity { Id = 5, Name = "test" });
        }
    }
    ```

=== "Java"

    ```java
    import net.typedrest.ElementEndpoint;
    import net.typedrest.ElementEndpointImpl;
    import okhttp3.mockwebserver.MockResponse;
    import org.junit.jupiter.api.Test;

    import static net.typedrest.tests.MockWebServerExtensions.*;
    import static org.junit.jupiter.api.Assertions.*;

    class ElementEndpointTest extends AbstractEndpointTest {
        private ElementEndpoint<MockEntity> endpoint;

        @BeforeEach
        void setUp() {
            endpoint = new ElementEndpointImpl<>(entryEndpoint, "./endpoint", MockEntity.class);
        }

        @Test
        void testRead() throws Exception {
            server.enqueue(new MockResponse().setJsonBody("""{"id":5,"name":"test"}"""));

            MockEntity result = endpoint.read();

            assertEquals(5, result.id);
            assertEquals("test", result.name);
            server.assertRequest(HttpMethod.GET).withPath("/endpoint");
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    import net.typedrest.ElementEndpoint
    import net.typedrest.ElementEndpointImpl
    import okhttp3.mockwebserver.MockResponse
    import kotlin.test.Test
    import kotlin.test.assertEquals

    class ElementEndpointTest : AbstractEndpointTest() {
        private lateinit var endpoint: ElementEndpoint<MockEntity>

        @BeforeTest
        fun setUp() {
            endpoint = ElementEndpointImpl(entryEndpoint, "./endpoint", MockEntity::class.java)
        }

        @Test
        fun testRead() {
            server.enqueue(MockResponse().setJsonBody("""{"id":5,"name":"test"}"""))

            val result = endpoint.read()

            assertEquals(5, result.id)
            assertEquals("test", result.name)
            server.assertRequest(HttpMethod.GET).withPath("/endpoint")
        }
    }
    ```

=== "TypeScript"

    ```typescript
    import fetchMock from 'jest-fetch-mock';
    import {EntryEndpoint, ElementEndpoint} from 'typedrest';

    fetchMock.enableMocks();

    let endpoint: ElementEndpoint<MockEntity>;

    beforeEach(() => {
        fetchMock.resetMocks();
        endpoint = new ElementEndpoint(new EntryEndpoint('http://localhost/'), 'endpoint');
    });

    test('read', async () => {
        fetchMock.mockOnceIf(
            'http://localhost/endpoint',
            JSON.stringify({id: 5, name: 'test'}),
            {status: 200}
        );

        const result = await endpoint.read();

        expect(result).toEqual({id: 5, name: 'test'});
    });
    ```

## Example: Testing a custom endpoint

Here's an example of testing a custom endpoint class with additional functionality:

=== "C#"

    ```csharp
    using System.Net.Http;
    using FluentAssertions;
    using RichardSzalay.MockHttp;
    using TypedRest;
    using Xunit;

    // Custom endpoint class
    public class ContactEndpoint(IEndpoint referrer, Uri relativeUri)
        : ElementEndpoint<Contact>(referrer, relativeUri)
    {
        public ElementEndpoint<Note> Note => new(this, relativeUri: "./note");
    }

    // Test class
    public class ContactEndpointTest : EndpointTestBase
    {
        private readonly ContactEndpoint _endpoint;

        public ContactEndpointTest()
        {
            _endpoint = new ContactEndpoint(EntryEndpoint, new Uri("./contact", UriKind.Relative));
        }

        [Fact]
        public async Task TestReadNote()
        {
            Mock.Expect(HttpMethod.Get, "http://localhost/contact/note")
                .Respond(JsonMime, """{"content":"Test note"}""");

            var result = await _endpoint.Note.ReadAsync();

            result.Should().Be(new Note { Content = "Test note" });
        }
    }
    ```

=== "Java"

    ```java
    import net.typedrest.Endpoint;
    import net.typedrest.ElementEndpoint;
    import net.typedrest.ElementEndpointImpl;
    import okhttp3.mockwebserver.MockResponse;
    import org.junit.jupiter.api.Test;

    import java.net.URI;

    import static org.junit.jupiter.api.Assertions.*;

    // Custom endpoint class
    class ContactEndpoint extends ElementEndpointImpl<Contact> {
        public ContactEndpoint(Endpoint referrer, URI relativeUri) {
            super(referrer, relativeUri, Contact.class);
        }

        public ElementEndpoint<Note> getNote() {
            return new ElementEndpointImpl<>(this, "./note", Note.class);
        }
    }

    // Test class
    class ContactEndpointTest extends AbstractEndpointTest {
        private ContactEndpoint endpoint;

        @BeforeEach
        void setUp() {
            endpoint = new ContactEndpoint(entryEndpoint, URI.create("./contact"));
        }

        @Test
        void testReadNote() throws Exception {
            server.enqueue(new MockResponse().setJsonBody("""{"content":"Test note"}"""));

            Note result = endpoint.getNote().read();

            assertEquals("Test note", result.content);
            server.assertRequest(HttpMethod.GET).withPath("/contact/note");
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    import net.typedrest.Endpoint
    import net.typedrest.ElementEndpoint
    import net.typedrest.ElementEndpointImpl
    import okhttp3.mockwebserver.MockResponse
    import java.net.URI
    import kotlin.test.Test
    import kotlin.test.assertEquals

    // Custom endpoint class
    class ContactEndpoint(referrer: Endpoint, relativeUri: URI) :
        ElementEndpointImpl<Contact>(referrer, relativeUri, Contact::class.java) {
        val note: ElementEndpoint<Note>
            get() = ElementEndpointImpl(this, "./note", Note::class.java)
    }

    // Test class
    class ContactEndpointTest : AbstractEndpointTest() {
        private lateinit var endpoint: ContactEndpoint

        @BeforeTest
        fun setUp() {
            endpoint = ContactEndpoint(entryEndpoint, URI.create("./contact"))
        }

        @Test
        fun testReadNote() {
            server.enqueue(MockResponse().setJsonBody("""{"content":"Test note"}"""))

            val result = endpoint.note.read()

            assertEquals("Test note", result.content)
            server.assertRequest(HttpMethod.GET).withPath("/contact/note")
        }
    }
    ```

=== "TypeScript"

    ```typescript
    import fetchMock from 'jest-fetch-mock';
    import {EntryEndpoint, ElementEndpoint, Endpoint} from 'typedrest';

    fetchMock.enableMocks();

    // Custom endpoint class
    class ContactEndpoint extends ElementEndpoint<Contact> {
        get note() {
            return new ElementEndpoint<Note>(this, './note');
        }
    }

    // Test setup
    let endpoint: ContactEndpoint;

    beforeEach(() => {
        fetchMock.resetMocks();
        endpoint = new ContactEndpoint(new EntryEndpoint('http://localhost/'), 'contact');
    });

    test('read note', async () => {
        fetchMock.mockOnceIf(
            'http://localhost/contact/note',
            JSON.stringify({content: 'Test note'}),
            {status: 200}
        );

        const result = await endpoint.note.read();

        expect(result).toEqual({content: 'Test note'});
    });
    ```

## More examples

For more comprehensive examples and advanced testing patterns, check out the test suites in the TypedRest implementations:

- [TypedRest-DotNet tests](https://github.com/TypedRest/TypedRest-DotNet/tree/master/src/UnitTests)
- [TypedRest-Java tests](https://github.com/TypedRest/TypedRest-Java/tree/main/typedrest/src/test)
- [TypedRest-TypeScript tests](https://github.com/TypedRest/TypedRest-TypeScript/tree/master/test)
