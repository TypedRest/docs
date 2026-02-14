# Upload endpoint

Endpoint that accepts binary uploads.

| Method | Input       | Result | HTTP Verb | Description                   |
| ------ | ----------- | ------ | --------- | ----------------------------- |
| Upload | Binary data | -      | `POST`    | Uploads data to the endpoint. |

## Usage

=== "C#"

    ```csharp
    var attachments = new UploadEndpoint(client, "issues/456/attachments");

    // Upload from a file
    await attachments.UploadFromAsync("document.pdf");

    // Or upload from a stream
    using var fileStream = File.OpenRead("document.pdf");
    await attachments.UploadFromAsync(fileStream, mimeType: "application/pdf");
    ```

=== "Java"

    ```java
    UploadEndpoint attachments = new UploadEndpointImpl(client, "issues/456/attachments");

    // Upload from a file
    attachments.uploadFrom("document.pdf");

    // Or upload from a stream
    try (InputStream fileStream = new FileInputStream("document.pdf")) {
        attachments.uploadFrom(fileStream, "application/pdf");
    }
    ```

=== "Kotlin"

    ```kotlin
    val attachments = UploadEndpointImpl(client, "issues/456/attachments")

    // Upload from a file
    attachments.uploadFrom("document.pdf")

    // Or upload from a stream
    FileInputStream("document.pdf").use { fileStream ->
        attachments.uploadFrom(fileStream, mimeType = "application/pdf")
    }
    ```

=== "TypeScript"

    ```typescript
    const attachments = new UploadEndpoint(client, "issues/456/attachments");

    // Upload from a file
    const file = new File([data], "document.pdf", { type: "application/pdf" });
    await attachments.uploadFrom(file);

    // Or upload from a stream
    const blob = new Blob([data], { type: "application/pdf" });
    await attachments.uploadFrom(blob);
    ```
