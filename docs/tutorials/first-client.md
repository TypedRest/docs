# Build your first client

This tutorial builds a client for the [example address book API](index.md#the-example-api) by hand. It starts with a single collection and grows into a client class that mirrors the API's URL structure, so the code that uses it reads like the API itself:

=== "C#"

    ```csharp
    await client.Contacts["1337"].Note.SetAsync(new Note { Content = "Call back on Monday" });
    ```

=== "Java"

    ```java
    client.getContacts().get("1337").getNote().set(new Note("Call back on Monday"));
    ```

=== "Kotlin"

    ```kotlin
    client.contacts["1337"].note.set(Note("Call back on Monday"))
    ```

=== "TypeScript"

    ```typescript
    await client.contacts.get("1337").note.set({ content: "Call back on Monday" });
    ```

If your API has an OpenAPI document, you can [generate the client](from-openapi.md) instead. Reading this tutorial first still helps you understand what the generator produces.

## 1. Install TypedRest

Follow the [setup guide](../setup/index.md) for your language and add the [entity classes](index.md#the-example-api) to your project.

## 2. Talk to a collection

Every client starts with an [entry endpoint](../endpoints/entry.md) for the API's base URI. Other endpoints are created relative to it. Before writing any classes, try a [collection endpoint](../endpoints/generic/collection.md) directly:

=== "C#"

    ```csharp
    var client = new EntryEndpoint(new Uri("https://example.com/api/"));
    var contacts = new CollectionEndpoint<Contact>(client, relativeUri: "./contacts");

    // GET /api/contacts
    List<Contact> all = await contacts.ReadAllAsync();

    // POST /api/contacts
    var smith = await contacts.CreateAsync(new Contact { Name = "Smith" });

    // GET /api/contacts/{id}, using the URI from the Location header
    Contact contact = await smith!.ReadAsync();
    ```

    Without a `Location` header, TypedRest only finds the ID in the response body if the entity has a property marked with `[Key]` (from `System.ComponentModel.DataAnnotations`). The `Contact` class above has no such attribute, so for it TypedRest relies on the `Location` header alone. Add `[Key]` to `Id` if your server returns the new entity without a `Location` header.

=== "Java"

    ```java
    EntryEndpoint client = new EntryEndpoint(URI.create("https://example.com/api/"), (HttpCredentials) null, new JacksonJsonSerializer());
    CollectionEndpoint<Contact> contacts = new CollectionEndpointImpl<>(client, "./contacts", Contact.class);

    // GET /api/contacts
    List<Contact> all = contacts.readAll();

    // POST /api/contacts
    ElementEndpoint<Contact> smith = contacts.create(new Contact("Smith"));

    // GET /api/contacts/{id}, using the URI from the Location header
    Contact contact = smith.read();
    ```

    Without a `Location` header, TypedRest finds the ID in the response body by calling the entity's `getId()` method.

=== "Kotlin"

    ```kotlin
    val client = EntryEndpoint(URI.create("https://example.com/api/"))
    val contacts = CollectionEndpointImpl(client, "./contacts", Contact::class.java)

    // GET /api/contacts
    val all = contacts.readAll()

    // POST /api/contacts
    val smith = contacts.create(Contact(name = "Smith"))

    // GET /api/contacts/{id}, using the URI from the Location header
    val contact = smith!!.read()
    ```

    Without a `Location` header, TypedRest finds the ID in the response body by calling the entity's `getId()` method, which Kotlin generates for the `id` property.

=== "TypeScript"

    ```typescript
    const client = new EntryEndpoint(new URL("https://example.com/api/"));
    const contacts = new CollectionEndpoint<Contact>(client, "./contacts");

    // GET /api/contacts
    const all = await contacts.readAll();

    // POST /api/contacts
    const smith = await contacts.create({ name: "Smith" });

    // GET /api/contacts/{id}, using the URI from the Location header
    const contact = await smith!.read();
    ```

    Without a `Location` header, TypedRest finds the ID in the response body by reading the entity's `id` property.

Creating an element returns an endpoint for it rather than the entity. TypedRest takes the new element's URI from the `Location` header, or from the ID in the response body if there is no such header. If the server sends neither, it returns `null`/`undefined`, which is why the code above needs `!` or `!!`.

!!! tip "The `./` prefix"
    `./contacts` resolves to `https://example.com/api/contacts`. Without the `./`, URI resolution would replace the last segment of the parent URI if it lacked a trailing slash. Prefixing relative URIs with `./` makes child URIs predictable. See [Relative URIs](../link-handling/relative-uris.md).

## 3. Wrap it in a client class

Creating endpoints inline works, but every caller would need to know the URL structure. Instead, derive from the entry endpoint and expose child endpoints as properties:

=== "C#"

    ```csharp
    public class AddressBookClient(Uri uri) : EntryEndpoint(uri)
    {
        public CollectionEndpoint<Contact> Contacts => new(this, relativeUri: "./contacts");
    }
    ```

=== "Java"

    ```java
    public class AddressBookClient extends EntryEndpoint {
        public AddressBookClient(URI uri) {
            super(uri, (HttpCredentials) null, new JacksonJsonSerializer());
        }

        public CollectionEndpoint<Contact> getContacts() {
            return new CollectionEndpointImpl<>(this, "./contacts", Contact.class);
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    class AddressBookClient(uri: URI) : EntryEndpoint(uri) {
        val contacts: CollectionEndpoint<Contact>
            get() = CollectionEndpointImpl(this, "./contacts", Contact::class.java)
    }
    ```

=== "TypeScript"

    ```typescript
    import { EntryEndpoint } from "typedrest/endpoints";
    import { CollectionEndpoint } from "typedrest/endpoints/generic";

    export class AddressBookClient extends EntryEndpoint {
        get contacts() {
            return new CollectionEndpoint<Contact>(this, "./contacts");
        }
    }
    ```

Each access to the property creates a new endpoint object. That is cheap, because an endpoint is only a URI plus a reference to its parent; the HTTP client, serializer and error handler are shared with the entry endpoint.

## 4. Add sub-resources

A contact has a note and a poke action, but `CollectionEndpoint<Contact>` hands out plain element endpoints that know nothing about them. Derive your own element endpoint and tell the collection to use it:

=== "C#"

    ```csharp
    public class ContactEndpoint(IEndpoint referrer, Uri relativeUri)
        : ElementEndpoint<Contact>(referrer, relativeUri)
    {
        public ElementEndpoint<Note> Note => new(this, relativeUri: "./note");

        public ActionEndpoint Poke => new(this, relativeUri: "./poke");
    }

    public class AddressBookClient(Uri uri) : EntryEndpoint(uri)
    {
        public CollectionEndpoint<Contact, ContactEndpoint> Contacts => new(this, relativeUri: "./contacts");
    }
    ```

    The collection creates element endpoints through a constructor with exactly the signature `(IEndpoint referrer, Uri relativeUri)`, so keep it.

=== "Java"

    ```java
    public class ContactEndpoint extends ElementEndpointImpl<Contact> {
        public ContactEndpoint(Endpoint referrer, URI relativeUri) {
            super(referrer, relativeUri, Contact.class);
        }

        public ElementEndpoint<Note> getNote() {
            return new ElementEndpointImpl<>(this, "./note", Note.class);
        }

        public ActionEndpoint getPoke() {
            return new ActionEndpointImpl(this, "./poke");
        }
    }

    public class AddressBookClient extends EntryEndpoint {
        public AddressBookClient(URI uri) {
            super(uri, (HttpCredentials) null, new JacksonJsonSerializer());
        }

        public GenericCollectionEndpoint<Contact, ContactEndpoint> getContacts() {
            return new GenericCollectionEndpointImpl<>(this, "./contacts", Contact.class, ContactEndpoint::new);
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    class ContactEndpoint(referrer: Endpoint, relativeUri: URI) :
        ElementEndpointImpl<Contact>(referrer, relativeUri, Contact::class.java) {
        val note: ElementEndpoint<Note>
            get() = ElementEndpointImpl(this, "./note", Note::class.java)

        val poke: ActionEndpoint
            get() = ActionEndpointImpl(this, "./poke")
    }

    class AddressBookClient(uri: URI) : EntryEndpoint(uri) {
        val contacts: GenericCollectionEndpoint<Contact, ContactEndpoint>
            get() = GenericCollectionEndpointImpl(this, "./contacts", Contact::class.java, ::ContactEndpoint)
    }
    ```

=== "TypeScript"

    ```typescript
    import { EntryEndpoint } from "typedrest/endpoints";
    import { ElementEndpoint, GenericCollectionEndpoint } from "typedrest/endpoints/generic";
    import { ActionEndpoint } from "typedrest/endpoints/rpc";

    export class ContactEndpoint extends ElementEndpoint<Contact> {
        get note() {
            return new ElementEndpoint<Note>(this, "./note");
        }

        get poke() {
            return new ActionEndpoint(this, "./poke");
        }
    }

    export class AddressBookClient extends EntryEndpoint {
        get contacts() {
            return new GenericCollectionEndpoint<Contact, ContactEndpoint>(this, "./contacts", ContactEndpoint);
        }
    }
    ```

The note is an [element endpoint](../endpoints/generic/element.md) because it can be read and replaced. Poking is not a resource at all, just something the server does, so it is an [action endpoint](../endpoints/rpc/action.md). TypedRest has an endpoint type for each of these common shapes; see [Endpoints](../endpoints/index.md) for the full list.

## 5. Use the client

The client now covers the whole API:

=== "C#"

    ```csharp
    var client = new AddressBookClient(new Uri("https://example.com/api/"));

    // POST /api/contacts -> Location: /api/contacts/1337
    ContactEndpoint smith = (await client.Contacts.CreateAsync(new Contact { Name = "Smith" }))!;

    // PUT /api/contacts/1337/note
    await smith.Note.SetAsync(new Note { Content = "Call back on Monday" });

    // POST /api/contacts/1337/poke
    await smith.Poke.InvokeAsync();

    // Address an existing contact by ID
    Note note = await client.Contacts["1337"].Note.ReadAsync();

    // DELETE /api/contacts/1337
    await smith.DeleteAsync();
    ```

=== "Java"

    ```java
    AddressBookClient client = new AddressBookClient(URI.create("https://example.com/api/"));

    // POST /api/contacts -> Location: /api/contacts/1337
    ContactEndpoint smith = client.getContacts().create(new Contact("Smith"));

    // PUT /api/contacts/1337/note
    smith.getNote().set(new Note("Call back on Monday"));

    // POST /api/contacts/1337/poke
    smith.getPoke().invoke();

    // Address an existing contact by ID
    Note note = client.getContacts().get("1337").getNote().read();

    // DELETE /api/contacts/1337
    smith.delete();
    ```

=== "Kotlin"

    ```kotlin
    val client = AddressBookClient(URI.create("https://example.com/api/"))

    // POST /api/contacts -> Location: /api/contacts/1337
    val smith = client.contacts.create(Contact(name = "Smith"))!!

    // PUT /api/contacts/1337/note
    smith.note.set(Note("Call back on Monday"))

    // POST /api/contacts/1337/poke
    smith.poke.invoke()

    // Address an existing contact by ID
    val note = client.contacts["1337"].note.read()

    // DELETE /api/contacts/1337
    smith.delete()
    ```

=== "TypeScript"

    ```typescript
    const client = new AddressBookClient(new URL("https://example.com/api/"));

    // POST /api/contacts -> Location: /api/contacts/1337
    const smith = (await client.contacts.create({ name: "Smith" }))!;

    // PUT /api/contacts/1337/note
    await smith.note.set({ content: "Call back on Monday" });

    // POST /api/contacts/1337/poke
    await smith.poke.invoke();

    // Address an existing contact by ID
    const note = await client.contacts.get("1337").note.read();

    // DELETE /api/contacts/1337
    await smith.delete();
    ```

`client.Contacts["1337"]` does not send a request. Building endpoints is purely local; a request goes out only when you call a method such as read or set.

## 6. Handle errors

When the server responds with an error status, TypedRest throws an exception (or rejects the promise) of a type that matches the status code. A missing contact, for example:

=== "C#"

    ```csharp
    try
    {
        var contact = await client.Contacts["999"].ReadAsync();
    }
    catch (KeyNotFoundException)
    {
        Console.WriteLine("No such contact");
    }
    ```

=== "Java"

    ```java
    try {
        Contact contact = client.getContacts().get("999").read();
    } catch (NotFoundException ex) {
        System.out.println("No such contact");
    }
    ```

=== "Kotlin"

    ```kotlin
    try {
        val contact = client.contacts["999"].read()
    } catch (ex: NotFoundException) {
        println("No such contact")
    }
    ```

=== "TypeScript"

    ```typescript
    import { NotFoundError } from "typedrest/errors";

    try {
        const contact = await client.contacts.get("999").read();
    } catch (err) {
        if (err instanceof NotFoundError) console.log("No such contact");
        else throw err;
    }
    ```

If the response body is JSON with a `message` or `details` field, its text becomes the exception message. See [Error handling](../error-handling/index.md) for the full mapping of status codes.

## Next steps

- [Authentication](authentication.md): add credentials or tokens to the client.
- [Testing your client](testing.md): check that the endpoints send the requests you expect.
- [Endpoints](../endpoints/index.md): the other endpoint types you can use for the rest of your API.
