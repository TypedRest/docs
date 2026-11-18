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
    await contact.MergeAsync(new Contact { Email = "newemail@example.com" });

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
    Contact update = new Contact();
    update.setEmail("newemail@example.com");
    contact.merge(update);

    // Delete the element
    contact.delete();
    ```

=== "Kotlin"

    ```kotlin
    val contact = ElementEndpointImpl<Contact>(client, "contacts/123")

    // Check if the element exists
    val exists = contact.exists()

    // Read the entity
    val entity = contact.read()

    // Update the entire entity
    contact.set(Contact("Jane Doe", "jane@example.com"))

    // Partially update the entity
    contact.merge(Contact(email = "newemail@example.com"))

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

## Generic entity types

The entity type can itself be generic, e.g., a `Page<Contact>` wrapper or a plain `List<Contact>`. This applies to all endpoint types that take an entity type, not just element endpoints.

=== "C#"

    ```csharp
    var page = new ElementEndpoint<Page<Contact>>(client, "contacts/page/1");
    ```

=== "Java"

    Java erases generic type arguments, so a `Class` cannot describe `Page<Contact>`. Pass a `java.lang.reflect.Type` instead, e.g., from a type token such as Jackson's `TypeReference` or Guava's `TypeToken`:

    ```java
    Type pageType = new TypeReference<Page<Contact>>() {}.getType();
    ElementEndpoint<Page<Contact>> page = new ElementEndpointImpl<>(client, "contacts/page/1", pageType);
    ```

=== "Kotlin"

    Specify the type argument and TypedRest captures the full generic type:

    ```kotlin
    val page = ElementEndpointImpl<Page<Contact>>(client, "contacts/page/1")
    ```

    When deriving from an endpoint class, pass `javaTypeOf<T>()` to the super constructor instead:

    ```kotlin
    import net.typedrest.javaTypeOf

    class PageEndpoint(referrer: Endpoint, relativeUri: URI) :
        ElementEndpointImpl<Page<Contact>>(referrer, relativeUri, javaTypeOf<Page<Contact>>())
    ```

=== "TypeScript"

    ```typescript
    const page = new ElementEndpoint<Page<Contact>>(client, "contacts/page/1");
    ```

The [serializer](../../serializers/index.md) must be able to handle the generic type, e.g., by annotating `Page` with `@Serializable` when using kotlinx.serialization.
