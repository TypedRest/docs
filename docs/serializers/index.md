# Serializers

Serializers control how entities are serialized when sending requests and deserialized when receiving responses.

TypedRest provides built-in serializers for various transport formats:

- [JSON](json.md)
- [BSON](bson.md)
- [XML](xml.md)

You can also create and use custom serializers.

## Inheritance

When creating [endpoints](../endpoints/index.md), the serializer is automatically inherited from the parent (referrer) endpoint:

=== "C#"

    ```csharp
    var client = new EntryEndpoint(new Uri("http://example.com/"));
    client.Serializer = new SystemTextJsonSerializer();

    var contacts = new CollectionEndpoint<Contact>(client, relativeUri: "./contacts");
    // contacts.Serializer is automatically set to the same SystemTextJsonSerializer
    ```

=== "TypeScript"

    ```typescript
    const client = new EntryEndpoint(new URL("http://example.com/"));
    client.serializer = new CustomSerializer();

    const contacts = new CollectionEndpoint<Contact>(client, "./contacts");
    // contacts.serializer is automatically set to the same CustomSerializer
    ```

This inheritance ensures consistent serialization behavior across your entire endpoint hierarchy.
