# XML

XML serialization is available in the .NET and Java/Kotlin implementations of TypedRest.

=== "C#"
    The XML serializer uses .NET's built-in `System.Xml.Serialization.XmlSerializer`:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new XmlSerializer());
    ```

=== "Java"

    **Jackson XML**

    Add the [typedrest-serializers-jackson-xml](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson-xml) dependency and pass a `JacksonXmlSerializer`:

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), (HttpCredentials) null, new JacksonXmlSerializer());
    ```

    To customize the `XmlMapper`:

    ```java
    XmlMapper mapper = XmlMapper.builder()
        .addModule(new KotlinModule.Builder().build())
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .build();
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), (HttpCredentials) null, new JacksonXmlSerializer(mapper));
    ```

    **XmlUtil**

    Add the [typedrest-serializers-xmlutil](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-xmlutil) dependency and pass an `XmlUtilSerializer`:

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(URI.create("http://example.com/"), (HttpCredentials) null, new XmlUtilSerializer());
    ```

    !!! warning "Not usable with entities written in Java"

        XmlUtil builds on [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html), which generates its serializers with a Kotlin compiler plugin. It only works for entity classes written in Kotlin and annotated with `@Serializable`. A Java class has no generated serializer, and deserializing one fails at runtime rather than at compile time.

        If your entities are Java classes, use Jackson XML above instead.

=== "Kotlin"

    **Jackson XML**

    Add the [typedrest-serializers-jackson-xml](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson-xml) dependency and pass a `JacksonXmlSerializer`:

    ```kotlin
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = JacksonXmlSerializer()
    )
    ```

    To customize the `XmlMapper`:

    ```kotlin
    val mapper = XmlMapper.builder()
        .addModule(kotlinModule())
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .build()
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = JacksonXmlSerializer(mapper)
    )
    ```

    **XmlUtil**

    Add the [typedrest-serializers-xmlutil](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-xmlutil) dependency and annotate entity classes with `@Serializable`:

    ```kotlin
    import kotlinx.serialization.Serializable

    @Serializable
    data class Contact(val name: String)
    ```

    ```kotlin
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = XmlUtilSerializer()
    )
    ```

    To customize the `XML` instance:

    ```kotlin
    val xml = XML.recommended_1_0 {
        indentString = "  "
    }
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = XmlUtilSerializer(xml)
    )
    ```

## Content type

The serializer handles the following content types:

- `application/xml`
- `text/xml`
