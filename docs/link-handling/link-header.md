# HTTP Link header

TypedRest can extract links from HTTP `Link` headers as defined in [RFC 8288](https://tools.ietf. org/html/rfc8288). This enables HATEOAS-style navigation where the server provides links to related resources in response headers.

## Header format

The HTTP `Link` header follows this format:

```
Link: <target-uri>; rel="relation-type"; title="optional title"
```

Multiple links can be specified in a single header, separated by commas:

```
Link: <http://example.com/users>; rel=self, <http://example.com/user/123>; rel=child
```

## Extracting links

TypedRest automatically extracts links from response headers after each request. You can then resolve them:

=== "C#"

    ```csharp
    // Perform a request
    await endpoint.ReadAsync();

    // Get all links with a specific relation type
    var links = endpoint.GetLinks("child");
    foreach (var (uri, title) in links)
    {
        Console.WriteLine($"URI: {uri}, Title: {title}");
    }

    // Get a single link
    Uri collectionUri = endpoint.Link("next");
    ```

=== "TypeScript"

    ```typescript
    // Perform a request
    await endpoint.read();

    // Get all links with a specific relation type
    const links = endpoint.getLinks("child");
    for (const { uri, title } of links) {
        console.log(`URI: ${uri}, Title: ${title}`);
    }

    // Get a single link
    const collectionUri = endpoint.link("next");
    ```

## Templated links

TypedRest extends the standard Link header format to support [URI templates](uri-templates.md). Templates are indicated with `templated=true`:

```
Link: </users/{id}>; rel=user; templated=true
```

This is a non-standard extension, but it allows servers to provide dynamic link templates via headers without needing HAL or another hypermedia format in the response body.

=== "C#"

    ```csharp
    // Server response:
    // Link: </users/{id}>; rel=user; templated=true

    var userUri = endpoint.LinkTemplate("user", new { id = "123" });
    // Result: http://example.com/users/123
    ```

=== "TypeScript"

    ```typescript
    // Server response:
    // Link: </users/{id}>; rel=user; templated=true

    const userUri = endpoint.linkTemplate("user", { id: "123" });
    // Result: http://example.com/users/123
    ```

## Link header attributes

The following attributes are supported:

| Attribute    | Description                                        |
|--------------|----------------------------------------------------|
| `rel`        | The relation type (required)                       |
| `title`      | Human-readable title for the link (optional)       |
| `templated`  | Indicates a URI template when set to `true` (non-standard) |
