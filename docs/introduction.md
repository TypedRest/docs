# Introduction

There have been innumerable papers, posts and articles providing guidelines on how to design RESTful web services. Most of these guidelines are intended to make the behavior of web services more predictable for humans and machines alike. For example, the appropriate use of HTTP verbs allows developers to easily distinguish safe operations (such as `GET` and `HEAD`) from unsafe operations (such as `POST` and `DELETE`). Intermediate proxy servers can use this same information to determine candidates for caching. The plethora of HTTP headers provide many additional ways to express metadata and API structure in a standardized form.

## The "old" way

While this slowly evolving consensus on "the right way" to design new APIs makes understanding such APIs easier, it does not necessarily help when actually consuming them. While HTTP libraries and tools expose all the underlying components (HTTP verbs/methods, headers, etc.) the burden of actually interpreting and combining all the pieces still lies with the developer. Let us take a look at an example:

Assume the URI `/contacts/` represents a collection of address book entries. `GET`ing the collection itself returns all entries. `GET`ing an URI like `/contacts/1337` gives you a specific entry. `POST`ing to the collection adds a new element to it and returns the URI of the newly created resource using the `Location` header.

Pretty standard stuff, right? The problem is that all this knowledge currently only exists in your head. When you actually perform operations on the collection in your code you are usually manually building your `GET`s and your `POST`s while serializing and deserializing message bodies to and from JSON.

=== "C#"

    ```csharp
    var client = new HttpClient {BaseAddress = new Uri("http://example.com/")};

    var contactsResponse = await client.GetAsync("contacts");
    var contactList = await contactsResponse.Content.ReadAsAsync<List<Contact>>();

    var createResponse = await client.PostAsJsonAsync("contacts", new Contact {Name = "Smith"});
    var contactUri = createResponse.Headers.Location;
    //var contactUri = new Uri("contacts/1337", UriKind.Relative);

    var contactResponse = await client.GetAsync(contactUri);
    var contact = await contactResponse.Content.ReadAsAsync<Contact>();
    ```

=== "Java"

    ```java
    HttpClient client = HttpClient.newBuilder()
        .build();

    HttpRequest contactsRequest = HttpRequest.newBuilder()
        .uri(URI.create("http://example.com/contacts"))
        .GET()
        .build();
    HttpResponse<String> contactsResponse = client.send(contactsRequest, HttpResponse.BodyHandlers.ofString());
    List<Contact> contactList = objectMapper.readValue(contactsResponse.body(), new TypeReference<List<Contact>>() {});

    String requestBody = objectMapper.writeValueAsString(new Contact("Smith"));
    HttpRequest createRequest = HttpRequest.newBuilder()
        .uri(URI.create("http://example.com/contacts"))
        .POST(HttpRequest.BodyPublishers.ofString(requestBody))
        .build();
    HttpResponse<String> createResponse = client.send(createRequest, HttpResponse.BodyHandlers.ofString());
    URI contactUri = URI.create(createResponse.headers().firstValue("Location").get());
    //URI contactUri = URI.create("contacts/1337");

    HttpRequest contactRequest = HttpRequest.newBuilder()
        .uri(contactUri)
        .GET()
        .build();
    HttpResponse<String> contactResponse = client.send(contactRequest, HttpResponse.BodyHandlers.ofString());
    Contact contact = objectMapper.readValue(contactResponse.body(), Contact.class);
    ```

=== "Kotlin"

    ```kotlin
    val client = HttpClient.newBuilder()
        .build()

    val contactsRequest = HttpRequest.newBuilder()
        .uri(URI.create("http://example.com/contacts"))
        .GET()
        .build()
    val contactsResponse = client.send(contactsRequest, HttpResponse.BodyHandlers.ofString())
    val contactList = objectMapper.readValue<List<Contact>>(contactsResponse.body())

    val requestBody = objectMapper.writeValueAsString(Contact("Smith"))
    val createRequest = HttpRequest.newBuilder()
        .uri(URI.create("http://example.com/contacts"))
        .POST(HttpRequest.BodyPublishers.ofString(requestBody))
        .build()
    val createResponse = client.send(createRequest, HttpResponse.BodyHandlers.ofString())
    val contactUri = URI.create(createResponse.headers().firstValue("Location").get())
    //val contactUri = URI.create("contacts/1337")

    val contactRequest = HttpRequest.newBuilder()
        .uri(contactUri)
        .GET()
        .build()
    val contactResponse = client.send(contactRequest, HttpResponse.BodyHandlers.ofString())
    val contact = objectMapper.readValue<Contact>(contactResponse.body())
    ```

