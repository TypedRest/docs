title: Home

# ![Logo](images/logo.svg)

TypedRest helps you build type-safe, fluent-style REST API clients. Common REST patterns such as collections are represented as classes, allowing you to write more idiomatic code.

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

=== "TypeScript"

    ```typescript
    const client = new MyClient(new URL("http://example.com/"));

    // GET /contacts
    const contactList: Contact[] = await client.contacts.readAll();

    // POST /contacts -> Location: /contacts/1337
    const smith: ContactEndpoint = await client.contacts.create(new Contact("Smith"));
    //const smith: ContactEndpoint = client.contacts.get("1337");

    // GET /contacts/1337
    contact: Contact = await smith.read();

    // PUT /contacts/1337/note
    await smith.note.set(new Note("some note"));

    // GET /contacts/1337/note
    const note: Note = await smith.note.read();

    // DELETE /contacts/1337
    await smith.delete();
    ```

## Documentation

[Introduction](introduction.md)
: What is TypedRest and how can it help me?

[Getting Started](getting-started/index.md)
: How do I use TypedRest in my projects?

[Endpoints](endpoints/index.md)
: Documentation for all endpoint types provided by TypedRest.

[Error handling](error-handling/index.md)
: How to handle API errors with TypedRest.

[Link handling](link-handling/index.md)
: How to handle relative URIs, link headers, HATEOS, etc. with TypedRest.

[Code generation](code-generation/index.md)
: Auto-generate code for TypedRest from Swagger/OpenAPI sepc.
