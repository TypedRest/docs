# Entry endpoint

Represent the top-level URI of an API. Used to address the various resources of the API.

The constructor of the entry endpoint requires you to specify the APIs root URL. It also provides optional parameters to override the default HTTP client, JSON serializer, [error handler](../error-handling/index.md) and [link handler](../link-handling/index.md).

## Usage

=== "C#"

    ```csharp
    // Create an entry endpoint
    var client = new EntryEndpoint(new Uri("http://example.com/api/"));

    // Create child endpoint
    var contacts = new CollectionEndpoint<Contact>(client, "contacts");
    ```

=== "Java"

    ```java
    // Create an entry endpoint
    EntryEndpoint client = new EntryEndpoint(URI.create("http://example.com/api/"));

    // Create child endpoint
    CollectionEndpoint<Contact> contacts = new CollectionEndpoint<>(client, "contacts", Contact.class);
    ```

=== "Kotlin"

    ```kotlin
    // Create an entry endpoint
    val client = EntryEndpoint(URI.create("http://example.com/api/"))

    // Create child endpoint
    val contacts = CollectionEndpoint(client, "contacts", Contact::class.java)
    ```

=== "TypeScript"

    ```typescript
    // Create an entry endpoint
    const client = new EntryEndpoint(new URL("http://example.com/api/"));

    // Create child endpoint
    const contacts = new CollectionEndpoint<Contact>(client, "contacts");
    ```
