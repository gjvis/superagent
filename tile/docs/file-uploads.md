# File Uploads and Form Data

SuperAgent provides comprehensive support for multipart form data uploads, allowing you to send files and form fields in both Node.js and browser environments. The library uses `FormData` internally to handle multipart/form-data encoding and provides a fluent API for attaching files and adding form fields.

## Platform Support

- **Node.js**: Full support via `form-data` package with file system access
- **Browser**: Full support via native `FormData` API (File, Blob, FormData objects)

## Capabilities

### Attaching Files (Node.js)

Upload files from the filesystem using the `.attach()` method. This method is Node.js specific and creates read streams for efficient file transfer.

```javascript { .api }
/**
 * Attach a file for multipart upload
 * @param field - Form field name
 * @param file - File path string, ReadStream, or Buffer
 * @param options - Optional filename or options object
 * @returns Request instance for chaining
 * @example
 * // Attach file by path
 * request.post('/upload')
 *   .attach('document', '/path/to/file.pdf')
 *   .end(callback);
 *
 * // Attach with custom filename
 * request.post('/upload')
 *   .attach('document', '/path/to/file.pdf', 'report.pdf')
 *   .end(callback);
 *
 * // Attach Buffer with options
 * request.post('/upload')
 *   .attach('image', buffer, { filename: 'photo.jpg' })
 *   .end(callback);
 */
Request.prototype.attach = function(
  field: string,
  file: string | Buffer | ReadStream,
  options?: string | { filename?: string }
): Request;
```

**Node.js Implementation Details:**

- Accepts file path strings, creates `fs.ReadStream` automatically
- Accepts `Buffer` objects for in-memory file uploads
- Accepts `fs.ReadStream` instances for streaming uploads
- Automatically detects filename from path if not provided
- Uses the `form-data` package to construct multipart boundaries
- Emits `'error'` event if file cannot be read

**Usage Examples:**

```javascript
const request = require('superagent');

// Upload single file
request
  .post('http://example.com/upload')
  .attach('document', 'path/to/document.pdf')
  .end((err, res) => {
    if (err) throw err;
    console.log('Upload complete');
  });

// Upload multiple files
request
  .post('http://example.com/upload')
  .attach('photo1', 'images/photo1.jpg')
  .attach('photo2', 'images/photo2.jpg')
  .attach('photo3', 'images/photo3.jpg')
  .end(callback);

// Upload with custom filename
request
  .post('http://example.com/upload')
  .attach('file', 'data.txt', 'custom-name.txt')
  .end(callback);

// Upload Buffer
const buffer = Buffer.from('<h1>Hello World</h1>');
request
  .post('http://example.com/upload')
  .attach('html', buffer, 'index.html')
  .end(callback);

// Upload with ReadStream
const fs = require('fs');
const stream = fs.createReadStream('large-file.zip');
request
  .post('http://example.com/upload')
  .attach('archive', stream, 'backup.zip')
  .end(callback);
```

### Attaching Files (Browser)

In browsers, use `.attach()` with File or Blob objects from user input or generated content.

```javascript { .api }
/**
 * Attach a file for multipart upload (Browser)
 * @param field - Form field name
 * @param file - File or Blob object
 * @param options - Optional filename or options object
 * @returns Request instance for chaining
 * @example
 * // Attach File from input element
 * const fileInput = document.querySelector('input[type="file"]');
 * request.post('/upload')
 *   .attach('photo', fileInput.files[0])
 *   .end(callback);
 *
 * // Attach Blob with filename
 * const blob = new Blob(['content'], { type: 'text/plain' });
 * request.post('/upload')
 *   .attach('file', blob, 'data.txt')
 *   .end(callback);
 */
Request.prototype.attach = function(
  field: string,
  file: File | Blob,
  options?: string | { filename?: string }
): Request;
```

**Browser Implementation Details:**

- Accepts `File` objects from `<input type="file">` elements
- Accepts `Blob` objects for generated content
- Uses native browser `FormData` API
- Filename is extracted from `File.name` property
- Content-Type is detected from `File.type` or `Blob.type`

