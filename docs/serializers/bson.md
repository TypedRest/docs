# BSON

BSON (Binary JSON) serialization is available in the .NET implementation of TypedRest.

=== "C#"

    The BSON serializer provides efficient binary serialization using [Newtonsoft.Json](https://www.newtonsoft.com/json)'s BSON support:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new BsonSerializer());
    ```

    The BSON serializer uses the same Newtonsoft.Json settings as the default JSON serializer:

    - Camel-case property naming
    - String enums with camel-case naming
    - Null values are not serialized
    - Automatic type name handling

## Content type

TODO: The serializer handles the `application/bson` content type.
