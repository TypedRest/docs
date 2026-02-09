# Link handling

TypedRest supports a variety of different ways to establish links between [endpoints](../endpoints/index.md):

- [Hard-coded relative URIs](relative-uris.md)
- [URI templates](uri-templates.md) for dynamic URL construction
- [Links via HTTP Link Header](link-header.md) transmitted in response headers
- [Links encoded in resources via HAL](hal.md) for hypermedia-driven APIs

## Link resolution

All endpoints provide methods for resolving links:

=== "C#"

    - `GetLinks(rel)` - Resolves all links with a specific relation type
    - `Link(rel)` - Resolves a single link with a specific relation type
    - `LinkTemplate(rel, variables)` - Resolves a link template with variables

=== "TypeScript"

    - `getLinks(rel)` - Resolves all links with a specific relation type
    - `link(rel)` - Resolves a single link with a specific relation type
    - `linkTemplate(rel, variables)` - Resolves a link template with variables

These methods use cached data from the last response. On cache miss, they perform a lazy lookup using HTTP `HEAD`.
