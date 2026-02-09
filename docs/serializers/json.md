# JSON

JSON is the default serialization format in TypedRest.

=== "C#"

    **Newtonsoft.Json (Default)**

    The default serializer uses [Newtonsoft.Json](https://www.newtonsoft.com/json) with the following settings:

    - Camel-case property naming
    - String enums with camel-case naming
    - Null values are not serialized
    - Automatic type name handling

    ```csharp
    var endpoint = new EntryEndpoint(new Uri("http://example.com/"));
    // Uses NewtonsoftJsonSerializer by default
    ```

    To customize the serializer settings:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new NewtonsoftJsonSerializer
        {
            SerializerSettings =
            {
                DateFormatString = "yyyy-MM-dd"
            }
        });
    ```

    **System.Text.Json**

    You can also use the [System.Text.Json](https://learn.microsoft.com/en-us/dotnet/api/system.text.json) serializer with the [TypedRest.SystemTextJson](https://www.nuget.org/packages/TypedRest.SystemTextJson) NuGet package:

    !!! note
        The `TypedRest.SystemTextJson` package version should match your main `TypedRest` package version. Both packages follow the same versioning scheme.

    Default settings:

    - Web defaults (camel-case property naming)
    - Null values are not serialized when writing

    Basic usage:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new SystemTextJsonSerializer());
    ```

    To customize the serializer options:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new SystemTextJsonSerializer
        {
            Options =
            {
                WriteIndented = true,
                Converters = {new JsonStringEnumConverter() }
            }
        });
    ```

=== "TypeScript"

    TypeScript uses the native `JSON.stringify()` and `JSON.parse()` methods:

    ```typescript
    const endpoint = new EntryEndpoint(new URL("http://example.com/"));
    // Uses JsonSerializer by default
    ```

    **Custom serializers**

    You can implement the `Serializer` interface for custom serialization:

    ```typescript
    import { Serializer } from "typedrest";

    class MySerializer implements Serializer {
        readonly supportedMediaTypes = ["application/json"];

        serialize<T>(entity: T): string {
            // Custom serialization logic
            return JSON.stringify(entity);
        }

        deserialize<T>(text: string): T {
            // Custom deserialization logic
            return JSON.parse(text) as T;
        }
    }

    const endpoint = new EntryEndpoint(
        new URL("http://example.com/"),
        new MySerializer());
    ```
