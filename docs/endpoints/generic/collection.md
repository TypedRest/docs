# Collection endpoint

Endpoint for a collection of entities addressable as [Element endpoints](element.md).

| Method     | Input       | Result   | HTTP Verb | Description                                                         |
| ---------- | ----------- | -------- | --------- | ------------------------------------------------------------------- |
| Get        | ID / Entity | Endpoint | -         | Get an [Element endpoint](element.md) for a specific child element. |
| Read all   | -           | Entities | `GET`     | Returns all entities in the collection.                             |
| Read range | Range       | Entities | `GET`     | Returns all entities within a specific range of the collection.     |
| Create     | Entity      | -        | `POST`    | Adds an entity as a new element to the collection.                  |
| Create All | Entities    | -        | `PATCH`   | Adds (or updates) multiple entities as elements in the collection.  |
| Set All    | Entities    | -        | `PUT`     | Replaces the entire content of the collection with new entities.    |

Extends [Indexer endpoint](indexer.md)

## Usage

=== "C#"

    ```csharp
    var contacts = new CollectionEndpoint<Contact>(client, "contacts");

    // Read all entities in the collection
    List<Contact> allContacts = await contacts.ReadAllAsync();

    // Read a range of entities
    PartialResponse<Contact> someContacts = await contacts.ReadRangeAsync(new RangeItemHeaderValue(0, 49));

    // Create a new entity
    ElementEndpoint<Contact> newContact = await contacts.CreateAsync(new Contact { Name = "John Doe" });

    // Get an element by ID
    ElementEndpoint<Contact> contact = contacts["123"];

    // Update multiple entities
    await contacts.CreateAllAsync([
        new Contact { Name = "Alice" },
        new Contact { Name = "Bob" }
    ]);

    // Replace entire collection
    await contacts.SetAllAsync([
        new Contact { Name = "Charlie" },
        new Contact { Name = "Diana" }
    ]);
    ```

    You can also configure `CollectionEndpoint` to use a custom type derived from `ElementEndpoint` for [elements](element.md).

    ```csharp
    var contacts = new CollectionEndpoint<Contact, ContactEndpoint>(client, "contacts");

    // Create a new entity
    ContactEndpoint newContact = await contacts.CreateAsync(new Contact { Name = "John Doe" });

    // Get an element by ID
    ContactEndpoint contact = contacts["123"];
    ```

=== "Java"

    ```java
    CollectionEndpoint<Contact> contacts = new CollectionEndpointImpl<>(client, "contacts", Contact.class);

    // Read all entities in the collection
    List<Contact> allContacts = contacts.readAll();

    // Read a range of entities
    PartialResponse<Contact> someContacts = contacts.readRange(Range.ofLength(0, 50));

    // Create a new entity
    ElementEndpoint<Contact> newContact = contacts.create(new Contact("John Doe"));

    // Get an element by ID
    ElementEndpoint<Contact> contact = contacts.get("123");

    // Update multiple entities
    contacts.createAll(List.of(
        new Contact("Alice"),
        new Contact("Bob")
    ));

    // Replace entire collection
    contacts.setAll(List.of(
        new Contact("Charlie"),
        new Contact("Diana")
    ));
    ```

    You can also use `GenericCollectionEndpointImpl` to use a custom type derived from `ElementEndpoint` for [elements](element.md).

    ```java
    GenericCollectionEndpoint<Contact, ContactEndpoint> contacts = new GenericCollectionEndpointImpl<>(client, "contacts", Contact.class, (ref, uri) -> new ContactEndpoint(ref, uri));

    // Create a new entity
    ContactEndpoint newContact = contacts.create(new Contact("John Doe"));

    // Get an element by ID
    ContactEndpoint contact = contacts.get("123");
    ```

=== "Kotlin"

    ```kotlin
    val contacts = CollectionEndpointImpl(client, "contacts", Contact::class.java)

    // Read all entities in the collection
    val allContacts = contacts.readAll()

    // Read a range of entities
    val someContacts = contacts.readRange(Range.ofLength(0, 50))

    // Create a new entity
    val newContact = contacts.create(Contact("John Doe"))

    // Get an element by ID
    val contact = contacts["123"]

    // Update multiple entities
    contacts.createAll(listOf(
        Contact("Alice"),
        Contact("Bob")
    ))

    // Replace entire collection
    contacts.setAll(listOf(
        Contact("Charlie"),
        Contact("Diana")
    ))
    ```

    You can also use `GenericCollectionEndpointImpl` to use a custom type derived from `ElementEndpoint` for [elements](element.md).

    ```kotlin
    val contacts = GenericCollectionEndpointImpl(client, "contacts", Contact::class.java, ::ContactEndpoint)

    // Create a new entity
    val newContact: ContactEndpoint = contacts.create(Contact("John Doe"))

    // Get an element by ID
    val contact: ContactEndpoint = contacts["123"]
    ```

=== "TypeScript"

    ```typescript
    const contacts = new CollectionEndpoint<Contact>(client, "contacts");

    // Read all entities in the collection
    const allContacts = await contacts.readAll();

    // Create a new entity
    const newContact = await contacts.create({ name: "John Doe" });

    // Get an element by ID
    const contact = contacts.get("123");

    // Update multiple entities
    await contacts.createAll([
        { name: "Alice" },
        { name: "Bob" }
    ]);

    // Replace entire collection
    await contacts.setAll([
        { name: "Charlie" },
        { name: "Diana" }
    ]);
    ```

    You can also use `GenericCollectionEndpoint` to use a custom type derived from `ElementEndpoint` for [elements](element.md).

    ```typescript
    const contacts = new GenericCollectionEndpoint<Contact, ContactEndpoint>(client, "contacts", ContactEndpoint);

    // Create a new entity
    const newContact: ContactEndpoint = await contacts.create({ name: "John Doe" });

    // Get an element by ID
    const contact: ContactEndpoint = contacts.get("123");
    ```
