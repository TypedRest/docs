# Source generator

The source generator builds the client while your project compiles. The generated code never touches your source tree; it exists only in the compilation.

It runs inside the C# compiler and therefore only generates C# clients. Use the [command-line tool](cli.md) for Java, Kotlin and TypeScript.

## Getting started

### Install the package

```bash
dotnet add package TypedRest.SourceGenerator # generates source code on build
dotnet add package TypedRest # library that the generated code depends on
```

### Reference the spec file

Add the OpenAPI/Swagger document to your project file as a `TypedRestOpenApi` item:

```xml
<ItemGroup>
  <TypedRestOpenApi Include="myapi.yml" ServiceName="MyService" Namespace="MyCompany.MyService" />
</ItemGroup>
```

`ServiceName` is required and names the entry endpoint, e.g. `MyServiceClient`. All other settings are optional; see the [full reference](#configuration-reference) below.

### Use the client

The generated types are available to the rest of the project immediately:

```csharp
using MyCompany.MyService;

var client = new MyServiceClient(new Uri("https://example.com/api/"));
```

By default the generator also emits interfaces for all endpoints and DTO classes for all schemas in the document, so there is nothing else to write by hand.

## Inspecting the generated code

Set `<EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>` in your project file. The generated files are then written to `obj/` in addition to being compiled.

## Multiple specs in one project

Add one `TypedRestOpenApi` item per document. Give each its own `ServiceName` and `Namespace`, otherwise the generated types will collide:

```xml
<ItemGroup>
  <TypedRestOpenApi Include="billing.yml" ServiceName="Billing" Namespace="MyCompany.Billing" />
  <TypedRestOpenApi Include="shipping.yml" ServiceName="Shipping" Namespace="MyCompany.Shipping" />
</ItemGroup>
```

Settings shared by all documents in a project can be set once as MSBuild properties instead, e.g. `<TypedRestGenerateDtos>false</TypedRestGenerateDtos>`.

## Choosing a serializer

Generated DTOs are annotated for Newtonsoft.Json by default, matching what [TypedRest uses out of the box](../serializers/json.md). If you configure your endpoints with `System.Text.Json` instead, generate for it too:

```xml
<PropertyGroup>
  <TypedRestSerializer>system-text-json</TypedRestSerializer>
</PropertyGroup>
```

This has to match your runtime configuration. The two read entirely different attributes, so a DTO annotated for one silently falls back to its C# member names under the other, changing the wire format without any error.

## Limitations

- The generator runs inside the compiler, so it requires a .NET SDK that ships Roslyn 5.6 or newer (.NET SDK 10.0.302+). Use the [command-line tool](cli.md) for older toolchains.
- Only local `$ref`s are resolved. Bundle multi-file specs into a single document first.

## Configuration reference

The complete list of `TypedRestOpenApi` metadata, the matching MSBuild properties and the `TRCG*` diagnostics is kept next to the generator source, so that it stays in sync with the code that reads it:

**[Source generator configuration reference](https://github.com/TypedRest/CodeGeneration/blob/master/src/TypedRest.SourceGenerator/README.md)**

## See also

- [Command-line tool](cli.md): the same generator, writing files to disk
- [Custom code](custom-code.md): for APIs the pattern inference does not cover
