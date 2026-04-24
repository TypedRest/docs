# Custom serializers

If none of the [built-in serializers](index.md) fits your needs, you can implement your own.

## Implement the interface

Implement the platform's `Serializer` interface (or appropriate base type):

=== "C#"

    On .NET, TypedRest uses `MediaTypeFormatter` from the `System.Net.Http.Formatting` family. Derive from `MediaTypeFormatter` (or one of its specializations) and override the read/write methods.

    ```csharp
    using System.Net.Http.Formatting;

    public class MySerializer : MediaTypeFormatter
    {
        public MySerializer()
        {
            SupportedMediaTypes.Add(new("application/my-format"));
        }

        public override bool CanReadType(Type type) => true;
        public override bool CanWriteType(Type type) => true;

        // Override ReadFromStreamAsync and WriteToStreamAsync to implement
        // custom (de)serialization logic.
    }
    ```

=== "Java"

    Implement the `net.typedrest.serializers.Serializer` interface or derive from `AbstractJsonSerializer` for JSON-based formats:

    ```java
    import net.typedrest.serializers.Serializer;
    import okhttp3.MediaType;
    import okhttp3.RequestBody;
    import okhttp3.ResponseBody;
    import java.util.List;

    public class MySerializer implements Serializer {
        private static final MediaType MEDIA_TYPE = MediaType.get("application/my-format");

        @Override
        public List<MediaType> getSupportedMediaTypes() {
            return List.of(MEDIA_TYPE);
        }

        @Override
        public <T> RequestBody serialize(T entity, Class<T> type) {
            // Custom serialization logic
            return RequestBody.create(toBytes(entity), MEDIA_TYPE);
        }

        @Override
        public <T> RequestBody serializeList(Iterable<T> entities, Class<T> type) {
            // Custom serialization logic
            return RequestBody.create(toBytes(entities), MEDIA_TYPE);
        }

        @Override
        public <T> T deserialize(ResponseBody body, Class<T> type) {
            // Custom deserialization logic
            return fromBytes(body.bytes(), type);
        }

        @Override
        public <T> List<T> deserializeList(ResponseBody body, Class<T> type) {
            // Custom deserialization logic
            return fromBytesAsList(body.bytes(), type);
        }
    }
    ```

=== "Kotlin"

    Implement the `net.typedrest.serializers.Serializer` interface or derive from `AbstractJsonSerializer` for JSON-based formats:

    ```kotlin
    import net.typedrest.serializers.Serializer
    import okhttp3.MediaType
    import okhttp3.MediaType.Companion.toMediaType
    import okhttp3.RequestBody
    import okhttp3.RequestBody.Companion.toRequestBody
    import okhttp3.ResponseBody

    class MySerializer : Serializer {
        private val mediaType = "application/my-format".toMediaType()

        override val supportedMediaTypes: List<MediaType> = listOf(mediaType)

        override fun <T> serialize(entity: T, type: Class<T>): RequestBody =
            toBytes(entity).toRequestBody(mediaType)

        override fun <T> serializeList(entities: Iterable<T>, type: Class<T>): RequestBody =
            toBytes(entities).toRequestBody(mediaType)

        override fun <T> deserialize(body: ResponseBody, type: Class<T>): T? =
            fromBytes(body.bytes(), type)

        override fun <T> deserializeList(body: ResponseBody, type: Class<T>): List<T>? =
            fromBytesAsList(body.bytes(), type)
    }
    ```

=== "TypeScript"

    Implement the `Serializer` interface from the `typedrest` package:

    ```typescript
    import { Serializer } from "typedrest";

    class MySerializer implements Serializer {
        readonly supportedMediaTypes = ["application/my-format"];

        serialize<T>(entity: T): string {
            // Custom serialization logic
            return JSON.stringify(entity);
        }

        deserialize<T>(text: string): T {
            // Custom deserialization logic
            return JSON.parse(text) as T;
        }
    }
    ```

## Content negotiation

The `supportedMediaTypes` property lists the MIME types this serializer can handle. TypedRest uses this information to:

- Set the `Content-Type` header when sending requests.
- Set the `Accept` header when receiving responses.
- Pick the right serializer for a response based on its `Content-Type` header (when multiple serializers are configured).

The first entry in the list is used as the default for outgoing requests.

## Register the serializer

Pass an instance of your serializer to the [entry endpoint](../endpoints/entry.md) constructor:

=== "C#"

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new MySerializer());
    ```

=== "Java"

    ```java
    EntryEndpoint endpoint = new EntryEndpoint(
        URI.create("http://example.com/"),
        new MySerializer());
    ```

=== "Kotlin"

    ```kotlin
    val endpoint = EntryEndpoint(
        URI.create("http://example.com/"),
        serializer = MySerializer())
    ```

=== "TypeScript"

    ```typescript
    const endpoint = new EntryEndpoint(
        new URL("http://example.com/"),
        new MySerializer());
    ```

All child endpoints created from this entry endpoint automatically inherit the serializer.
