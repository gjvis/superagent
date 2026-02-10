# File Uploads

This document covers file upload functionality using multipart/form-data in Node.js. SuperAgent supports uploading files from file paths, Buffers, and Streams with automatic content-type detection.

**Note:** File upload features (`.attach()` and `.field()`) are only available in Node.js, not in browsers.

## Capabilities

### Attaching Files

Attach files to the request using multipart/form-data encoding.

```javascript { .api }
/**
 * Queue file as attachment to specified field
 * @param field - Form field name
 * @param file - File path (string), Buffer, or Stream
 * @param options - Filename string or options object
 * @returns Request instance for chaining
 */
attach(field: string, file: string | Buffer | Stream, options?: string | { filename?: string }): Request;
```

**File parameter types:**
- **String**: Path to file on disk (automatically creates ReadStream)
- **Buffer**: File contents as Buffer
- **Stream**: Readable stream (e.g., `fs.createReadStream()`)

**Options:**
- **String**: Used as filename
- **Object with `filename`**: Specify custom filename for the attachment

**Automatic filename detection:**
- For file paths: Uses the path as filename
- For streams with `.path` property: Uses stream.path as filename
- For Buffers: Requires explicit filename in options

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Attach file by path (filename auto-detected)
request
  .post('/api/upload')
  .attach('avatar', '/path/to/image.jpg')
  .then(res => console.log('Uploaded:', res.body));

// Attach file with custom filename
request
  .post('/api/upload')
  .attach('avatar', '/path/to/photo.jpg', 'profile-pic.jpg');

// Attach Buffer with filename
const buffer = Buffer.from('file contents');
request
  .post('/api/upload')
  .attach('document', buffer, 'readme.txt');

// Attach Buffer with options object
request
  .post('/api/upload')
  .attach('file', buffer, { filename: 'data.bin' });

// Attach stream
const stream = fs.createReadStream('/path/to/file.pdf');
request
  .post('/api/upload')
  .attach('document', stream);

// Attach multiple files to different fields
request
  .post('/api/upload')
  .attach('avatar', '/path/to/avatar.jpg')
  .attach('cover', '/path/to/cover.png')
  .attach('document', '/path/to/resume.pdf');

// Attach multiple files to same field
request
  .post('/api/upload')
  .attach('images', '/path/to/image1.jpg')
  .attach('images', '/path/to/image2.jpg')
  .attach('images', '/path/to/image3.jpg');
```

### Adding Form Fields

Add regular form fields to multipart/form-data requests alongside file attachments.

```javascript { .api }
/**
 * Add form field for multipart/form-data request
 * @param name - Field name
 * @param value - Field value (string, Blob, File, Buffer, or fs.ReadStream)
 * @returns Request instance for chaining
 */
field(name: string, value: string | Blob | File | Buffer): Request;

/**
 * Add multiple form fields from object
 * @param fields - Object with field name/value pairs
 * @returns Request instance for chaining
 */
field(fields: object): Request;
```

**Value types:**
- **String**: Regular form field value
- **Boolean**: Converted to string ('true' or 'false')
- **Buffer**: Binary data
- **Arrays**: Multiple values for same field name (calls `.field()` for each)

**Usage Examples:**

```javascript
// Add single field
request
  .post('/api/upload')
  .field('name', 'Alice')
  .attach('avatar', '/path/to/image.jpg');

// Add multiple fields individually
request
  .post('/api/upload')
  .field('name', 'Alice')
  .field('email', 'alice@example.com')
  .field('age', '30')
  .attach('avatar', '/path/to/image.jpg');

// Add multiple fields from object
request
  .post('/api/upload')
  .field({
    name: 'Alice',
    email: 'alice@example.com',
    bio: 'Software developer'
  })
  .attach('avatar', '/path/to/image.jpg');

// Add array values (multiple fields with same name)
request
  .post('/api/upload')
  .field('tags', ['javascript', 'node', 'http']);
// Results in: tags=javascript&tags=node&tags=http

// Boolean values
request
  .post('/api/upload')
  .field('active', true)
  .attach('document', '/path/to/file.pdf');
// Results in: active=true
```

### Combining Fields and Files

Build complex multipart/form-data requests with multiple fields and files.

**Usage Example:**

```javascript
const request = require('superagent');

