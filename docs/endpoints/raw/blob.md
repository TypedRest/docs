# Blob endpoint

Endpoint for a binary blob that can be downloaded or uploaded.

| Method   | Input       | Result      | HTTP Verb | Description                   |
| -------- | ----------- | ----------- | --------- | ----------------------------- |
| Download | -           | Binary data | `GET`     | Downloads the blob's content. |
| Upload   | Binary data | -           | `PUT`     | Uploads content for the blob. |
