title: Home

# ![Logo](images/logo.svg)

**Describe your REST API as a tree of endpoints, not a list of HTTP calls.**

TypedRest helps you build type-safe, fluent-style REST API clients. Common REST patterns are represented as classes that you compose to mirror the URL structure of your API.

Tell TypedRest that `/contacts` is a collection and get type-safe listing, creating, reading, updating and deleting in return.

[Build your first client](tutorials/first-client.md){ .md-button .md-button--primary }
[Generate one from OpenAPI](tutorials/from-openapi.md){ .md-button }

## Define your client

A few lines are enough to describe an API with a collection of contacts plus a "notes" endpoint nested underneath:

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

## Use it

Navigate the API like an object model, with code completion guiding the way:

=== "C#"

    ```csharp
    var client = new MyClient(new Uri("http://example.com/"));

    // GET /contacts
    List<Contact> contactList = await client.Contacts.ReadAllAsync();

    // POST /contacts -> Location: /contacts/1337
    ContactEndpoint smith = await client.Contacts.CreateAsync(new Contact {Name = "Smith"});
    //ContactEndpoint smith = client.Contacts["1337"];

    // GET /contacts/1337
    Contact contact = await smith.ReadAsync();

    // PUT /contacts/1337/note
    await smith.Note.SetAsync(new Note {Content = "some note"});

    // GET /contacts/1337/note
    Note note = await smith.Note.ReadAsync();

    // DELETE /contacts/1337
    await smith.DeleteAsync();
    ```

=== "Java"

    ```java
    MyClient client = new MyClient(URI.create("http://example.com/"));

    // GET /contacts
    List<Contact> contactList = client.getContacts().readAll();

    // POST /contacts -> Location: /contacts/1337
    ContactEndpoint smith = client.getContacts().create(new Contact("Smith"));
    //ContactEndpoint smith = client.getContacts().get("1337");

    // GET /contacts/1337
    Contact contact = smith.read();

    // PUT /contacts/1337/note
    smith.getNote().set(new Note("some note"));

    // GET /contacts/1337/note
    Note note = smith.getNote().read();

    // DELETE /contacts/1337
    smith.delete();
    ```

=== "Kotlin"

    ```kotlin
    val client = MyClient(URI.create("http://example.com/"))

    // GET /contacts
    val contactList: List<Contact> = client.contacts.readAll()

    // POST /contacts -> Location: /contacts/1337
    val smith: ContactEndpoint = client.contacts.create(Contact("Smith"))
    //val smith: ContactEndpoint = client.contacts["1337"]

    // GET /contacts/1337
    val contact: Contact = smith.read()

    // PUT /contacts/1337/note
    smith.note.set(Note("some note"))

    // GET /contacts/1337/note
    val note: Note = smith.note.read()

    // DELETE /contacts/1337
    smith.delete()
    ```

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));

    // GET /contacts
    const contactList: Contact[] = await client.contacts.readAll();

    // POST /contacts -> Location: /contacts/1337
    const smith: ContactEndpoint = await client.contacts.create({name: "Smith"});
    //const smith: ContactEndpoint = client.contacts.get("1337");

    // GET /contacts/1337
    const contact: Contact = await smith.read();

    // PUT /contacts/1337/note
    await smith.note.set({content: "some note"});

    // GET /contacts/1337/note
    const note: Note = await smith.note.read();

    // DELETE /contacts/1337
    await smith.delete();
    ```

## Why TypedRest?

**Plain HTTP clients** leave the REST conventions in your head: you build the URLs, pick the verbs, follow the `Location` headers and (de)serialize the bodies yourself.

**Typical generated clients** give you one flat method per operation, such as `getContactNote(id)`, without capturing how the resources relate to each other.

**TypedRest** gives you a navigable tree of endpoints that mirrors the URL structure of your API. You declare which pattern each resource follows and inherit its behavior.

## Documentation

[Introduction](introduction.md)
: What is TypedRest and how can it help me?

[Setup](setup/index.md)
: How do I use TypedRest in my projects?

[Tutorials](tutorials/index.md)
: Step-by-step guides for building, generating, securing and testing clients.

[Endpoints](endpoints/index.md)
: Documentation for all endpoint types provided by TypedRest.

[Serializers](serializers/index.md)
: How to convert objects to and from the wire format (e.g., JSON).

[Error handling](error-handling/index.md)
: How to handle API errors with TypedRest.

[Testing](testing/index.md)
: How to test TypedRest endpoint wrappers and mock endpoints in unit tests.

[Link handling](link-handling/index.md)
: How to handle relative URIs, link headers, HATEOAS, etc. with TypedRest.

[Code generation](code-generation/index.md)
: Auto-generate code for TypedRest from Swagger/OpenAPI spec.
