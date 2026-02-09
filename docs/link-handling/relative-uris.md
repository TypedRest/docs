# Relative URIs

The most straightforward way to navigate between endpoints is using hard-coded relative URIs.  When creating child endpoints, you pass a relative URI that is resolved against the parent endpoint's URI.

## Basic usage

=== "C#"

    ```csharp
    var client = new EntryEndpoint(new Uri("http://example.com/api/"));
    var contacts = new CollectionEndpoint<Contact>(client, "contacts");
    // Results in: http://example.com/api/contacts
    ```

=== "TypeScript"

    ```typescript
    const client = new EntryEndpoint(new URL("http://example.com/api/"));
    const contacts = new CollectionEndpoint<Contact>(client, "contacts");
    // Results in: http://example.com/api/contacts
    ```

## Trailing slash pattern

Standard URI resolution can be tricky when the parent URI doesn't have a trailing slash. Consider:

- Base URI: `http://example.com/endpoint`
- Relative URI: `subresource`
- Standard resolution: `http://example.com/subresource` (replaces `endpoint`)

This is often not the desired behavior. TypedRest provides a non-standard `./` prefix pattern to handle this:

=== "C#"

    ```csharp
    var parent = new ElementEndpoint<MyEntity>(client, "endpoint");
    // parent.Uri = http://example.com/endpoint

    // Without ./ prefix - replaces last segment
    var child1 = new ActionEndpoint(parent, "subresource");
    // child1.Uri = http://example.com/subresource

    // With ./ prefix - appends to path
    var child2 = new ActionEndpoint(parent, "./subresource");
    // child2.Uri = http://example.com/endpoint/subresource
    ```

=== "TypeScript"

    ```typescript
    const parent = new ElementEndpoint<MyEntity>(client, "endpoint");
    // parent.uri = http://example.com/endpoint

    // Without ./ prefix - replaces last segment
    const child1 = new ActionEndpoint(parent, "subresource");
    // child1.uri = http://example.com/subresource

    // With ./ prefix - appends to path
    const child2 = new ActionEndpoint(parent, "./subresource");
    // child2.uri = http://example.com/endpoint/subresource
    ```

The `./` prefix tells TypedRest to ensure a trailing slash on the parent URI before resolving the relative URI. This makes the relative URI resolution behave as if the parent URI was `http://example.com/endpoint/`.

## Default links

You can also register default links that will be used when the server doesn't provide links for a specific relation type:

=== "C#"

    ```csharp
    class MyEndpoint : EndpointBase
    {
        public MyEndpoint(IEndpoint referrer, string relativeUri) 
            : base(referrer, relativeUri)
        {
            SetDefaultLink("related", "./related-resource");
        }
    }
    ```

=== "TypeScript"

    ```typescript
    class MyEndpoint extends Endpoint {
        constructor(referrer:  Endpoint, relativeUri: string) {
            super(referrer, relativeUri);
            this.setDefaultLink("related", "./related-resource");
        }
    }
    ```
