# Java / Kotlin

## Getting started

### 1. Install the dependency

Add the [typedrest](https://central.sonatype.com/artifact/net.typedrest/typedrest) Maven artifact to your project.

=== "Maven"

    ```xml
    <dependency>
        <groupId>net.typedrest</groupId>
        <artifactId>typedrest</artifactId>
        <version>VERSION</version>
    </dependency>
    ```

=== "Gradle"

    ```groovy
    implementation 'net.typedrest:typedrest:VERSION'
    ```

### 2. Create an entry endpoint

Create an `EntryEndpoint` pointing at your API:

=== "Java"

    ```java
    import net.typedrest.EntryEndpoint;
    import java.net.URI;

    EntryEndpoint client = new EntryEndpoint(URI.create("https://example.com/api/"));
    ```

=== "Kotlin"

    ```kotlin
    import net.typedrest.EntryEndpoint
    import java.net.URI

    val client = EntryEndpoint(URI.create("https://example.com/api/"))
    ```

### 3. Define your client class

Extend `EntryEndpoint` and expose your API's resources as methods or properties:

=== "Java"

    ```java
    import net.typedrest.CollectionEndpoint;
    import net.typedrest.CollectionEndpointImpl;

    class MyClient extends EntryEndpoint {
        public MyClient(URI uri) {
            super(uri);
        }

        public CollectionEndpoint<Contact> getContacts() {
            return new CollectionEndpointImpl<>(this, "./contacts", Contact.class);
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    import net.typedrest.CollectionEndpoint
    import net.typedrest.CollectionEndpointImpl

    class MyClient(uri: URI) : EntryEndpoint(uri) {
        val contacts: CollectionEndpoint<Contact>
            get() = CollectionEndpointImpl(this, "./contacts", Contact::class.java)
    }
    ```

### 4. Use the client

Use your client to interact with the API:

=== "Java"

    ```java
    MyClient client = new MyClient(URI.create("https://example.com/api/"));

    // Read all contacts
    List<Contact> contacts = client.getContacts().readAll();

    // Create a new contact
    ElementEndpoint<Contact> newContact = client.getContacts().create(new Contact("Smith"));

    // Read a specific contact
    Contact contact = newContact.read();
    ```

=== "Kotlin"

    ```kotlin
    val client = MyClient(URI.create("https://example.com/api/"))

    // Read all contacts
    val contacts: List<Contact> = client.contacts.readAll()

    // Create a new contact
    val newContact: ElementEndpoint<Contact> = client.contacts.create(Contact("Smith"))

    // Read a specific contact
    val contact: Contact = newContact.read()
    ```

### 5. Next steps

- Read the [Introduction](../introduction.md) for the full conceptual walkthrough
- Explore all available [Endpoints](../endpoints/index.md) types
- Check out the [API documentation](https://java.typedrest.net/) for detailed reference

## Additional packages

[typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive)  
Adds support for streaming with [ReactiveX (Rx)](http://reactivex.io/).  
Create endpoints using the types in the `net.typedrest.endpoints.reactive` package.

[typedrest-serializers-jackson](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson)  
Adds support for serializing using [Jackson](https://github.com/FasterXML/jackson) instead of [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html).  
Pass `new JacksonJsonSerializer()` to the `EntryEndpoint` constructor.

[typedrest-serializers-moshi](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-moshi)  
Adds support for serializing using [Moshi](https://github.com/square/moshi) instead of [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html).  
Pass `new MoshiJsonSerializer()` to the `EntryEndpoint` constructor.

## See also

- [API documentation](https://java.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/TypedRest-Java)