**Browser Usage Examples:**

```javascript
// Upload from file input
const fileInput = document.querySelector('#fileUpload');
request
  .post('http://example.com/upload')
  .attach('file', fileInput.files[0])
  .end((err, res) => {
    if (err) throw err;
    console.log('File uploaded');
  });

// Upload multiple files from input
const files = document.querySelector('#multipleFiles').files;
const req = request.post('http://example.com/upload');
for (let i = 0; i < files.length; i++) {
  req.attach('files', files[i]);
}
req.end(callback);

// Upload generated Blob
const blob = new Blob(['{"name": "data"}'], { type: 'application/json' });
request
  .post('http://example.com/upload')
  .attach('data', blob, 'data.json')
  .end(callback);

// Upload canvas as image
canvas.toBlob((blob) => {
  request
    .post('http://example.com/upload')
    .attach('image', blob, 'drawing.png')
    .end(callback);
});
```

### Adding Form Fields

Add text form fields to multipart requests alongside file attachments using `.field()`.

```javascript { .api }
/**
 * Add multipart form field(s)
 * @param name - Field name or object of name-value pairs
 * @param value - Field value (string, number, boolean, or array)
 * @returns Request instance for chaining
 * @throws Error if name or value is null/undefined
 * @throws Error if used after .send()
 * @example
 * // Single field
 * request.post('/upload')
 *   .field('name', 'John Doe')
 *   .end(callback);
 *
 * // Multiple fields
 * request.post('/upload')
 *   .field({ name: 'John', age: 30, active: true })
 *   .end(callback);
 *
 * // Array values
 * request.post('/upload')
 *   .field('tags', ['photo', 'vacation'])
 *   .end(callback);
 */
Request.prototype.field = function(
  name: string | object,
  value?: string | number | boolean | Array<any>
): Request;
```

**Implementation Details:**

- Cannot be mixed with `.send()` - must use either `.send()` OR `.field()/.attach()`
- Accepts single name-value pairs: `.field('name', 'value')`
- Accepts objects for multiple fields: `.field({ name: 'value', other: 'data' })`
- Accepts arrays to send multiple values for same field name
- Boolean values are converted to strings ('true'/'false')
- Null or undefined names/values throw errors
- Uses `_getFormData()` internally to initialize FormData instance

**Usage Examples:**

```javascript
// Add form fields with file upload
request
  .post('http://example.com/upload')
  .field('username', 'tobi')
  .field('email', 'tobi@example.com')
  .attach('avatar', 'avatar.jpg')
  .end(callback);

// Multiple fields at once
request
  .post('http://example.com/upload')
  .field({
    title: 'My Document',
    category: 'reports',
    public: true
  })
  .attach('document', 'report.pdf')
  .end(callback);

// Array values for multiple selections
request
  .post('http://example.com/upload')
  .field('tags', ['javascript', 'nodejs', 'http'])
  .field('categories', ['web', 'backend'])
  .attach('file', 'code.js')
  .end(callback);

// Nested field names
request
  .post('http://example.com/upload')
  .field('user[name]', 'John')
  .field('user[email]', 'john@example.com')
  .field('user[age]', 25)
  .attach('file', 'data.txt')
  .end(callback);

// Boolean and number fields
request
  .post('http://example.com/api')
  .field('completed', true)  // converted to 'true'
  .field('priority', 5)
  .field('ratio', 0.75)
  .end(callback);
```

### Multipart Form Data Handling

SuperAgent automatically handles multipart/form-data encoding when you use `.attach()` or `.field()`.

```javascript { .api }
/**
 * Internal FormData management
 * @private
 * @returns FormData instance
 * @description
 * - Creates FormData instance on first .field() or .attach() call
 * - Sets up error handling for FormData stream
 * - Automatically sets Content-Type with boundary
 * - Calculates Content-Length when possible (Node.js)
 */
Request.prototype._getFormData = function(): FormData;

/**
 * Type definition for FormData usage
 * @internal
 */
interface FormDataOptions {
  // Node.js: form-data package
  // Browser: native FormData API
}
```

**Automatic Behavior:**

