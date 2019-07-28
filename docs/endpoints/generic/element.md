Endpoint for an individual resource.

| Method | Input  | Result  | HTTP Verb | Description                                      |
| ------ | ------ | ------- | --------- | ------------------------------------------------ |
| Exists | -      | Boolean | `HEAD`    | Determines whether the element currently exists. |
| Read   | -      | Entity  | `GET`     | Returns the entity.                              |
| Set    | Entity | Entity  | `PUT`     | Sets/replaces the entity.                        |
| Merge  | Entity | Entity  | `PATCH`   | Modifies the existing entity by merging changes. |
| Delete | -      | -       | `DELETE`  | Deletes the element.                             |
