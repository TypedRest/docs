# Tutorials

The other sections of this site describe each part of TypedRest on its own. The tutorials here combine those parts to complete a whole task, from an empty project to working code.

## Getting started

Pick one of these depending on whether your API has an OpenAPI document:

[Build your first client](first-client.md)
: Write a client by hand, one endpoint at a time, and learn how TypedRest models an API along the way.

[Generate a client from OpenAPI](from-openapi.md)
: Let the code generator build the client, check what it inferred and fix what it got wrong.

## Guides

[Authentication](authentication.md)
: Send credentials, API keys and OAuth 2.0 tokens, and handle authentication errors.

[Dependency injection](dependency-injection.md)
: Register a client in ASP.NET Core and configure its HTTP client.

[Command-line tool](command-line.md)
: Turn a .NET client into a command-line interface.

[Optimistic concurrency](concurrency.md)
: Use ETags to keep two clients from overwriting each other's changes.

[Paging](paging.md)
: Read large collections in pages using HTTP range requests.

[Live updates](live-updates.md)
: Get notified of changes by polling, streaming or Server-Sent Events, and choose between them.

[Testing your client](testing.md)
: Test endpoints, the code that uses them, and client and service together.

## The example API

All tutorials use the address book API from the [Introduction](../introduction.md):

| Path                    | Methods                         | Description                                                                 |
| ----------------------- | ------------------------------- | --------------------------------------------------------------------------- |
| `/contacts`             | `GET`, `POST`                   | All contacts. `POST` adds one and returns its URI in the `Location` header. |
| `/contacts/{id}`        | `GET`, `PUT`, `PATCH`, `DELETE` | A single contact.                                                           |
| `/contacts/{id}/note`   | `GET`, `PUT`                    | A note attached to a contact.                                               |
| `/contacts/{id}/poke`   | `POST`                          | Pokes a contact. Takes no input and returns no output.                      |

The tutorials share these entity classes:

=== "C#"

    ```csharp
    public class Contact
    {
        public string? Id { get; set; }
        public string Name { get; set; } = "";
        public string? Email { get; set; }
    }

    public class Note
    {
        public string Content { get; set; } = "";
    }
    ```

=== "Java"

    ```java
    public class Contact {
        private String id;
        private String name;
        private String email;

        public Contact() {}

        public Contact(String name) {
            this.name = name;
        }

        // Getters and setters omitted
    }

    public class Note {
        private String content;

        public Note() {}

        public Note(String content) {
            this.content = content;
        }

        // Getter and setter omitted
    }
    ```

    These are plain Java classes, so they need a reflection-based [serializer](../serializers/json.md) such as Jackson. The tutorials pass a `JacksonJsonSerializer` wherever an entry endpoint is created.

=== "Kotlin"

    ```kotlin
    @Serializable
    data class Contact(val id: String? = null, val name: String, val email: String? = null)

    @Serializable
    data class Note(val content: String)
    ```

=== "TypeScript"

    ```typescript
    interface Contact {
        id?: string;
        name: string;
        email?: string;
    }

    interface Note {
        content: string;
    }
    ```

The [Sample-DotNet](https://github.com/TypedRest/Sample-DotNet) and [Sample-Kotlin](https://github.com/TypedRest/Sample-Kotlin) projects implement the same API as a working service and client, if you want to run the examples against a real server.