- Content-Type is automatically set to `multipart/form-data` with boundary
- Content-Length is calculated automatically (Node.js)
- Multipart boundaries are generated automatically
- Form data is properly encoded and streamed
- Files and fields are sent in the order they were added

**Internal Flow:**

1. First `.field()` or `.attach()` call initializes FormData
2. Subsequent calls append to the same FormData instance
3. On `.end()`, FormData is piped to the request
4. Progress events are emitted during upload
5. Response is parsed based on Content-Type header

```javascript
// Example showing order preservation
request
  .post('http://example.com/upload')
  .field('field1', 'first')
  .attach('file1', 'file1.txt')
  .field('field2', 'second')
  .attach('file2', 'file2.txt')
  .end(callback);
// Order in multipart body: field1, file1, field2, file2
```

### Progress Tracking

Monitor upload progress for file attachments and form data submissions.

```javascript { .api }
/**
 * Progress event listener
 * @event progress
 * @param event - Progress event object
 * @property {string} direction - 'upload' or 'download'
 * @property {number} loaded - Bytes transferred
 * @property {number} total - Total bytes (if known)
 * @property {boolean} lengthComputable - Whether total is known
 * @property {number} percent - Upload percentage (browser only)
 * @example
 * request.post('/upload')
 *   .attach('file', 'large.zip')
 *   .on('progress', (event) => {
 *     console.log(`${event.loaded} / ${event.total} bytes`);
 *     if (event.percent) {
 *       console.log(`${event.percent}% complete`);
 *     }
 *   })
 *   .end(callback);
 */
interface ProgressEvent {
  direction: 'upload' | 'download';
  loaded: number;
  total: number;
  lengthComputable: boolean;
  percent?: number;  // Browser only
}
```

**Progress Tracking Details:**

- **Node.js**: Progress events emitted via Transform stream
- **Browser**: Uses `XMLHttpRequest.upload.onprogress`
- Progress events fire during upload with current byte count
- Total size is known for files, may not be available for streams
- Percentage is automatically calculated in browser
- Events continue until upload completes

**Usage Examples:**

```javascript
// Node.js progress tracking
request
  .post('http://example.com/upload')
  .attach('file', 'large-video.mp4')
  .on('progress', (event) => {
    if (event.direction === 'upload') {
      const percentComplete = (event.loaded / event.total) * 100;
      console.log(`Upload: ${percentComplete.toFixed(2)}%`);
      console.log(`${event.loaded} of ${event.total} bytes`);
    }
  })
  .end((err, res) => {
    if (err) throw err;
    console.log('Upload complete');
  });

// Browser progress with UI update
const progressBar = document.querySelector('#uploadProgress');
request
  .post('http://example.com/upload')
  .attach('file', fileInput.files[0])
  .on('progress', (event) => {
    if (event.direction === 'upload' && event.percent) {
      progressBar.style.width = event.percent + '%';
      progressBar.textContent = Math.round(event.percent) + '%';
    }
  })
  .end((err, res) => {
    if (err) {
      progressBar.classList.add('error');
    } else {
      progressBar.classList.add('complete');
    }
  });

// Multiple file upload with total progress
let totalLoaded = 0;
let totalSize = 0;

const files = ['file1.jpg', 'file2.jpg', 'file3.jpg'];
const promises = files.map((file) => {
  return new Promise((resolve, reject) => {
    request
      .post('http://example.com/upload')
      .attach('file', file)
      .on('progress', (event) => {
        if (event.direction === 'upload') {
          totalLoaded += event.loaded;
          totalSize += event.total;
          console.log(`Total: ${(totalLoaded/totalSize*100).toFixed(1)}%`);
        }
      })
      .then(resolve)
      .catch(reject);
  });
});

Promise.all(promises)
  .then(() => console.log('All uploads complete'))
  .catch(err => console.error('Upload failed:', err));
```

### Platform Differences

Key differences between Node.js and browser implementations for file uploads.

**Node.js Features:**

