# Code generation

TypedRest can build clients for you: the code generation tool reads an [OpenAPI/Swagger](https://swagger.io/resources/open-api/) document, infers [TypedRest Endpoints](../endpoints/index.md) from patterns in the described paths and generates source code for a client.

It can generate C#, Java, Kotlin and TypeScript clients.

## Two ways to use it

[Source generator](source-generator.md)
: Generates the C# client during compilation, straight from a spec file referenced by your project. Nothing is written to your source tree. Best for .NET projects that build the client alongside the code that consumes it.

[Command-line tool](cli.md)
: Writes the generated code to disk as `.cs`, `.java`, `.kotlin` or `.ts` files that you check in or post-process.

Both use the same inference, so they infer the same endpoints from the same spec.

## How inference works

The paths in the document are arranged into a tree, with URI template placeholders such as `/contacts/{id}` collapsed into a single wildcard node. Each node is then matched against a list of patterns. The first pattern that matches wins:

| Pattern    | Matches a path with                                                                 | Generated endpoint                                        |
| ---------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Collection | `GET` returning an array of `T`, and a child wildcard path with `GET` returning `T` | [Collection endpoint](../endpoints/generic/collection.md) |
| Indexer    | a child wildcard path, but no matching array response                               | [Indexer endpoint](../endpoints/generic/indexer.md)       |
| Element    | `GET` with a JSON response body                                                     | [Element endpoint](../endpoints/generic/element.md)       |
| Function   | `POST` with both a JSON request and a JSON response body                            | [Function endpoint](../endpoints/rpc/function.md)         |
| Consumer   | `POST` with a JSON request body                                                     | [Consumer endpoint](../endpoints/rpc/consumer.md)         |
| Producer   | `POST` with a JSON response body                                                    | [Producer endpoint](../endpoints/rpc/producer.md)         |
| Action     | `POST` with neither                                                                 | [Action endpoint](../endpoints/rpc/action.md)             |
| Blob       | `GET` with a non-JSON response body                                                 | [Blob endpoint](../endpoints/raw/blob.md)                 |
| Upload     | `POST` with a non-JSON request body                                                 | [Upload endpoint](../endpoints/raw/upload.md)             |
| Default    | nothing of the above, but child paths that matched                                  | plain endpoint holding the children                       |

The whole tree becomes an [entry endpoint](../endpoints/entry.md) with the matched endpoints as (nested) properties.

The [reactive endpoints](../endpoints/index.md) (polling, streaming, streaming collection) have no pattern, because an OpenAPI document does not describe the difference between a regular and a streaming response. You can still request them explicitly; see [Custom code](custom-code.md).

## Example

Given this excerpt of an OpenAPI document:

```yaml
paths:
  /contacts:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Contact'
  '/contacts/{id}':
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Contact'
  '/contacts/{id}/note':
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Note'
  '/contacts/{id}/poke':
    post:
      responses:
        '204':
          description: ''
  '/contacts/{id}/picture':
    get:
      responses:
        '200':
          content:
            image/jpeg:
              schema: { }
```

`/contacts` becomes a collection of `Contact` elements, `note` an element endpoint, `poke` an action endpoint and `picture` a blob endpoint. Generated with the service name `Sample`, the client is used like this:

=== "C#"

    ```csharp
    var client = new SampleClient(new Uri("https://example.com/api/"));

    // GET /contacts
    List<Contact> contacts = await client.Contacts.ReadAllAsync();

    // GET /contacts/1337/note
    Note note = await client.Contacts["1337"].Note.ReadAsync();

    // POST /contacts/1337/poke
    await client.Contacts["1337"].Poke.InvokeAsync();
    ```

=== "Java"

    ```java
    SampleClient client = new SampleClient(URI.create("https://example.com/api/"));

    // GET /contacts
    List<Contact> contacts = client.getContacts().readAll();

    // GET /contacts/1337/note
    Note note = client.getContacts().get("1337").getNote().read();

    // POST /contacts/1337/poke
    client.getContacts().get("1337").getPoke().invoke();
    ```

=== "Kotlin"

    ```kotlin
    val client = SampleClient(URI.create("https://example.com/api/"))

    // GET /contacts
    val contacts: List<Contact> = client.contacts.readAll()

    // GET /contacts/1337/note
    val note: Note = client.contacts["1337"].note.read()

    // POST /contacts/1337/poke
    client.contacts["1337"].poke.invoke()
    ```

=== "TypeScript"

    ```typescript
    const client = new SampleClient("https://example.com/api/");

    // GET /contacts
    const contacts: Contact[] = await client.contacts.readAll();

    // GET /contacts/1337/note
    const note: Note = await client.contacts.get("1337").note.read();

    // POST /contacts/1337/poke
    await client.contacts.get("1337").poke.invoke();
    ```

## Naming

Type and property names are derived from the path segments:

- The entry endpoint is named after the service, e.g. `SampleClient`.
- Collection and indexer endpoints are depluralized, e.g. `/contacts` becomes a `ContactCollectionEndpoint` exposed as a `Contacts` property.
- The element of a collection gets the `Element` suffix, e.g. `ContactElementEndpoint`.
- `kebab-case`, `snake_case` and `camelCase` segments are all converted to `PascalCase` for C#, and to `camelCase` for TypeScript, Kotlin and Java.
- DTO types are named after the schemas in `components`, in a namespace (C#), package (Kotlin/Java) or directory (TypeScript) you can keep separate from the endpoints.

If the same name would be generated in more than one place, the parent name is used as a prefix to disambiguate.

DTO *members* are an exception in TypeScript: they keep the exact name used on the wire, because TypedRest for TypeScript deserializes with `JSON.parse()` and so has no way to map a property to a differently named field.

## Serializers

Generated DTOs have to be annotated for whichever [serializer](../serializers/json.md) the endpoint uses at runtime, so the tool takes a `--serializer` option:

| Language   | Values                                     | Default            |
| ---------- | ------------------------------------------ | ------------------ |
| C#         | `newtonsoft`, `system-text-json`           | `newtonsoft`       |
| Kotlin     | `kotlinx`, `jackson`, `moshi`              | `kotlinx`          |
| Java       | `jackson`, `moshi`                         | `jackson`          |
| TypeScript | *(none)*                                   |                    |

Each default matches what the corresponding TypedRest port uses out of the box, so most projects never set this.

**The choice has to match your runtime configuration.** The serializers read entirely different attributes, so a DTO annotated for one silently falls back to its member names under the other, changing the wire format without any error.

**Java cannot use `kotlinx`.** kotlinx.serialization generates its serializers with a Kotlin compiler plugin and cannot handle a class written in Java, so the tool rejects the combination.

## Overriding the inference

For APIs where the patterns do not produce the client you want, you can write the inferred structure into the document as an `x-typedrest` extension and edit it by hand, or plug in your own patterns and code builders. See [Custom code](custom-code.md).

## See also

- [API documentation](https://code-generation.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/CodeGeneration)