// Upload user profile with avatar
request
  .post('/api/users')
  .field('name', 'Alice Johnson')
  .field('email', 'alice@example.com')
  .field('age', '30')
  .field('bio', 'Software developer and open source contributor')
  .attach('avatar', '/path/to/avatar.jpg')
  .then(res => {
    console.log('User created:', res.body);
  })
  .catch(err => {
    console.error('Upload failed:', err);
  });

// Upload blog post with images
request
  .post('/api/posts')
  .field({
    title: 'My Blog Post',
    content: 'Post content here...',
    tags: ['javascript', 'tutorial'],
    published: true
  })
  .attach('thumbnail', '/path/to/thumbnail.jpg')
  .attach('images', '/path/to/image1.jpg')
  .attach('images', '/path/to/image2.jpg')
  .then(res => {
    console.log('Post created:', res.body);
  });
```

### Important Constraints

**Cannot mix `.send()` with `.attach()` or `.field()`:**

```javascript
// ❌ This will throw an error
request
  .post('/api/upload')
  .send({ name: 'Alice' })  // Uses .send()
  .attach('file', '/path/to/file.pdf');  // Cannot use .attach() after .send()
// Error: .send() can't be used if .attach() or .field() is used

// ✅ Correct: Use .field() instead
request
  .post('/api/upload')
  .field('name', 'Alice')
  .attach('file', '/path/to/file.pdf');
```

**Cannot mix `.attach()` or `.field()` with `.send()`:**

```javascript
// ❌ This will throw an error
request
  .post('/api/upload')
  .attach('file', '/path/to/file.pdf')
  .send({ name: 'Alice' });  // Cannot use .send() after .attach()
// Error: .field() can't be used if .send() is used

// ✅ Correct: Use .field() for all data
request
  .post('/api/upload')
  .attach('file', '/path/to/file.pdf')
  .field('name', 'Alice');
```

### Headers and Content-Type

When using `.attach()` or `.field()`, SuperAgent automatically:
- Sets `Content-Type` to `multipart/form-data` with boundary
- Calculates `Content-Length` header if possible
- Handles FormData stream piping internally

**You should not manually set Content-Type** when using file uploads.

### Error Handling

```javascript
request
  .post('/api/upload')
  .attach('file', '/path/to/nonexistent.pdf')
  .end((err, res) => {
    if (err) {
      // File not found or upload error
      console.error('Upload error:', err.message);
      console.error('Error code:', err.code);  // e.g., 'ENOENT' for file not found
    } else {
      console.log('Upload successful:', res.body);
    }
  });
```

### Complete Upload Example

```javascript
const request = require('superagent');
const fs = require('fs');

// Complete file upload with metadata
request
  .post('https://api.example.com/upload')
  .set('Authorization', 'Bearer token123')
  .field('title', 'Annual Report 2025')
  .field('description', 'Company annual report document')
  .field('category', 'reports')
  .field('tags', ['finance', 'annual', '2025'])
  .field('public', false)
  .attach('document', '/path/to/report.pdf')
  .attach('thumbnail', '/path/to/thumbnail.png', 'preview.png')
  .on('progress', event => {
    console.log('Upload progress:', event.percent + '%');
  })
  .then(res => {
    console.log('Upload successful!');
    console.log('File ID:', res.body.id);
    console.log('File URL:', res.body.url);
  })
  .catch(err => {
    if (err.status === 413) {
      console.error('File too large');
    } else if (err.code === 'ENOENT') {
      console.error('File not found');
    } else {
      console.error('Upload failed:', err.message);
    }
  });
```

### Progress Events (Node.js)

Monitor upload progress using the `'progress'` event:

```javascript
request
  .post('/api/upload')
  .attach('file', '/path/to/large-file.zip')
  .on('progress', event => {
    console.log('Direction:', event.direction);  // 'upload'
    console.log('Loaded:', event.loaded);        // Bytes uploaded
    console.log('Total:', event.total);          // Total bytes
    console.log('Percent:', Math.round(event.loaded / event.total * 100) + '%');
  })
  .then(res => {
    console.log('Upload complete');
  });
```
