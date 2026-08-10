# Command-line tool

The `typedrest-codegen` tool writes the generated client to disk, as C#, Java, Kotlin or TypeScript files.

## Installation

Make sure you have the [.NET SDK](https://dotnet.microsoft.com/download) installed and run:

```bash
dotnet tool install -g typedrest-codegen
```

## Generating a client

=== "C#"

    ```bash
    typedrest-codegen generate -f myapi.yml -o myclient/ -s MyService --generate-interfaces --generate-dtos
    ```

    The generated code derives from the [`TypedRest`](https://www.nuget.org/packages/TypedRest/) NuGet package, so run `dotnet add package TypedRest` in the consuming project.

    Add [`TypedRest.Reactive`](https://www.nuget.org/packages/TypedRest.Reactive/) if the document asks for any [polling or streaming](custom-code.md) endpoints.

    Add [`TypedRest.SystemTextJson`](https://www.nuget.org/packages/TypedRest.SystemTextJson/) if you set the serializer to `system-text-json``.

=== "Java"

    ```bash
    typedrest-codegen generate -l java -f myapi.yml -o src/main/java/ -s MyService -n com.mycompany.myservice --generate-dtos
    ```

    The generated code derives from [TypedRest for the JVM](https://github.com/TypedRest/TypedRest-Java), so add the `net.typedrest:typedrest` and `net.typedrest:typedrest-serializers-jackson` dependencies to the consuming project.

    Generated Java DTOs carry JSpecify `@Nullable` annotations for optional properties, so that Kotlin consumers get real null safety instead of platform types. Add `org.jspecify:jspecify`, or turn the annotations off.

    Add `net.typedrest:typedrest-reactive` if the document asks for any [polling or streaming](custom-code.md) endpoints.

    !!! tip
        Prefer generating Kotlin if you can: TypedRest for the JVM is written in Kotlin, so that is the lower-friction direction. Use Java when your own source is Java.

=== "Kotlin"

    ```bash
    typedrest-codegen generate -l kotlin -f myapi.yml -o src/main/kotlin/ -s MyService -n com.mycompany.myservice --generate-dtos
    ```

    The generated code derives from [TypedRest for the JVM](https://github.com/TypedRest/TypedRest-Java), so add the `net.typedrest:typedrest` dependency to the consuming project.

    Generating Kotlin DTOs for the default `kotlinx` serializer needs both the `kotlin("plugin.serialization")` Gradle plugin **and** an explicit `org.jetbrains.kotlinx:kotlinx-serialization-json` dependency. TypedRest depends on it only as `implementation`, so it does not reach your compile classpath, and the plugin adds the compiler plugin but no dependency. Without it the generated `@Serializable` annotations do not resolve.

    Add `net.typedrest:typedrest-reactive` if the document asks for any [polling or streaming](custom-code.md) endpoints.

=== "TypeScript"

    ```bash
    typedrest-codegen generate -l typescript -f myapi.yml -o src/myclient/ -s MyService --generate-dtos
    ```

    The generated code imports from the [`typedrest`](https://www.npmjs.com/package/typedrest) package, so run `npm install typedrest` in the consuming project.

`--file`, `--output` and `--service-name` are required; the language defaults to C#. Everything else (the namespaces, interfaces, DTOs, [serializer](index.md#serializers) and C# language version) is optional, and a few options only apply to one language. See the [full reference](#option-reference) below.

Unlike the [source generator](source-generator.md), interfaces and DTOs are opt-in here. Generated endpoints reference the DTO types by name, so without `--generate-dtos` you have to provide those types yourself.

`--generate-interfaces` has no effect for TypeScript, which is structurally typed and whose TypedRest endpoints have no interfaces.

On the JVM it also changes the generated names: the interface takes the plain name and the class beside it gets the `Impl` suffix. Without it there is no interface and the class keeps the plain name.

```kotlin
// --generate-interfaces
interface ContactElementEndpoint : ElementEndpoint<Contact> { val note: ElementEndpoint<Note> }
open class ContactElementEndpointImpl(...) : ElementEndpointImpl<Contact>(...), ContactElementEndpoint

// without it
open class ContactElementEndpoint(...) : ElementEndpointImpl<Contact>(...)
```

Each generated type gets a file of its own. For TypeScript that means one module per type plus an `index.ts` re-exporting all of them; for Kotlin and Java it means a directory tree matching the package, so `--output` is the source root (`src/main/kotlin/`) rather than the package directory.

## Inspecting the inferred structure

The `pattern` command runs only the inference step and writes the result back into an OpenAPI document as an `x-typedrest` extension:

```bash
typedrest-codegen pattern --file myapi.yml --output myapi-annotated.yml
```

Without `--output` it overwrites the input file. The spec version and format of the output default to those of the input.

This is useful for two things: seeing what the tool inferred before generating any code, and hand-editing the result. When a document already contains an `x-typedrest` extension, `generate` uses it as-is instead of re-running the inference. See [Custom code](custom-code.md).

## Option reference

**[Command-line option reference](https://github.com/TypedRest/CodeGeneration/blob/master/src/TypedRest.CodeGeneration.Cli/README.md)**

You can also run `typedrest-codegen help generate` or `typedrest-codegen help pattern`.

## See also

- [Source generator](source-generator.md): the same generator, running during compilation (C# only)
- [Custom code](custom-code.md): for APIs the pattern inference does not cover
