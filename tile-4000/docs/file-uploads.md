# File Uploads

Multipart form data and file attachment support for Node.js and browser environments.

## Capabilities

### File Attachment (Node.js)

Attach files to multipart/form-data requests in Node.js.

```javascript { .api }
/**
 * Attach a file to the request
 * @param field - Form field name
 * @param file - File path (string), Buffer, or ReadStream
 * @param options - Optional filename or options object
 * @returns Request instance for chaining
 */
Request.prototype.attach(field: string, file: string | Buffer | ReadStream, options?: string | object): Request;
```

**Options object:**

```javascript { .api }
interface AttachOptions {
  filename?: string;      // Custom filename
  contentType?: string;   // Custom content type
}
```

**Usage Examples:**

```javascript
// Attach file by path
request.post('https://api.example.com/upload')
  .attach('document', '/path/to/file.pdf');

// Attach with custom filename
request.post('https://api.example.com/upload')
  .attach('document', '/path/to/file.pdf', 'report.pdf');

// Attach with options
request.post('https://api.example.com/upload')
  .attach('document', '/path/to/file.pdf', {
    filename: 'report.pdf',
    contentType: 'application/pdf'
  });

// Attach Buffer
const buffer = Buffer.from('file content');
request.post('https://api.example.com/upload')
  .attach('document', buffer, 'file.txt');

// Attach ReadStream
const fs = require('fs');
const stream = fs.createReadStream('/path/to/large-file.zip');
request.post('https://api.example.com/upload')
  .attach('archive', stream, 'backup.zip');

// Multiple files
request.post('https://api.example.com/upload')
  .attach('photo1', '/path/to/image1.jpg')
  .attach('photo2', '/path/to/image2.jpg')
  .attach('photo3', '/path/to/image3.jpg');

// Multiple files with same field name
request.post('https://api.example.com/upload')
  .attach('photos', '/path/to/image1.jpg')
  .attach('photos', '/path/to/image2.jpg')
  .attach('photos', '/path/to/image3.jpg');
```

### File Attachment (Browser)

Attach files to multipart/form-data requests in browser environments.

```javascript { .api }
/**
 * Attach a file to the request (browser)
 * @param field - Form field name
 * @param file - File or Blob object
 * @param filename - Optional custom filename
 * @returns Request instance for chaining
 */
Request.prototype.attach(field: string, file: File | Blob, filename?: string): Request;
```

**Usage Examples:**

```javascript
// From file input element
const fileInput = document.querySelector('input[type="file"]');
const file = fileInput.files[0];

request.post('https://api.example.com/upload')
  .attach('avatar', file);

// With custom filename
request.post('https://api.example.com/upload')
  .attach('avatar', file, 'profile-picture.jpg');

// Multiple files from input
const fileInput = document.querySelector('input[type="file"][multiple]');
Array.from(fileInput.files).forEach(file => {
  request.post('https://api.example.com/upload')
    .attach('photos', file);
});

// From Blob
fetch('https://example.com/image.jpg')
  .then(res => res.blob())
  .then(blob => {
    request.post('https://api.example.com/upload')
      .attach('image', blob, 'downloaded-image.jpg');
  });
```

### Combining Files and Form Fields

Attach files along with additional form data.

**Usage Examples:**

```javascript
// Node.js: Upload file with metadata
request.post('https://api.example.com/upload')
  .field('title', 'My Document')
  .field('description', 'Important document')
  .field('category', 'reports')
  .attach('file', '/path/to/document.pdf');

// Browser: Upload image with form data
const fileInput = document.querySelector('input[type="file"]');
const file = fileInput.files[0];

request.post('https://api.example.com/upload')
  .field('username', 'john')
  .field('caption', 'My profile picture')
  .attach('avatar', file);

// Multiple files with shared metadata
request.post('https://api.example.com/batch-upload')
  .field('album', 'Vacation 2024')
  .field('tags', ['travel', 'summer', 'beach'])
  .attach('photos', '/path/to/photo1.jpg')
  .attach('photos', '/path/to/photo2.jpg')
  .attach('photos', '/path/to/photo3.jpg');
```

### Progress Monitoring

Monitor upload progress using event listeners.

**Usage Examples:**

```javascript
// Node.js: Monitor upload progress
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/large-file.zip')
  .on('progress', event => {
    console.log('Upload progress:', event.percent + '%');
    console.log('Uploaded:', event.loaded, 'bytes');
    console.log('Total:', event.total, 'bytes');
  })
  .then(res => {
    console.log('Upload complete');
  });

// Browser: Progress with UI update
const progressBar = document.querySelector('.progress-bar');

request.post('https://api.example.com/upload')
  .attach('file', file)
  .on('progress', event => {
    if (event.direction === 'upload') {
      progressBar.style.width = event.percent + '%';
      progressBar.textContent = Math.round(event.percent) + '%';
    }
  })
  .then(res => {
    console.log('Upload complete:', res.body);
  })
  .catch(err => {
    console.error('Upload failed:', err);
  });
```

## Important Notes

### Restrictions

- **Cannot mix `.send()` with `.attach()` or `.field()`**: Choose either `.send()` for JSON/text data OR `.field()`/`.attach()` for multipart forms
- **Content-Type is automatic**: Using `.attach()` or `.field()` automatically sets `Content-Type: multipart/form-data`
- **Browser file size limits**: Large file uploads may be restricted by browser memory limits

### Error Handling

```javascript
// Handle upload errors
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/file.pdf')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    if (err.code === 'ENOENT') {
      console.error('File not found');
    } else if (err.status === 413) {
      console.error('File too large');
    } else {
      console.error('Upload failed:', err.message);
    }
  });
```

### Best Practices

```javascript
// Set timeout for large uploads
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/large-file.zip')
  .timeout(60000) // 60 second timeout
  .then(res => console.log('Success'));

// Retry on failure
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/file.pdf')
  .retry(3) // Retry up to 3 times on failure
  .then(res => console.log('Success'));

// Custom headers for uploads
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/file.pdf')
  .set('Authorization', 'Bearer token123')
  .then(res => console.log('Success'));
```
