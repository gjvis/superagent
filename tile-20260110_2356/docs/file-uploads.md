# File Uploads

Upload files using multipart/form-data with SuperAgent.

## Capabilities

### Attach Files

Attach files to a request for upload.

```javascript { .api }
/**
 * Attach file to request
 * @param {string} field - Form field name
 * @param {string|Buffer|Blob|File|ReadStream} file - File to attach
 * @param {string|object} [options] - Filename string or options object
 * @param {string} [options.filename] - Override filename
 * @returns {Request} Request instance for chaining
 */
Request.prototype.attach = function(field, file, options);
```

**Usage Examples:**

```javascript
// Node.js: Attach file by path
request
  .post('/api/upload')
  .attach('file', '/path/to/document.pdf')
  .end(callback);

// Node.js: Attach file with custom filename
request
  .post('/api/upload')
  .attach('file', '/path/to/document.pdf', 'custom-name.pdf')
  .end(callback);

// Node.js: Attach Buffer
const buffer = Buffer.from('file contents');
request
  .post('/api/upload')
  .attach('file', buffer, 'filename.txt')
  .end(callback);

// Node.js: Attach ReadStream
const fs = require('fs');
const stream = fs.createReadStream('/path/to/file.pdf');
request
  .post('/api/upload')
  .attach('document', stream)
  .end(callback);

// Browser: Attach File object from input
const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];
request
  .post('/api/upload')
  .attach('file', file)
  .end(callback);

// Browser: Attach Blob
const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
request
  .post('/api/upload')
  .attach('file', blob, 'hello.txt')
  .end(callback);

// Attach with options object
request
  .post('/api/upload')
  .attach('file', buffer, {
    filename: 'custom.txt',
    contentType: 'text/plain'
  })
  .end(callback);
```

### Add Form Fields

Add form fields to multipart request alongside file attachments.

```javascript { .api }
/**
 * Add form field
 * @param {string} name - Field name
 * @param {string|Blob} value - Field value
 * @returns {Request} Request instance for chaining
 */
Request.prototype.field = function(name, value);
```

**Usage Examples:**

```javascript
// Single field
request
  .post('/api/upload')
  .field('name', 'John Doe')
  .attach('file', fileBuffer, 'doc.pdf')
  .end(callback);

// Multiple fields
request
  .post('/api/upload')
  .field('name', 'John Doe')
  .field('email', 'john@example.com')
  .field('category', 'documents')
  .attach('file', fileBuffer, 'doc.pdf')
  .end(callback);

// Fields and multiple files
request
  .post('/api/upload')
  .field('description', 'My files')
  .attach('file1', file1, 'document.pdf')
  .attach('file2', file2, 'image.jpg')
  .end(callback);
```

### Multiple File Upload

Upload multiple files in a single request.

**Usage Examples:**

```javascript
// Multiple files with same field name
request
  .post('/api/upload')
  .attach('files', file1, 'doc1.pdf')
  .attach('files', file2, 'doc2.pdf')
  .attach('files', file3, 'doc3.pdf')
  .end(callback);

// Multiple files with different field names
request
  .post('/api/upload')
  .attach('document', docFile, 'contract.pdf')
  .attach('image', imageFile, 'photo.jpg')
  .attach('video', videoFile, 'clip.mp4')
  .end(callback);

// Loop through files
const files = [file1, file2, file3];
const req = request.post('/api/upload');
files.forEach((file, index) => {
  req.attach('files', file, `file-${index}.pdf`);
});
req.end(callback);
```

### Complete Upload Examples

Full examples combining fields and file attachments.

**Usage Examples:**

```javascript
// Node.js: Upload with metadata
request
  .post('/api/documents')
  .field('title', 'Quarterly Report')
  .field('author', 'John Doe')
  .field('date', '2024-01-15')
  .attach('document', '/path/to/report.pdf')
  .end((err, res) => {
    if (err) {
      console.error('Upload failed:', err);
    } else {
      console.log('Uploaded:', res.body);
    }
  });

// Browser: Upload from file input with form data
const form = document.getElementById('uploadForm');
const fileInput = document.getElementById('fileInput');

form.addEventListener('submit', (e) => {
  e.preventDefault();

  const file = fileInput.files[0];

  request
    .post('/api/upload')
    .field('name', form.elements.name.value)
    .field('description', form.elements.description.value)
    .attach('file', file)
    .on('progress', (e) => {
      console.log('Upload progress:', e.percent + '%');
    })
    .end((err, res) => {
      if (err) {
        console.error('Upload failed:', err);
      } else {
        console.log('Upload successful:', res.body);
      }
    });
});

// Node.js: Upload multiple files with authentication
request
  .post('/api/batch-upload')
  .auth('username', 'password')
  .field('batch_id', '12345')
  .attach('files', file1, 'report1.pdf')
  .attach('files', file2, 'report2.pdf')
  .attach('files', file3, 'report3.pdf')
  .end(callback);

// Upload with progress tracking
request
  .post('/api/upload')
  .attach('file', largeFile)
  .on('progress', (event) => {
    if (event.direction === 'upload') {
      console.log(`Uploaded: ${event.loaded} / ${event.total} bytes`);
      if (event.percent) {
        console.log(`Progress: ${event.percent}%`);
      }
    }
  })
  .end(callback);
```

### Error Handling

Handle errors during file upload.

**Usage Examples:**

```javascript
request
  .post('/api/upload')
  .attach('file', fileBuffer, 'doc.pdf')
  .end((err, res) => {
    if (err) {
      if (err.status === 413) {
        console.error('File too large');
      } else if (err.status === 415) {
        console.error('Unsupported file type');
      } else if (err.timeout) {
        console.error('Upload timed out');
      } else {
        console.error('Upload failed:', err.message);
      }
      return;
    }

    console.log('Upload successful');
    console.log('File ID:', res.body.fileId);
    console.log('File URL:', res.body.url);
  });
```

### Important Notes

- **Cannot mix `.send()` and `.attach()`**: Using `.attach()` or `.field()` automatically creates a multipart/form-data request. You cannot use `.send()` on the same request.
- **Content-Type**: The Content-Type header is automatically set to `multipart/form-data` with appropriate boundary when using `.attach()` or `.field()`.
- **File streams (Node.js)**: When passing a file path, SuperAgent automatically creates a read stream. You can also pass a stream directly.
- **Browser File objects**: In browsers, use the File API to get File objects from `<input type="file">` elements.
- **Progress events**: Listen to `progress` events to track upload progress (see Progress Events documentation).

### Multipart Data Format

When using `.attach()` or `.field()`, the request automatically becomes `multipart/form-data`:

```javascript
// This request
request
  .post('/api/upload')
  .field('name', 'John')
  .attach('file', buffer, 'doc.pdf')

// Sends as multipart/form-data:
// Content-Type: multipart/form-data; boundary=----WebKitFormBoundary...
//
// ------WebKitFormBoundary...
// Content-Disposition: form-data; name="name"
//
// John
// ------WebKitFormBoundary...
// Content-Disposition: form-data; name="file"; filename="doc.pdf"
// Content-Type: application/pdf
//
// [binary file data]
// ------WebKitFormBoundary...--
```
