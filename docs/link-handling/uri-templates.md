# URI templates

TypedRest supports URI templates as defined in [RFC 6570](https://tools.ietf.org/html/rfc6570). URI templates allow you to construct URIs dynamically by substituting variables into a template string.

## How URI templates work

A URI template contains placeholders (variables) that are replaced with actual values at runtime. For example:

- Template: `/users/{id}` with `{id:  "123"}` → `/users/123`
- Template: `/search{?q,limit}` with `{q: "test", limit: 10}` → `/search?q=test&limit=10`

## Server-provided templates

The server can provide URI templates via the [HTTP Link header](link-header.md):

```
Link: </users/{id}>; rel=user; templated=true
```

Or in [HAL](hal.md):

```json
{
    "_links": {
        "user": {
            "href": "/users/{id}",
            "templated": true
        }
    }
}
```

## Resolving templates

Use the `LinkTemplate` method to resolve a template with variables:

=== "C#"

    ```csharp
    // After receiving a response with the link template
    var userUri = endpoint.LinkTemplate("user", new { id = "123" });
    // Result: http://example.com/users/123

    // With query parameters
    var searchUri = endpoint.LinkTemplate("search", new { q = "test", limit = 10 });
    // Result: http://example.com/search?q=test&limit=10

    // Using a dictionary
    var variables = new Dictionary<string, object> { ["id"] = "123" };
    var uri = endpoint.LinkTemplate("user", variables);
    ```

=== "TypeScript"

    ```typescript
    // After receiving a response with the link template
    const userUri = endpoint.linkTemplate("user", { id: "123" });
    // Result: http://example.com/users/123

    // With query parameters
    const searchUri = endpoint.linkTemplate("search", { q: "test", limit: 10 });
    // Result: http://example.com/search?q=test&limit=10
    ```

## Default link templates

You can register default link templates that will be used when the server doesn't provide a template for a specific relation type:

=== "C#"

    ```csharp
    class MyEndpoint : EndpointBase
    {
        public MyEndpoint(IEndpoint referrer, string relativeUri) 
            : base(referrer, relativeUri)
        {
            // Set a default template for the "child" relation
            SetDefaultLinkTemplate("child", "./{id}");
        }
    }
    ```

=== "TypeScript"

    ```typescript
    class MyEndpoint extends Endpoint {
        constructor(referrer: Endpoint, relativeUri: string) {
            super(referrer, relativeUri);
            // Set a default template for the "child" relation
            this.setDefaultLinkTemplate("child", "./{id}");
        }
    }
    ```

### Built-in defaults

[Collection](../endpoints/generic/collection.md) and [Indexer](../endpoints/generic/indexer.md) endpoints automatically set a default link template of `./{id}` for the `child` relation. This allows you to access individual elements by ID without the server explicitly providing a link template.