=== "TypeScript"

    ```typescript
    const contactsResponse = await fetch("http://example.com/contacts");
    const contactList = (await contactsResponse.json()) as Contact[];

    const createResponse = await fetch("http://example.com/contacts", {
      body: JSON.stringify({name: "Smith"})
    });
    const contactUri = createResponse.headers.get("Location");
    //const contactUri = "http://example.com/contacts/1337"

    const contactResponse = await fetch(contactUri);
    const contact = (await contactResponse.json()) as Contact;
    ```

## Better with TypedRest

This is where TypedRest comes in. TypedRest is a set of libraries for consuming RESTful APIs that behave in a "predictable" way. Rather than applying your knowledge about how a REST collection usually behaves you simply tell TypedRest that this particular endpoint *is* a collection and get a collection-like interface in return.

=== "C#"

    ```csharp
    var client = new EntryEndpoint(new Uri("http://example.com/"));

    var contacts = new CollectionEndpoint<Contact>(client, relativeUri: "./contacts");
    var contactList = await contacts.ReadAllAsync();

    var smith = await contacts.CreateAsync(new Contact {Name = "Smith"});
    //var smith = contacts["1337"];

    var contact = await smith.ReadAsync();
    ```

=== "Java"

    ```java
    EntryEndpoint client = new EntryEndpoint(URI.create("http://example.com/"));

    CollectionEndpoint<Contact> contacts = new CollectionEndpointImpl<>(client, "./contacts", Contact.class);
    List<Contact> contactList = contacts.readAll();

    ElementEndpoint<Contact> smith = contacts.create(new Contact("Smith"));
    //ElementEndpoint<Contact> smith = contacts.get("1337");

    Contact contact = smith.read();
    ```

=== "Kotlin"

    ```kotlin
    val client = EntryEndpoint(URI.create("http://example.com/"))

    val contacts = CollectionEndpointImpl(client, "./contacts", Contact::class.java)
    val contactList = contacts.readAll()

    val smith = contacts.create(Contact("Smith"))
    //val smith = contacts["1337"]

    val contact = smith.read()
    ```

=== "TypeScript"

    ```typescript
    const client = new EntryEndpoint(new URL("http://example.com/"));

    const contacts = new CollectionEndpoint<Contact>(client, "./contacts");
    const contactList = await contacts.readAll();

    const smith = await contacts.create({name: "Smith"});
    //const smith = contacts.get("1337");

    const contact = await smith.read();
    ```

## Endpoints

TypedRest uses an object-oriented approach to provide you with building blocks for modeling [RESTful endpoints](endpoints/index.md). Behavior of endpoints is described by inheritance while navigation between them is described by composition. For example, we could redesign our sample from above to make the service's functionality easy to discover and consume using code completion:

=== "C#"

    ```csharp
    class MyClient(Uri uri) : EntryEndpoint(uri)
    {
      public CollectionEndpoint<Contact> Contacts => new(this, relativeUri: "./contacts");
    }
    ```

=== "Java"

    ```java
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
    class MyClient(uri: URI) : EntryEndpoint(uri) {
      val contacts: CollectionEndpoint<Contact>
        get() = CollectionEndpointImpl(this, "./contacts", Contact::class.java)
    }
    ```

=== "TypeScript"

    ```typescript
    class MyClient extends EntryEndpoint {
      get contacts() {
        return new CollectionEndpoint<Contact>(this, "./contacts");
      }
    }
    ```

The consuming code could look this:

=== "C#"

    ```csharp
    var client = new MyClient(new Uri("http://example.com/"));
    var contactList = await client.Contacts.ReadAllAsync();
    var smith = await client.Contacts.CreateAsync(new Contact {Name = "Smith"});
    var contact = await smith.ReadAsync();
    ```

=== "Java"

    ```java
    MyClient client = new MyClient(URI.create("http://example.com/"));
    List<Contact> contactList = client.getContacts().readAll();
    ElementEndpoint<Contact> smith = client.getContacts().create(new Contact("Smith"));
    Contact contact = smith.read();
    ```

