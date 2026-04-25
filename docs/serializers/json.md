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
    var endpoint = new EntryEndpoint(new Uri("http://example.com/")); // Uses NewtonsoftJsonSerializer by default
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

    You can also use the [System.Text.Json](https://learn.microsoft.com/en-us/dotnet/api/system.text.json) serializer with the [TypedRest.SystemTextJson](https://www.nuget.org/packages/TypedRest.SystemTextJson) NuGet package.
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

=== "Java"

    **kotlinx.serialization (Default)**

    The default serializer uses [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html). Entity classes are typically defined in Kotlin with the `@Serializable` annotation.

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/")); // Uses KotlinxJsonSerializer by default
    ```

    **Jackson**

    For Java POJOs or more control over serialization, add the [typedrest-serializers-jackson](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson) dependency and pass a `JacksonJsonSerializer`:

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), null, new JacksonJsonSerializer());
    ```

    To customize the `JsonMapper`:

    ```java
    JsonMapper mapper = JsonMapper.builder()
        .addModule(new KotlinModule.Builder().build())
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .build();
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), null, new JacksonJsonSerializer(mapper));
    ```

    **Moshi**

    Add the [typedrest-serializers-moshi](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-moshi) dependency and pass a `MoshiJsonSerializer`:

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), null, new MoshiJsonSerializer());
    ```

=== "Kotlin"

    **kotlinx.serialization (Default)**

    The default serializer uses [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html). Annotate entity classes with `@Serializable`:

    ```kotlin
    import kotlinx.serialization.Serializable

    @Serializable
    data class Contact(val name: String)
    ```

    ```kotlin
    val endpoint = EntryEndpoint(URI.create("http://example.com/")) // Uses KotlinxJsonSerializer by default
    ```

    **Jackson**

    Add the [typedrest-serializers-jackson](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson) dependency and pass a `JacksonJsonSerializer`:

    ```kotlin
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = JacksonJsonSerializer()
    )
    ```

    To customize the `JsonMapper`:

    ```kotlin
    val mapper = JsonMapper.builder()
        .addModule(kotlinModule())
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .build()
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = JacksonJsonSerializer(mapper)
    )
    ```

    **Moshi**

    Add the [typedrest-serializers-moshi](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-moshi) dependency and pass a `MoshiJsonSerializer`:

    ```kotlin
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = MoshiJsonSerializer()
    )
    ```

    To customize the `Moshi` instance:

    ```kotlin
    val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .add(Date::class.java, Rfc3339DateJsonAdapter())
        .build()
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = MoshiJsonSerializer(moshi)
    )
    ```

=== "TypeScript"

    **Native JSON (Default)**

    The default serializer uses the native `JSON.stringify()` and `JSON.parse()` methods.

    ```typescript
    const endpoint = new EntryEndpoint(new URL("http://example.com/")); // Uses JsonSerializer by default
    ```

## Content type

The serializer handles the `application/json` content type. TypedRest also automatically treats custom media types ending in `+json` (e.g., `application/vnd.api+json`, `application/hal+json`) as JSON and deserializes them using the configured JSON serializer.
