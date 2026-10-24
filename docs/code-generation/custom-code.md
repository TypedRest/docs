# Custom code

For APIs where the [pattern inference](index.md#how-inference-works) does not produce the client you want, there are three levels of customization, in increasing order of effort.

## 1. Annotate the document

When a document contains an `x-typedrest` extension, both the [source generator](source-generator.md) and the [command-line tool](cli.md) use it as-is instead of running the inference. Let the tool write a starting point for you:

```bash
typedrest-codegen pattern -f myapi.yml
```

and then edit the result. The extension is a tree of endpoints, keyed by the property names to generate:

```yaml
x-typedrest:
  contacts:
    kind: collection
    uri: ./contacts
    description: Collection of contacts.
    schema:
      $ref: '#/components/schemas/Contact'
    element:
      kind: element
      description: A specific contact.
      children:
        note:
          kind: element
          uri: ./note
          schema:
            $ref: '#/components/schemas/Note'
        poke:
          kind: action
          uri: ./poke
        picture:
          kind: blob
          uri: ./picture
```

Every endpoint takes `kind`, `uri`, `description` and `children`. Some kinds take additional properties:

| `kind`                 | Additional properties               | Generated endpoint                                                             |
| ---------------------- | ----------------------------------- | ------------------------------------------------------------------------------ |
| *(omitted)*            |                                     | plain endpoint that only holds children                                        |
| `element`              | `schema`                            | [Element endpoint](../endpoints/generic/element.md)                            |
| `collection`           | `schema`, `element`                 | [Collection endpoint](../endpoints/generic/collection.md)                      |
| `indexer`              | `element`                           | [Indexer endpoint](../endpoints/generic/indexer.md)                            |
| `action`               |                                     | [Action endpoint](../endpoints/rpc/action.md)                                  |
| `consumer`             | `schema`                            | [Consumer endpoint](../endpoints/rpc/consumer.md)                              |
| `producer`             | `schema`                            | [Producer endpoint](../endpoints/rpc/producer.md)                              |
| `function`             | `request-schema`, `response-schema` | [Function endpoint](../endpoints/rpc/function.md)                              |
| `blob`                 |                                     | [Blob endpoint](../endpoints/raw/blob.md)                                      |
| `upload`               | `form-field`                        | [Upload endpoint](../endpoints/raw/upload.md)                                  |
| `polling`              | `schema`                            | [Polling endpoint](../endpoints/reactive/polling.md)                           |
| `streaming`            | `schema`                            | [Streaming endpoint](../endpoints/reactive/streaming.md)                       |
| `sse`                  | `schema`, `event-type`              | [SSE endpoint](../endpoints/reactive/sse.md)                                   |
| `streaming-collection` | `schema`, `element`                 | [Streaming collection endpoint](../endpoints/reactive/streaming-collection.md) |

The reactive kinds are never inferred, since an OpenAPI document does not describe the difference between a regular and a streaming response. Annotating the document is the only way to get them.

Because the generator uses the extension as-is, **paths you add to the document later are not picked up automatically**. Add them to `x-typedrest` by hand as well, or remove the extension, rerun `pattern` and reapply your edits.

## 2. Extend the generated code

Generated code is overwritten whenever it is regenerated, so do not edit it. Add to it instead. Adding the methods you need for special cases usually beats fighting the generator.

=== "C#"

    All generated classes and interfaces are `partial`. Add a file of your own in the same namespace:

    ```csharp
    namespace MyCompany.MyService;

    public partial class ContactElementEndpoint
    {
        public Task<Contact?> RenameAsync(string name, CancellationToken cancellationToken = default)
            => UpdateAsync(contact => contact.Name = name, cancellationToken: cancellationToken);
    }
    ```

    If you generate interfaces and want the method on `IContactElementEndpoint` too, add a `partial interface` declaration the same way.

    A partial class cannot change a constructor that has already been generated. If the entry endpoint needs a custom error handler or default headers, turn off its generated constructor (`GenerateEntryConstructor="false"` for the [source generator](source-generator.md), `--generate-entry-constructor false` for the [command-line tool](cli.md)) and write your own in a partial class.

=== "Java"

    Derive from the generated class (the `Impl` class if you generate interfaces):

    ```java
    public class MyClient extends MyServiceClient {
        public MyClient(URI uri) {
            super(uri);
        }

        public List<Contact> findByName(String name) {
            return getContacts().readAll().stream()
                .filter(x -> name.equals(x.name))
                .collect(Collectors.toList());
        }
    }
    ```

=== "Kotlin"

    Generated endpoints are `open class`es, so you can derive from them (the `Impl` class if you generate interfaces):

    ```kotlin
    class MyClient(uri: URI) : MyServiceClient(uri) {
        fun findByName(name: String): List<Contact> =
            contacts.readAll().filter { it.name == name }
    }
    ```

    For small additions, an extension function works without subclassing, and on child endpoints too:

    ```kotlin
    fun MyServiceClient.findByName(name: String) = contacts.readAll().filter { it.name == name }
    ```

=== "TypeScript"

    Derive from the generated class:

    ```typescript
    export class MyClient extends MyServiceClient {
        async findByName(name: string) {
            return (await this.contacts.readAll()).filter(x => x.name === name);
        }
    }
    ```

When you derive from a generated class, use your subclass in place of the generated one. The generated code still creates child endpoints as the generated types, so subclassing is mostly useful for the entry endpoint. Put helpers for child endpoints on your entry endpoint subclass, or use partial classes (C#) or extension functions (Kotlin).

## 3. Build your own tool

If you need to change how the inference or the code generation itself works, write your own command-line tool on top of these NuGet packages:

[TypedRest.CodeGeneration](https://www.nuget.org/packages/TypedRest.CodeGeneration/)
: Parses OpenAPI/Swagger documents and infers TypedRest Endpoints from patterns. Implement `IPattern` and add it to a `PatternRegistry` to recognize your own path shapes.

[TypedRest.CodeGeneration.CSharp](https://www.nuget.org/packages/TypedRest.CodeGeneration.CSharp/)
: Generates C# source code for [TypedRest .NET](https://github.com/TypedRest/TypedRest-DotNet) clients.

[TypedRest.CodeGeneration.TypeScript](https://www.nuget.org/packages/TypedRest.CodeGeneration.TypeScript/)
: Generates TypeScript source code for [TypedRest for TypeScript](https://github.com/TypedRest/TypedRest-TypeScript) clients.

[TypedRest.CodeGeneration.Jvm](https://www.nuget.org/packages/TypedRest.CodeGeneration.Jvm/)
: Everything the Kotlin and Java generators have in common: the JVM type model, the naming strategy and the endpoint builders.

[TypedRest.CodeGeneration.Kotlin](https://www.nuget.org/packages/TypedRest.CodeGeneration.Kotlin/) and [TypedRest.CodeGeneration.Java](https://www.nuget.org/packages/TypedRest.CodeGeneration.Java/)
: Generate Kotlin and Java source code for [TypedRest for the JVM](https://github.com/TypedRest/TypedRest-Java) clients.

Implement `IBuilder<TEndpoint>` and add it to a `BuilderRegistry` to change the emitted code, or derive from `NamingStrategy` to change how types and properties are named.

A minimal tool looks like this:

```csharp
var reader = new OpenApiStreamReader(new OpenApiReaderSettings().AddTypedRest());
var doc = reader.Read(File.OpenRead("myapi.yml"), out _);

foreach (var type in doc.GenerateTypedRest(new GenerationOptions("MyService")
{
    Namespace = "MyCompany.MyService",
    GenerateInterfaces = true,
    GenerateDtos = true
}))
    type.WriteToDirectory("myclient/");
```

`GenerateTypedRest()` takes optional `PatternRegistry` and `BuilderRegistry` parameters for the extension points above.

For details see the **[API documentation](https://code-generation.typedrest.net/)**.