=== "Kotlin"

    ```kotlin
    val client = MyClient(URI.create("http://example.com/"))
    val contactList = client.contacts.readAll()
    val smith = client.contacts.create(Contact("Smith"))
    val contact = smith.read()
    ```

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));
    const contactList = await client.contacts.readAll();
    const smith = await client.contacts.create({name: "Smith"});
    const contact = await smith.read();
    ```

## Patterns

TypedRest is all about nomenclature and patterns. An endpoint describes any resource addressable via an URI.

- An [entry endpoint](endpoints/entry.md) represents the top-level URI of an API. It takes care of shared concerns such as authentication.
- An [element endpoint](endpoints/generic/element.md) is a singular resource that can be read, modified and deleted.
- A [collection endpoint](endpoints/generic/collection.md) can list and add elements as well as provide element endpoints for individual elements.

There are also a number of more specialized endpoint types such as [streaming-capable collections](endpoints/reactive/streaming-collection.md). Each of these endpoint types has one or more corresponding classes in TypedRest. 

The only requirement on the server-side is that at least a part of the underlying pattern is implemented. For example, a collection endpoint class works as long as the server responds with an array of elements when `GET`ting the collection while `GET`ting a child endpoint provides a specific element of the same type. If adding new elements via `POST` is not supported (yet) this will simply result in an exception at runtime when calling `.CreateAsync()`. For more graceful degradation TypedRest also exposes information about "allowed" methods as reported by `OPTIONS`.

We consider TypedRest's design to be opinionated yet pragmatic. The path of least resistance is to make your API match the patterns implemented in the built-in classes. These usually align with what is widely considered as "best practice". However:

- We explicitly support some unRESTful concepts such as RPC endpoints.
- HATEOAS-style, link-based navigation is possible but entirely optional.
- Link information is preferably encoded in HTTP Link headers instead of response bodies, although the latter is also supported in form of HAL.
- TypedRest does not use custom MIME types for API versioning, navigation, etc..

Of course, we don't expect our predefined patterns to cover all possible use cases. This where good old "extension through inheritance" comes into play. Let's say our sample API from above also allows us to store a note associated with a contact. We need to extend `ElementEndpoint` for individual `Contact` instances to expose this functionality. We also need to replace `CollectionEndpoint` with something that builds instances of our specialized element endpoint rather than using `ElementEndpoint`. Let's get coding!

=== "C#"

    ```csharp
    class MyClient(Uri uri) : EntryEndpoint(uri)
    {
      public ContactCollectionEndpoint Contacts => new(this);
    }

    class ContactCollectionEndpoint(IEndpoint referrer)
      : CollectionEndpoint<Contact, ContactEndpoint>(referrer, relativeUri: "./contacts");

    class ContactEndpoint(IEndpoint referrer, Uri relativeUri)
      : ElementEndpoint<Contact>(referrer, relativeUri)
    {
      public ElementEndpoint<Note> Note => new(this, relativeUri: "./note");
    }
    ```

=== "Java"

    ```java
    class MyClient extends EntryEndpoint {
      public MyClient(URI uri) {
        super(uri);
      }

      public ContactCollectionEndpoint getContacts() {
        return new ContactCollectionEndpoint(this);
      }
    }

    class ContactCollectionEndpoint extends GenericCollectionEndpointImpl<Contact, ContactEndpoint> {
      public ContactCollectionEndpoint(Endpoint referrer) {
        super(referrer, "./contacts", Contact.class, ContactEndpoint::new);
      }
    }

    class ContactEndpoint extends ElementEndpointImpl<Contact> {
      public ContactEndpoint(Endpoint referrer, URI relativeUri) {
        super(referrer, relativeUri, Contact.class);
      }

      public ElementEndpoint<Note> getNote() {
        return new ElementEndpointImpl<>(this, "./note", Note.class);
      }
    }
    ```

=== "Kotlin"

    ```kotlin
    class MyClient(uri: URI) : EntryEndpoint(uri) {
      val contacts: ContactCollectionEndpoint
        get() = ContactCollectionEndpoint(this)
    }

    class ContactCollectionEndpoint(referrer: Endpoint) :
      GenericCollectionEndpointImpl<Contact, ContactEndpoint>(referrer, "./contacts", Contact::class.java, ::ContactEndpoint)

    class ContactEndpoint(referrer: Endpoint, relativeUri: URI) :
      ElementEndpointImpl<Contact>(referrer, relativeUri, Contact::class.java) {
      val note: ElementEndpoint<Note>
        get() = ElementEndpointImpl(this, "./note", Note::class.java)
    }
    ```

=== "TypeScript"

    ```typescript
    class MyClient extends EntryEndpoint {
      get contacts() {
        return new ContactCollectionEndpoint(this);
      }
    }

    class ContactCollectionEndpoint extends GenericCollectionEndpoint<Contact, ContactEndpoint> {
      constructor(referrer: Endpoint) {
        super(referrer, "./contacts", ContactEndpoint);
      }
    }

    class ContactEndpoint extends ElementEndpoint<Contact> {
      get note() {
        return new ElementEndpoint<Note>(this, "./note");
      }
    }
    ```

The consuming code could look this:

=== "C#"

    ```csharp
    var client = new MyClient(new Uri("http://example.com/"));
    await client.Contacts["1337"].Note.SetAsync(new Note {Content = "some note"});
    ```

=== "Java"

    ```java
    MyClient client = new MyClient(URI.create("http://example.com/"));
    client.getContacts().get("1337").getNote().set(new Note("some note"));
    ```

=== "Kotlin"

    ```kotlin
    val client = MyClient(URI.create("http://example.com/"))
    client.contacts["1337"].note.set(Note("some note"))
    ```

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));
    await client.contacts.get("1337").note.set({content: "some note"});
    ```

## Next steps

Continue on to the **[Setup](setup/index.md)** guide.
