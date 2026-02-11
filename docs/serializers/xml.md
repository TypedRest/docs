# XML

XML serialization is available in the .NET implementation of TypedRest.

=== "C#"
    TODO: The XML serializer uses .NET's built-in `System.Xml.Serialization.XmlSerializer`:

    ```csharp
    var endpoint = new EntryEndpoint(
        new Uri("http://example.com/"),
        serializer: new XmlSerializer());
    ```

## Content type

TODO: The serializer handles the following content types:

- `application/xml`
- `text/xml`
