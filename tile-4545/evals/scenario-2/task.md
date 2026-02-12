# File Upload Service

Build a file upload utility that sends files along with metadata to a server endpoint.

## Requirements

Your implementation should:

1. Upload one or more files to a specified endpoint
2. Include form fields with metadata (e.g., title, description, category)
3. Support uploading files from file paths
4. Handle the multipart/form-data encoding automatically
5. Return the server response indicating successful upload

## Test Cases

- Uploading a single text file with metadata fields (title: "document", category: "text") sends multipart request @test
- Uploading multiple files in a single request includes all files in the multipart data @test
- Form fields are correctly encoded alongside file attachments @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
