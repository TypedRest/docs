Endpoint for a resource that can be polled for state changes.

| Method         | Input  | Result        | HTTP Verb | Description                                      |
| -------------- | ------ | ------------- | --------- | ------------------------------------------------ |
| Get observable | -      | Entity stream | `GET`     | Provides an observable stream of entity states.  |
| Exists         | -      | Boolean       | `HEAD`    | Determines whether the element currently exists. |
| Read           | -      | Entity        | `GET`     | Returns the entity.                              |
| Set            | Entity | Entity        | `PUT`     | Sets/replaces the entity.                        |
| Merge          | Entity | Entity        | `PATCH`   | Modifies the existing entity by merging changes. |
| Delete         | -      | -             | `DELETE`  | Deletes the element.                             |

Extends [Element](../generic/element.md)
