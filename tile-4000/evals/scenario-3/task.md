# File Upload Service

Create a file upload utility that sends files to a server with metadata.

## Requirements

Implement a function `uploadFile(filePath, metadata)` that:

1. Uploads a file from the given file path to `https://api.storage.com/upload`
2. Includes the file in a field named "document"
3. Sends additional form fields from the metadata object (e.g., `title`, `description`, `tags`)
4. Supports uploading multiple files in a single request
5. Returns the server response with upload confirmation

Example usage:
```javascript
await uploadFile('./report.pdf', {
  title: 'Annual Report',
  description: 'Q4 2023 results',
  category: 'financial'
})

// Multiple files
await uploadFiles([
  { path: './photo1.jpg', fieldName: 'photos' },
  { path: './photo2.jpg', fieldName: 'photos' }
], { album: 'Vacation 2023' })
```

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Upload single PDF file with metadata
Expected: Sends multipart/form-data request with file and form fields

@test Input: Upload multiple image files
Expected: All files included in multipart request

@test Input: Metadata with multiple fields (title, description, category)
Expected: All metadata fields included as form fields
