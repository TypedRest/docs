# Authentication

Every endpoint in a client sends its requests through the [HTTP client](../endpoints/entry.md#http-client) of the entry endpoint. Authentication therefore goes in one place: configure the entry endpoint or its HTTP client, and every request from the client is authenticated.

This tutorial extends the `AddressBookClient` from [Build your first client](first-client.md) with each common scheme, from the simplest to the most involved.

## HTTP Basic authentication

Pass a username and password when creating the entry endpoint:

=== "C#"

    ```csharp
    public class AddressBookClient(Uri uri, NetworkCredential? credentials = null)
        : EntryEndpoint(uri, credentials)
    {
        // ...
    }

    var client = new AddressBookClient(new Uri("https://example.com/api/"), new NetworkCredential("alice", "secret"));
    ```

=== "Java"

    ```java
    public class AddressBookClient extends EntryEndpoint {
        public AddressBookClient(URI uri, HttpCredentials credentials) {
            super(uri, credentials, new JacksonJsonSerializer());
        }

        // ...
    }

    AddressBookClient client = new AddressBookClient(URI.create("https://example.com/api/"), new HttpCredentials("alice", "secret"));
    ```

=== "Kotlin"

    ```kotlin
    class AddressBookClient(uri: URI, credentials: HttpCredentials? = null) : EntryEndpoint(uri, credentials) {
        // ...
    }

    val client = AddressBookClient(URI.create("https://example.com/api/"), HttpCredentials("alice", "secret"))
    ```

=== "TypeScript"

    The TypeScript entry endpoint has no credentials parameter. Set the header on its HTTP client instead:

    ```typescript
    const client = new AddressBookClient(new URL("https://example.com/api/"));
    client.httpClient.defaultHeaders.set("Authorization", `Basic ${btoa("alice:secret")}`);
    ```

In .NET, Java and Kotlin you can also [put the credentials in the URI](../endpoints/entry.md#http-basic-authentication).

## API keys and static bearer tokens

Many APIs take a fixed key or token in a header. Add the header to the HTTP client once:

=== "C#"

    ```csharp
    var client = new AddressBookClient(new Uri("https://example.com/api/"));
    client.HttpClient.DefaultRequestHeaders.Authorization = new("Bearer", token);

    // or, for a custom header:
    client.HttpClient.DefaultRequestHeaders.Add("X-Api-Key", apiKey);
    ```

=== "Java"

    TypedRest for the JVM uses [OkHttp](https://square.github.io/okhttp/). Add an interceptor that sets the header, and pass the configured `OkHttpClient` to the entry endpoint:

    ```java
    public class AddressBookClient extends EntryEndpoint {
        public AddressBookClient(URI uri, OkHttpClient httpClient) {
            super(uri, httpClient, new JacksonJsonSerializer());
        }

        // ...
    }

    OkHttpClient httpClient = new OkHttpClient.Builder()
        .addInterceptor(chain -> chain.proceed(chain.request().newBuilder()
            .header("Authorization", "Bearer " + token)
            .build()))
        .build();
    AddressBookClient client = new AddressBookClient(URI.create("https://example.com/api/"), httpClient);
    ```

=== "Kotlin"

    TypedRest for the JVM uses [OkHttp](https://square.github.io/okhttp/). Add an interceptor that sets the header, and pass the configured `OkHttpClient` to the entry endpoint:

    ```kotlin
    class AddressBookClient(uri: URI, httpClient: OkHttpClient) : EntryEndpoint(uri, httpClient) {
        // ...
    }

    val httpClient = OkHttpClient.Builder()
        .addInterceptor { chain ->
            chain.proceed(chain.request().newBuilder()
                .header("Authorization", "Bearer $token")
                .build())
        }
        .build()
    val client = AddressBookClient(URI.create("https://example.com/api/"), httpClient)
    ```

=== "TypeScript"

    ```typescript
    const client = new AddressBookClient(new URL("https://example.com/api/"));
    client.httpClient.defaultHeaders.set("Authorization", `Bearer ${token}`);

    // or, for a custom header:
    client.httpClient.defaultHeaders.set("X-Api-Key", apiKey);
    ```

Child endpoints share the entry endpoint's HTTP client, so this covers every request the client makes.

## OAuth 2.0 tokens that expire

Tokens from an OAuth 2.0 identity server expire, so a fixed header is not enough. The client has to request a token, reuse it while it is valid, and request a new one before or when it expires.

=== "C#"

    The [TypedRest.OAuth](https://www.nuget.org/packages/TypedRest.OAuth/) package does all of this for the [client credentials flow](https://oauth.net/2/grant-types/client-credentials/), the usual flow for service-to-service calls:

    ```bash
    dotnet add package TypedRest.OAuth
    ```

    Give the client a constructor that takes an `HttpClient`, and build that `HttpClient` with an `OAuthHandler`:

    ```csharp
    public class AddressBookClient : EntryEndpoint
    {
        public AddressBookClient(Uri uri) : base(uri) {}

        public AddressBookClient(HttpClient httpClient) : base(httpClient) {}

        // ...
    }

    var handler = new OAuthHandler(
        new OAuthOptions
        {
            Uri = new Uri("https://identity.example.com/"),
            ClientId = "address-book-sync",
            ClientSecret = clientSecret,
            Scope = "contacts"
        },
        innerHandler: new HttpClientHandler());

    var client = new AddressBookClient(new HttpClient(handler) { BaseAddress = new Uri("https://example.com/api/") });
    ```

    The handler finds the token endpoint on the identity server, reuses each token until shortly before it expires, and retries once with a new token if the server rejects one. [OAuth 2.0](../setup/dotnet.md#oauth-20) describes this in detail and lists all options.

    In ASP.NET Core and other applications that use `IServiceCollection`, call `.AddOAuthHandler()` instead of building the handler yourself; see [Dependency injection](dependency-injection.md#5-add-authentication).

=== "Java"

    TypedRest for the JVM has no built-in OAuth support. An OkHttp interceptor of your own can cache the token and refresh it on a `401 Unauthorized`:

    ```java
    public record AccessToken(String value, Instant expiresAt) {}

    public class OAuthInterceptor implements Interceptor {
        private final Supplier<AccessToken> tokenSource;
        private AccessToken token;

        public OAuthInterceptor(Supplier<AccessToken> tokenSource) {
            this.tokenSource = tokenSource;
        }

        private synchronized String getToken(boolean refresh) {
            if (refresh || token == null || Instant.now().isAfter(token.expiresAt().minusSeconds(60))) {
                token = tokenSource.get();
            }
            return token.value();
        }

        @Override
        public Response intercept(Chain chain) throws IOException {
            Response response = chain.proceed(withBearer(chain.request(), getToken(false)));
            if (response.code() != 401) return response;

            response.close();
            return chain.proceed(withBearer(chain.request(), getToken(true)));
        }

        private static Request withBearer(Request request, String token) {
            return request.newBuilder().header("Authorization", "Bearer " + token).build();
        }
    }

    OkHttpClient httpClient = new OkHttpClient.Builder()
        .addInterceptor(new OAuthInterceptor(() -> requestTokenFromIdentityServer()))
        .build();
    AddressBookClient client = new AddressBookClient(URI.create("https://example.com/api/"), httpClient);
    ```

    `requestTokenFromIdentityServer()` stands for whatever obtains a token in your setup, such as a call to your identity server's token endpoint or an OAuth library like [Nimbus](https://connect2id.com/products/nimbus-oauth-openid-connect-sdk).

=== "Kotlin"

    TypedRest for the JVM has no built-in OAuth support. An OkHttp interceptor of your own can cache the token and refresh it on a `401 Unauthorized`:

    ```kotlin
    data class AccessToken(val value: String, val expiresAt: Instant)

    class OAuthInterceptor(private val tokenSource: () -> AccessToken) : Interceptor {
        private var token: AccessToken? = null

        @Synchronized
        private fun getToken(refresh: Boolean): String {
            val current = token
            if (!refresh && current != null && Instant.now() < current.expiresAt.minusSeconds(60)) return current.value
            return tokenSource().also { token = it }.value
        }

        override fun intercept(chain: Interceptor.Chain): Response {
            val response = chain.proceed(chain.request().withBearer(getToken(refresh = false)))
            if (response.code != 401) return response

            response.close()
            return chain.proceed(chain.request().withBearer(getToken(refresh = true)))
        }

        private fun Request.withBearer(token: String) =
            newBuilder().header("Authorization", "Bearer $token").build()
    }

    val httpClient = OkHttpClient.Builder()
        .addInterceptor(OAuthInterceptor { requestTokenFromIdentityServer() })
        .build()
    val client = AddressBookClient(URI.create("https://example.com/api/"), httpClient)
    ```

    `requestTokenFromIdentityServer()` stands for whatever obtains a token in your setup, such as a call to your identity server's token endpoint or an OAuth library like [Nimbus](https://connect2id.com/products/nimbus-oauth-openid-connect-sdk).

=== "TypeScript"

    TypedRest for TypeScript has no built-in OAuth support. Derive from `FetchHttpClient` to add the token to each request and refresh it on a `401 Unauthorized`:

    ```typescript
    import { FetchHttpClient, HttpMethod } from "typedrest/http";

    interface AccessToken {
        value: string;
        expiresAt: number; // milliseconds since the epoch
    }

    export class OAuthHttpClient extends FetchHttpClient {
        private token?: AccessToken;

        constructor(private readonly tokenSource: () => Promise<AccessToken>) {
            super();
        }

        async send(uri: URL, method: HttpMethod, signal?: AbortSignal, headers?: HeadersInit, body?: BodyInit) {
            const response = await super.send(uri, method, signal, await this.withBearer(headers), body);
            if (response.status !== 401) return response;

            this.token = undefined;
            return super.send(uri, method, signal, await this.withBearer(headers), body);
        }

        private async withBearer(headers?: HeadersInit) {
            if (!this.token || Date.now() >= this.token.expiresAt - 60_000) {
                this.token = await this.tokenSource();
            }
            const result = new Headers(headers);
            result.set("Authorization", `Bearer ${this.token.value}`);
            return result;
        }
    }

    // The HTTP client is the fifth argument of the entry endpoint constructor
    const client = new AddressBookClient(
        new URL("https://example.com/api/"), undefined, undefined, undefined,
        new OAuthHttpClient(() => requestTokenFromIdentityServer()));
    ```

    `requestTokenFromIdentityServer()` stands for whatever obtains a token in your setup. In a browser, that is usually the library handling the user's login, such as [oidc-client-ts](https://github.com/authts/oidc-client-ts).

!!! warning "Retrying requests with streamed bodies"
    The retry after a `401` sends the request body a second time. That is fine for the serialized entities TypedRest sends, but a body that can only be read once, such as a stream passed to an [upload endpoint](../endpoints/raw/upload.md), cannot be replayed. The .NET `OAuthHandler` skips the retry for such requests; do the same if your own implementation handles uploads.

## Handling authentication errors

When the server rejects the credentials or the caller lacks permissions, TypedRest throws an exception you can catch:

| Status               | Meaning                       | .NET                          | Java/Kotlin               | TypeScript            |
| -------------------- | ----------------------------- | ----------------------------- | ------------------------- | --------------------- |
| **401 Unauthorized** | missing or invalid credentials | `AuthenticationException`     | `AuthenticationException` | `AuthenticationError` |
| **403 Forbidden**    | valid credentials, no permission | `UnauthorizedAccessException` | `AuthorizationException`  | `AuthorizationError`  |

=== "C#"

    ```csharp
    try
    {
        await client.Contacts["1337"].DeleteAsync();
    }
    catch (AuthenticationException)
    {
        // Ask the user to log in again
    }
    catch (UnauthorizedAccessException)
    {
        Console.WriteLine("You are not allowed to delete contacts.");
    }
    ```

=== "Java"

    ```java
    try {
        client.getContacts().get("1337").delete();
    } catch (AuthenticationException ex) {
        // Ask the user to log in again
    } catch (AuthorizationException ex) {
        System.out.println("You are not allowed to delete contacts.");
    }
    ```

=== "Kotlin"

    ```kotlin
    try {
        client.contacts["1337"].delete()
    } catch (ex: AuthenticationException) {
        // Ask the user to log in again
    } catch (ex: AuthorizationException) {
        println("You are not allowed to delete contacts.")
    }
    ```

=== "TypeScript"

    ```typescript
    import { AuthenticationError, AuthorizationError } from "typedrest/errors";

    try {
        await client.contacts.get("1337").delete();
    } catch (err) {
        if (err instanceof AuthenticationError) {
            // Ask the user to log in again
        } else if (err instanceof AuthorizationError) {
            console.log("You are not allowed to delete contacts.");
        } else {
            throw err;
        }
    }
    ```

Instead of handling `403` after the fact, you can often ask first. Endpoints record the [`Allow` header](../endpoints/index.md#allowed-methods) of each response, so after reading a contact you can check on the same endpoint object whether deleting is allowed (`DeleteAllowed` in .NET, `isDeleteAllowed` in Java/Kotlin, `deleteAllowed` in TypeScript) and hide the delete button if it is not.

## Next steps

- [Dependency injection](dependency-injection.md): configure authentication for a client registered in ASP.NET Core.
- [Command-line tool](command-line.md): CLI clients that prompt for and cache a token.