```javascript
// File system access
request
  .post('/upload')
  .attach('file', '/absolute/path/to/file.pdf')  // Works in Node.js
  .end(callback);

// ReadStream support
const fs = require('fs');
const stream = fs.createReadStream('file.dat');
request
  .post('/upload')
  .attach('data', stream)
  .end(callback);

// Buffer support
const buffer = fs.readFileSync('file.bin');
request
  .post('/upload')
  .attach('binary', buffer, 'data.bin')
  .end(callback);

// Uses form-data package
// Content-Length calculated automatically
// Supports all Node.js stream features
```

**Browser Features:**

```javascript
// File input access
const fileInput = document.querySelector('input[type="file"]');
request
  .post('/upload')
  .attach('file', fileInput.files[0])  // File object
  .end(callback);

// Blob support
const blob = new Blob(['content'], { type: 'text/plain' });
request
  .post('/upload')
  .attach('data', blob, 'file.txt')
  .end(callback);

// Canvas/Media API integration
canvas.toBlob((blob) => {
  request.post('/upload').attach('image', blob, 'canvas.png').end(callback);
});

// Uses native FormData API
// Works with File, Blob, and FormData objects
// XMLHttpRequest.upload provides progress events
```

**Compatibility Matrix:**

| Feature | Node.js | Browser |
|---------|---------|---------|
| File paths | ✓ | ✗ |
| File objects | ✗ | ✓ |
| Blob objects | ✗ | ✓ |
| Buffer objects | ✓ | ✗ |
| ReadStream | ✓ | ✗ |
| FormData objects | ✓ | ✓ |
| Progress events | ✓ | ✓ |
| Content-Length auto | ✓ | ✗ |

### Error Handling

Handle file upload errors and validation failures.

```javascript { .api }
/**
 * Error handling for file uploads
 * @event error
 * @param error - Error object
 * @property {string} code - Error code (ENOENT, EACCES, etc.)
 * @property {string} path - File path that caused error (Node.js)
 * @property {string} message - Error description
 * @example
 * request.post('/upload')
 *   .attach('file', 'nonexistent.txt')
 *   .on('error', (err) => {
 *     console.error('Upload error:', err.message);
 *     if (err.code === 'ENOENT') {
 *       console.error('File not found:', err.path);
 *     }
 *   })
 *   .end((err, res) => {
 *     if (err) {
 *       // Error already handled in listener
 *       return;
 *     }
 *   });
 */
interface UploadError extends Error {
  code?: string;
  path?: string;
  status?: number;
}
```

**Common Error Scenarios:**

```javascript
// File not found (Node.js)
request
  .post('/upload')
  .attach('file', 'missing.txt')
  .on('error', (err) => {
    // err.code === 'ENOENT'
    // err.path === 'missing.txt'
    console.error('File not found');
  })
  .end(callback);

// Permission denied (Node.js)
request
  .post('/upload')
  .attach('file', '/root/secret.txt')
  .on('error', (err) => {
    // err.code === 'EACCES'
    console.error('Permission denied');
  })
  .end(callback);

// Mixed .send() and .attach() error
try {
  request
    .post('/upload')
    .send({ data: 'json' })
    .attach('file', 'file.txt')  // Throws Error
    .end(callback);
} catch (err) {
  console.error(err.message);
  // "superagent can't mix .send() and .attach()"
}

// Mixed .send() and .field() error
try {
  request
    .post('/upload')
    .send({ data: 'json' })
    .field('name', 'value')  // Throws Error
    .end(callback);
} catch (err) {
  console.error(err.message);
  // ".field() can't be used if .send() is used..."
}

// Network errors during upload
request
  .post('http://unreachable.example.com/upload')
  .attach('file', 'large.zip')
  .on('error', (err) => {
    if (err.code === 'ECONNREFUSED') {
      console.error('Server not available');
    } else if (err.code === 'ETIMEDOUT') {
      console.error('Upload timed out');
    }
  })
  .end(callback);

// Server-side validation errors
request
  .post('/upload')
  .attach('file', 'document.pdf')
  .end((err, res) => {
    if (err) {
      if (res && res.status === 413) {
        console.error('File too large');
      } else if (res && res.status === 415) {
        console.error('Unsupported file type');
      } else if (res && res.status === 400) {
        console.error('Validation error:', res.body.message);
      }
    }
  });
```

