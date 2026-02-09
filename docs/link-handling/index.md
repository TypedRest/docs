# Link handling

TypedRest supports a variety of different ways to establish links between [endpoints](../endpoints/index.md):

- [Hard-coded relative URIs](relative-uris.md)
- [URI templates](uri-templates.md) for dynamic URL construction
- [Links via HTTP Link Header](link-header.md) transmitted in response headers
- [Links encoded in resources via HAL](hal.md) for hypermedia-driven APIs

## Link resolution

All endpoints provide methods for resolving links:

=== "C#"

    ```csharp
    // Resolves all links with a specific relation type.
    IReadOnlyList<(Uri uri, string? title)> GetLinks(string rel);

    // Resolves a single link with a specific relation type.
    Uri Link(string rel);

    // Resolves a link template with a specific relation type.
    Uri LinkTemplate(string rel, IDictionary<string, object> variables);
    ```

=== "Java"

    ```java
    // Resolves all links with a specific relation type.
    List<Pair<URI, String>> getLinks(String rel);

    // Resolves a single link with a specific relation type.
    URI link(String rel);

    // Resolves a link template with a specific relation type.
    URI linkTemplate(String rel, Map<String, Object> variables);
    ```

=== "Kotlin"

    ```kotlin
    // Resolves all links with a specific relation type.
    fun getLinks(rel: String): List<Pair<URI, String?>>

    // Resolves a single link with a specific relation type.
    fun link(rel: String): URI

    // Resolves a link template with a specific relation type.
    fun linkTemplate(rel: String, variables: Map<String, Any>): URI
    ```

=== "TypeScript"

    ```typescript
    // Resolves all links with a specific relation type.
    getLinks(rel: string): { uri: URL; title?: string; }[];

    // Resolves a single link with a specific relation type.
    link(rel: string): URL;

    // Resolves a link template with a specific relation type.
    linkTemplate(rel: string, variables: { [key: string]: any; }): URL;
    ```

These methods use cached data from the last response. On cache miss, they perform a lazy lookup using HTTP `HEAD`.
