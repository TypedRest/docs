# Streaming Collection endpoint

Endpoint for a collection of entities observable as an append-only stream using long-polling.


| Method         | Input       | Result        | HTTP Verb | Description                                                                    |
| -------------- | ----------- | ------------- | --------- | ------------------------------------------------------------------------------ |
| Get observable | -           | Entity stream | `GET`     | Provides an observable stream of elements.                                     |
| Get            | ID / Entity | Endpoint      | -         | Get an [Element endpoint](../generic/element.md) for a specific child element. |
| Read all       | -           | Entities      | `GET`     | Returns all entities in the collection.                                        |
| Read range     | Range       | Entities      | `GET`     | Returns all entities within a specific range of the collection.                |
| Create         | Entity      | -             | `POST`    | Adds an entity as a new element to the collection.                             |
| Create All     | Entities    | -             | `PATCH`   | Adds (or updates) multiple entities as elements in the collection.             |
| Set All        | Entities    | -             | `PUT`     | Replaces the entire content of the collection with new entities.               |

Extends [Collection endpoint](../generic/collection.md)
