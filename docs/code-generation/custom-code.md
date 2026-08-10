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

## 2. Extend the generated code

In C#, all generated classes and interfaces are `partial`. Adding the methods you need for special cases in a separate file of the same namespace usually beats fighting the generator.

In TypeScript, derive from the generated class and use your subclass in place of the generated one.

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
