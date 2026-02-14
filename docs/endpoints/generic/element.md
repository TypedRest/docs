# Element endpoint

Endpoint for an individual resource.

| Method | Input  | Result  | HTTP Verb | Description                                      |
| ------ | ------ | ------- | --------- | ------------------------------------------------ |
| Exists | -      | Boolean | `HEAD`    | Determines whether the element currently exists. |
| Read   | -      | Entity  | `GET`     | Returns the entity.                              |
| Set    | Entity | Entity  | `PUT`     | Sets/replaces the entity.                        |
| Merge  | Entity | Entity  | `PATCH`   | Modifies the existing entity by merging changes. |
| Delete | -      | -       | `DELETE`  | Deletes the element.                             |

## Usage

=== "C#"

    ```csharp
    var contact = new ElementEndpoint<Contact>(client, "contacts/123");

    // Check if the element exists
    bool exists = await contact.ExistsAsync();

    // Read the entity
    Contact entity = await contact.ReadAsync();

    // Update the entire entity
    await contact.SetAsync(new Contact { Name = "Jane Doe", Email = "jane@example.com" });

    // Partially update the entity
    await contact.MergeAsync(new { Email = "newemail@example.com" });

    // Delete the element
    await contact.DeleteAsync();
    ```

=== "Java"

    ```java
    ElementEndpoint<Contact> contact = new ElementEndpointImpl<>(client, "contacts/123", Contact.class);

    // Check if the element exists
    boolean exists = contact.exists();

    // Read the entity
    Contact entity = contact.read();

    // Update the entire entity
    contact.set(new Contact("Jane Doe", "jane@example.com"));

    // Partially update the entity
    Map<String, Object> updates = Map.of("email", "newemail@example.com");
    contact.merge(updates);

    // Delete the element
    contact.delete();
    ```

=== "Kotlin"

    ```kotlin
    val contact = ElementEndpointImpl(client, "contacts/123", Contact::class.java)

    // Check if the element exists
    val exists = contact.exists()

    // Read the entity
    val entity = contact.read()

    // Update the entire entity
    contact.set(Contact("Jane Doe", "jane@example.com"))

    // Partially update the entity
    contact.merge(mapOf("email" to "newemail@example.com"))

    // Delete the element
    contact.delete()
    ```

=== "TypeScript"

    ```typescript
    const contact = new ElementEndpoint<Contact>(client, "contacts/123");

    // Check if the element exists
    const exists = await contact.exists();

    // Read the entity
    const entity = await contact.read();

    // Update the entire entity
    await contact.set({ name: "Jane Doe", email: "jane@example.com" });

    // Partially update the entity
    await contact.merge({ email: "newemail@example.com" });

    // Delete the element
    await contact.delete();
    ```
