# Blob endpoint

Endpoint for a binary blob that can be downloaded or uploaded.

| Method   | Input       | Result      | HTTP Verb | Description                   |
| -------- | ----------- | ----------- | --------- | ----------------------------- |
| Download | -           | Binary data | `GET`     | Downloads the blob's content. |
| Upload   | Binary data | -           | `PUT`     | Uploads content for the blob. |

## Usage

=== "C#"

    ```csharp
    var avatar = new BlobEndpoint(client, "users/123/avatar");

    // Download to a stream
    Stream data = await avatar.DownloadAsync();
    
    // Or download to a file
    await avatar.DownloadAsync("avatar.jpg");

    // Upload from a file
    await avatar.UploadFromAsync("new-avatar.jpg");
    
    // Or upload from a stream
    using var fileStream = File.OpenRead("new-avatar.jpg");
    await avatar.UploadFromAsync(fileStream, mimeType: "image/jpeg");
    ```

=== "Java"

    ```java
    BlobEndpoint avatar = new BlobEndpointImpl(client, "users/123/avatar");

    // Download to a stream
    InputStream data = avatar.download();

    // Or download to a file
    avatar.download("avatar.jpg");

    // Upload from a file
    avatar.uploadFrom("new-avatar.jpg");

    // Or upload from a stream
    try (InputStream fileStream = new FileInputStream("new-avatar.jpg")) {
        avatar.uploadFrom(fileStream, "image/jpeg");
    }
    ```

=== "Kotlin"

    ```kotlin
    val avatar = BlobEndpointImpl(client, "users/123/avatar")

    // Download to a stream
    val data = avatar.download()

    // Or download to a file
    avatar.download("avatar.jpg")

    // Upload from a file
    avatar.uploadFrom("new-avatar.jpg")

    // Or upload from a stream
    FileInputStream("new-avatar.jpg").use { fileStream ->
        avatar.uploadFrom(fileStream, mimeType = "image/jpeg")
    }
    ```

=== "TypeScript"

    ```typescript
    const avatar = new BlobEndpoint(client, "users/123/avatar");

    // Download binary data
    const data = await avatar.download();

    // Upload from a file
    const file = new File([data], "new-avatar.jpg", { type: "application/jpeg" });
    await avatar.uploadFrom(file);

    // Or upload from a stream
    const blob = new Blob([data], { type: "image/jpeg" });
    await avatar.uploadFrom(blob);
    ```
