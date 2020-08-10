# Collection endpoint

Endpoint for a collection of entities addressable as [Element endpoints](element.md).

| Method     | Input       | Result   | HTTP Verb | Description                                                         |
| ---------- | ----------- | -------- | --------- | ------------------------------------------------------------------- |
| Get        | ID / Entity | Endpoint | -         | Get an [Element endpoint](element.md) for a specific child element. |
| Read all   | -           | Entities | `GET`     | Returns all entities in the collection.                             |
| Read range | Range       | Entities | `GET`     | Returns all entities within a specific range of the collection.     |
| Create     | Entity      | -        | `POST`    | Adds an entity as a new element to the collection.                  |
| Create All | Entities    | -        | `PATCH`   | Adds (or updates) multiple entities as elements in the collection.  |
| Set All    | Entities    | -        | `PUT`     | Replaces the entire content of the collection with new entities.    |

Extends [Indexer endpoint](indexer.md)
