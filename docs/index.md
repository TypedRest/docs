title: Home

# ![Logo](images/logo.svg)

TypedRest helps you build type-safe, fluent-style REST API clients. Common REST patterns such as collections are represented as classes, allowing you to write more idiomatic code.

Read the **[Introduction](introduction.md)** to TypedRest or jump right in with the **[Getting started](getting-started/index.md)** guide.

=== "C#"

    ```csharp
    var client = new MyClient(new Uri("http://example.com/"));

    // GET /contacts
    var contactList = await client.Contacts.ReadAllAsync();

    // POST /contacts -> Location: /contacts/1337
    var smith = await client.Contacts.CreateAsync(new Contact {Name = "Smith"});
    //ContactEndpoint smith = client.Contacts["1337"];

    // GET /contacts/1337
    var contact = await smith.ReadAsync();

    // PUT /contacts/1337/note
    await smith.Note.SetAsync(new Note {Content = "some note"});

    // GET /contacts/1337/note
    var note = await smith.Note.ReadAsync();

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