### Complete Upload Examples

Real-world examples combining multiple features.

**Node.js: Multi-file Upload with Metadata**

```javascript
const request = require('superagent');
const fs = require('fs');

async function uploadDocuments(userId, documents) {
  try {
    const res = await request
      .post('http://api.example.com/documents')
      .field('userId', userId)
      .field('uploadDate', new Date().toISOString())
      .field('category', 'reports')
      .field('tags', ['annual', 'financial', '2024'])
      .attach('mainReport', documents.main)
      .attach('appendixA', documents.appendixA)
      .attach('appendixB', documents.appendixB)
      .on('progress', (event) => {
        if (event.direction === 'upload') {
          const percent = (event.loaded / event.total * 100).toFixed(1);
          console.log(`Upload progress: ${percent}%`);
        }
      });

    console.log('Upload complete:', res.body);
    return res.body;
  } catch (err) {
    if (err.status === 413) {
      throw new Error('Files too large');
    } else if (err.status === 401) {
      throw new Error('Authentication required');
    }
    throw err;
  }
}

// Usage
uploadDocuments('user-123', {
  main: 'reports/annual-report-2024.pdf',
  appendixA: 'reports/financial-data.xlsx',
  appendixB: 'reports/charts.pdf'
});
```

**Browser: Image Upload with Preview**

```javascript
function setupImageUpload() {
  const fileInput = document.querySelector('#imageUpload');
  const previewImg = document.querySelector('#preview');
  const progressBar = document.querySelector('#progress');
  const uploadBtn = document.querySelector('#uploadButton');

  let selectedFile = null;

  fileInput.addEventListener('change', (e) => {
    selectedFile = e.target.files[0];

    // Show preview
    const reader = new FileReader();
    reader.onload = (e) => {
      previewImg.src = e.target.result;
    };
    reader.readAsDataURL(selectedFile);

    uploadBtn.disabled = false;
  });

  uploadBtn.addEventListener('click', async () => {
    if (!selectedFile) return;

    try {
      const res = await request
        .post('/api/images')
        .field('title', document.querySelector('#title').value)
        .field('description', document.querySelector('#description').value)
        .field('tags', document.querySelector('#tags').value.split(','))
        .attach('image', selectedFile)
        .on('progress', (event) => {
          if (event.direction === 'upload' && event.percent) {
            progressBar.style.width = event.percent + '%';
            progressBar.textContent = Math.round(event.percent) + '%';
          }
        });

      alert('Upload successful! Image ID: ' + res.body.id);
      previewImg.src = res.body.url;
    } catch (err) {
      alert('Upload failed: ' + err.message);
    }
  });
}

setupImageUpload();
```

**Node.js: Streaming Large File Upload**

```javascript
const request = require('superagent');
const fs = require('fs');
const path = require('path');

async function uploadLargeFile(filePath, options = {}) {
  const fileName = path.basename(filePath);
  const stats = fs.statSync(filePath);
  const fileSize = stats.size;

  console.log(`Uploading ${fileName} (${(fileSize / 1024 / 1024).toFixed(2)} MB)`);

  let lastProgress = 0;

  try {
    const res = await request
      .post('http://api.example.com/files')
      .field('filename', fileName)
      .field('filesize', fileSize)
      .field('checksum', options.checksum || '')
      .attach('file', fs.createReadStream(filePath), fileName)
      .timeout({ deadline: 600000 })  // 10 minute deadline
      .on('progress', (event) => {
        if (event.direction === 'upload') {
          const currentProgress = Math.floor(event.loaded / event.total * 100);
          if (currentProgress > lastProgress) {
            lastProgress = currentProgress;
            const uploadedMB = (event.loaded / 1024 / 1024).toFixed(2);
            const totalMB = (event.total / 1024 / 1024).toFixed(2);
            console.log(`Progress: ${currentProgress}% (${uploadedMB}/${totalMB} MB)`);
          }
        }
      });

    console.log('Upload complete:', res.body);
    return res.body;
  } catch (err) {
    if (err.timeout) {
      throw new Error('Upload timed out');
    }
    throw err;
  }
}

// Usage
uploadLargeFile('/path/to/large-video.mp4', {
  checksum: 'sha256-hash-here'
});
```

