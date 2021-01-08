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

=== "TypeScript"

    ```typescript
    const contactsResponse = await fetch("http://example.com/contacts");
    const contactList = (await contactsResponse.json()) as Contact[];

    const createResponse = await fetch("http://example.com/contacts", {
      body: JSON.stringify(new Contact("Smith"))
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

=== "TypeScript"

    ```typescript
    const client = new EntryEndpoint(new URL("http://example.com/"));

    const contacts = new CollectionEndpoint<Contact>(client, "./contacts");
    const contactList = await contacts.readAll();

    const smith = await contacts.create(new Contact("Smith"));
    //const smith = contacts["1337"];

    const contact = await smith.read();
    ```

## Endpoints

TypedRest uses an object-oriented approach to provide you with building blocks for modeling [RESTful endpoints](endpoints/index.md). Behavior of endpoints is described by inheritance while navigation between them is described by composition. For example, we could redesign our sample from above to make the service's functionality easy to discover and consume using code completion:

=== "C#"

    ```csharp
    class MyClient : EntryEndpoint
    {
      public MyClient(Uri uri)
        : base(uri)
      {}

      public ICollectionEndpoint<Contact> Contacts
        => new CollectionEndpoint<Contact>(this, relativeUri: "./contacts");
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

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));
    const contactList = await client.contacts.readAll();
    const smith = await client.contacts.create(new Contact("Smith"));
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
    class MyClient : EntryEndpoint
    {
      public MyClient(Uri uri) : base(uri)
      {}
    
      public ContactCollectionEndpoint Contacts
        => new ContactCollectionEndpoint(this);
    }

    class ContactCollectionEndpoint : CollectionEndpoint<Contact, ContactEndpoint>
    {
      public ContactCollectionEndpoint(IEndpoint referrer)
        : base(referrer, relativeUri: "./contacts")
      {}
    }

    class ContactEndpoint : ElementEndpoint<Contact>
    {
      public ContactEndpoint(IEndpoint referrer, Uri relativeUri)
        : base(referrer, relativeUri)
      {}

      public IElementEndpoint<Note> Note
        => new ElementEndpoint<Note>(this, relativeUri: "./note");
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

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));
    await client.contacts.get("1337").note.set(new Note("some note"));
    ```

## Next steps

Continue on to the **[Getting started](getting-started/index.md)** guide.
