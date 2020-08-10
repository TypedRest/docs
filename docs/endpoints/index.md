# Overview

Endpoints are the main building blocks of TypedRest. An endpoint represents an URI that provides methods for interacting with a specific resource. The type of the endpoint determines the available methods. An endpoint can also provide child endpoints (represent child URIs) via composition.

TypedRest provides a number of endpoint types modelling common REST patterns. Most APIs can be consumed by either directly using these types or by deriving from them and adding a few additional methods for special use cases.

**[Entry](entry.md)** endpoints represent the top-level URI of APIs.

**Generic** endpoints allow you to model collections and elements:

- [Element](generic/element.md) - individual resource (read, write, delete)
- [Collection](generic/collection.md) - collection of entities addressable as elements (read all, create, get child by ID)
- [Indexer](generic/indexer.md) - address child endpoints by ID

**Raw** endpoints allow you to transmit binary data rather than serialized objects:

- [Upload](raw/upload.md) - accept binary uploads
- [Blob](raw/blob.md) - download or upload a binary blob

**Reactive** endpoints allow you to receive data as push streams rather than explicitly pulling:

- [Polling](reactive/polling.md) - poll resource for state changes
- [Streaming](reactive/streaming.md) - stream of entities via persistent connection
- [Streaming Collection](reactive/streaming-collection.md) - collection of entities observable as append-only stream

**RPC** endpoints allow you to interact with non-RESTful resources that act like callable functions:

- [Action](rpc/action.md) - no input or output
- [Consumer](rpc/consumer.md) - takes entity as input
- [Producer](rpc/producer.md) - produces entity as output
- [Function](rpc/function.md) - takes entity as input and produces entity as output

The constructors of all endpoints except entry endpoints take a `referrer` parameter. This is uses to inherit relative URI bases and configuration such as [error handling](../error-handling/index.md) and [link handling](../link-handling/index.md).
