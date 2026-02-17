# Java / Kotlin

## Dependencies

Add one or more of the following Maven artifacts to your project:

[typedrest](https://central.sonatype.com/artifact/net.typedrest/typedrest)  
The main TypedRest library.

[typedrest-reactive](https://central.sonatype.com/artifact/net.typedrest/typedrest-reactive)  
Adds support for streaming with [ReactiveX (Rx)](http://reactivex.io/).  
Create endpoints using the types in the `net.typedrest.endpoints.reactive` package.

[typedrest-serializers-jackson](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-jackson)  
Adds support for serializing using [Jackson](https://github.com/FasterXML/jackson) instead of [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html).  
Pass `new JacksonJsonSerializer()` to the `EntryEndpoint` constructor.

[typedrest-serializers-moshi](https://central.sonatype.com/artifact/net.typedrest/typedrest-serializers-moshi)  
Adds support for serializing using [Moshi](https://github.com/square/moshi) instead of [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html).  
Pass `new MoshiJsonSerializer()` to the `EntryEndpoint` constructor.

## See also

- [API documentation](https://java.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/TypedRest-Java)