**Browser: Multiple Files with Drag and Drop**

```javascript
function setupDragAndDrop() {
  const dropZone = document.querySelector('#dropZone');
  const fileList = document.querySelector('#fileList');
  const uploadBtn = document.querySelector('#uploadAll');

  let filesToUpload = [];

  dropZone.addEventListener('dragover', (e) => {
    e.preventDefault();
    dropZone.classList.add('drag-over');
  });

  dropZone.addEventListener('dragleave', () => {
    dropZone.classList.remove('drag-over');
  });

  dropZone.addEventListener('drop', (e) => {
    e.preventDefault();
    dropZone.classList.remove('drag-over');

    const files = Array.from(e.dataTransfer.files);
    filesToUpload = files;

    // Update UI
    fileList.innerHTML = files.map(f =>
      `<li>${f.name} (${(f.size / 1024).toFixed(1)} KB)</li>`
    ).join('');

    uploadBtn.disabled = false;
  });

  uploadBtn.addEventListener('click', async () => {
    const results = [];

    for (const file of filesToUpload) {
      try {
        const res = await request
          .post('/api/upload')
          .field('category', 'documents')
          .attach('file', file)
          .on('progress', (event) => {
            if (event.direction === 'upload' && event.percent) {
              console.log(`${file.name}: ${event.percent.toFixed(1)}%`);
            }
          });

        results.push({ file: file.name, success: true, id: res.body.id });
      } catch (err) {
        results.push({ file: file.name, success: false, error: err.message });
      }
    }

    // Show results
    const successful = results.filter(r => r.success).length;
    alert(`Uploaded ${successful} of ${results.length} files`);
  });
}

setupDragAndDrop();
```

## Type Definitions

Complete TypeScript type definitions for file upload functionality.

```typescript { .api }
interface Request {
  /**
   * Attach file for multipart upload
   */
  attach(
    field: string,
    file: string | Buffer | ReadStream | File | Blob,
    options?: string | AttachOptions
  ): Request;

  /**
   * Add multipart form field
   */
  field(name: string, value: string | number | boolean): Request;
  field(name: string, value: Array<string | number | boolean>): Request;
  field(fields: Record<string, any>): Request;

  /**
   * Progress event listener
   */
  on(event: 'progress', listener: (event: ProgressEvent) => void): Request;
  on(event: 'error', listener: (error: Error) => void): Request;
}

interface AttachOptions {
  filename?: string;
  contentType?: string;
}

interface ProgressEvent {
  direction: 'upload' | 'download';
  lengthComputable: boolean;
  loaded: number;
  total: number;
  percent?: number;
}

interface Response {
  files?: Record<string, UploadedFile>;
}

interface UploadedFile {
  name: string;
  type: string;
  path: string;
  size?: number;
}
```

## Best Practices

**File Upload Guidelines:**

1. Always validate file types and sizes on the server
2. Use progress events for files larger than 1MB
3. Set appropriate timeouts for large uploads
4. Handle errors gracefully with user feedback
5. Consider chunked uploads for very large files
6. Use descriptive field names for better debugging
7. Compress files before upload when possible
8. Implement retry logic for failed uploads

**Security Considerations:**

1. Never trust client-provided filenames
2. Sanitize uploaded file paths on server
3. Scan uploaded files for malware
4. Implement file size limits
5. Validate file MIME types on server
6. Use secure file storage locations
7. Implement access controls for uploaded files
8. Consider encryption for sensitive files

**Performance Tips:**

1. Stream large files instead of loading into memory
2. Use progress events to show upload status
3. Implement proper timeout handling
4. Consider parallel uploads for multiple files
5. Use compression when appropriate
6. Clean up temporary files in Node.js
7. Optimize FormData boundary size
8. Monitor memory usage for large uploads
