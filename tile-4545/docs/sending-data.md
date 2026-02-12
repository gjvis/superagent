# Sending Request Data

Send request bodies in various formats including JSON, form data, and multipart uploads.

## Capabilities

### Sending Request Body

Send data in the request body, automatically serialized based on Content-Type.

```javascript { .api }
/**
 * Send request body data
 * @param data - Data to send (object, string, Buffer, etc.)
 * @returns Request instance for chaining
 */
send(data: any): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Send JSON (default for objects)
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' });

// Send plain text
request
  .post('/api/data')
  .type('text/plain')
  .send('plain text data');

// Send Buffer (Node.js)
const buffer = Buffer.from('binary data');
request
  .post('/api/upload')
  .type('application/octet-stream')
  .send(buffer);

// Multiple sends are merged for objects
request
  .post('/api/users')
  .send({ name: 'John' })
  .send({ email: 'john@example.com' })
  .send({ role: 'admin' });
// Sends: { name: 'John', email: 'john@example.com', role: 'admin' }

// For strings, multiple sends concatenate
request
  .post('/api/data')
  .type('text')
  .send('Hello ')
  .send('World');
// Sends: 'Hello World'
```

### Form Fields

Add individual form fields for multipart/form-data requests.

```javascript { .api }
/**
 * Set form field
 * @param name - Field name
 * @param value - Field value
 * @returns Request instance for chaining
 */
field(name: string, value: string | number): Request;

/**
 * Set multiple form fields at once
 * @param fields - Object of field names and values
 * @returns Request instance for chaining
 */
field(fields: object): Request;
```

**Usage Examples:**

```javascript
// Create multipart form
request
  .post('/api/profile')
  .field('name', 'John Doe')
  .field('email', 'john@example.com')
  .field('age', 30);

// Set multiple fields at once
request
  .post('/api/profile')
  .field({
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
  });

// Combined with file upload
request
  .post('/api/profile')
  .field('name', 'John Doe')
  .attach('avatar', '/path/to/image.jpg');
```

### File Attachments

Attach files for multipart/form-data uploads.

```javascript { .api }
/**
 * Attach file to request
 * @param field - Form field name
 * @param file - File path (Node.js) or Blob/File object (Browser)
 * @param options - Optional filename and contentType
 * @returns Request instance for chaining
 */
attach(
  field: string,
  file: string | Buffer | Blob | File,
  options?: {filename?: string, contentType?: string}
): Request;

// Node.js also accepts filename as second parameter
attach(field: string, file: string | Buffer, filename?: string): Request;
```

**Usage Examples (Node.js):**

```javascript
// Attach file by path
request
  .post('/api/upload')
  .attach('document', '/path/to/file.pdf');

// Attach Buffer with filename
const buffer = fs.readFileSync('/path/to/file.pdf');
request
  .post('/api/upload')
  .attach('document', buffer, 'file.pdf');

// Attach with options
request
  .post('/api/upload')
  .attach('document', '/path/to/file.pdf', {
    filename: 'custom-name.pdf',
    contentType: 'application/pdf'
  });

// Multiple file attachments
request
  .post('/api/upload')
  .attach('document1', '/path/to/file1.pdf')
  .attach('document2', '/path/to/file2.pdf')
  .attach('image', '/path/to/photo.jpg');

// Combined with form fields
request
  .post('/api/upload')
  .field('description', 'My files')
  .attach('file1', '/path/to/file1.pdf')
  .attach('file2', '/path/to/file2.pdf');
```

**Usage Examples (Browser):**

```javascript
// Attach File object from input
const fileInput = document.querySelector('input[type="file"]');
const file = fileInput.files[0];

request
  .post('/api/upload')
  .attach('document', file);

// Attach Blob
const blob = new Blob(['file contents'], {type: 'text/plain'});
request
  .post('/api/upload')
  .attach('document', blob, 'file.txt');

// Attach with custom filename
request
  .post('/api/upload')
  .attach('document', file, {
    filename: 'custom-name.pdf'
  });
```

### Query Parameters

Add query string parameters to the request URL.

```javascript { .api }
/**
 * Add query string parameters
 * @param params - Query parameters as object or string
 * @returns Request instance for chaining
 */
query(params: object | string): Request;
```

**Usage Examples:**

```javascript
// Single query object
request
  .get('/api/users')
  .query({ role: 'admin', active: true, limit: 10 });
// URL: /api/users?role=admin&active=true&limit=10

// Multiple query calls (merged)
request
  .get('/api/users')
  .query({ role: 'admin' })
  .query({ active: true });

// Query with arrays
request
  .get('/api/users')
  .query({ ids: [1, 2, 3] });
// URL: /api/users?ids[0]=1&ids[1]=2&ids[2]=3

// Query with nested objects
request
  .get('/api/search')
  .query({
    filter: {
      name: 'John',
      age: 30
    }
  });
// URL: /api/search?filter[name]=John&filter[age]=30

// Query string format
request
  .get('/api/users')
  .query('role=admin&active=true');
```

## Content-Type Auto-Detection

SuperAgent automatically sets Content-Type based on the data being sent:

```javascript
// Sends as application/json
request
  .post('/api/users')
  .send({ name: 'John' });

// Use .type() to override
request
  .post('/api/data')
  .type('application/xml')
  .send('<user><name>John</name></user>');

// Form data (application/x-www-form-urlencoded)
request
  .post('/api/login')
  .type('form')
  .send({ username: 'admin', password: 'secret' });

// Multipart form data (when using .field() or .attach())
request
  .post('/api/upload')
  .field('name', 'John')
  .attach('file', '/path/to/file.pdf');
// Automatically sets Content-Type: multipart/form-data
```
