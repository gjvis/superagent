# Request Methods

Request configuration methods for building HTTP requests with headers, body data, query parameters, content types, and authentication.

## Capabilities

### Header Management

Get, set, and remove request headers with case-insensitive field names.

```javascript { .api }
/**
 * Get request header value (case-insensitive)
 * @param field - Header field name
 * @returns Header value
 */
request.get(url).get(field: string): string;

/**
 * Set request header(s)
 * @param field - Header field name or object with multiple headers
 * @param value - Header value (if field is string)
 * @returns Request for chaining
 */
request.get(url).set(field: string, value: string): Request;
request.get(url).set(headers: object): Request;

/**
 * Remove request header (case-insensitive)
 * @param field - Header field name
 * @returns Request for chaining
 */
request.get(url).unset(field: string): Request;
```

**Usage Examples:**

```javascript
// Set single header
request
  .get('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('Accept', 'application/json');

// Set multiple headers
request
  .get('/api/users')
  .set({
    'Authorization': 'Bearer token123',
    'Accept': 'application/json',
    'X-API-Key': 'key456'
  });

// Get header value
const auth = request.get('/api/users').get('Authorization');

// Remove header
request
  .get('/api/users')
  .set('X-Custom', 'value')
  .unset('X-Custom');
```

### Request Body

Send request body data with automatic content-type detection.

```javascript { .api }
/**
 * Send request body
 * Auto-detects content type: objects become JSON, strings become form-urlencoded
 * Multiple .send() calls merge objects or concatenate strings
 * @param data - Request body (object, string, or Buffer)
 * @returns Request for chaining
 */
request.post(url).send(data: any): Request;
```

**Usage Examples:**

```javascript
// Send JSON (automatic)
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' });
// Sets Content-Type: application/json

// Send form data (automatic)
request
  .post('/api/users')
  .send('name=John&email=john@example.com');
// Sets Content-Type: application/x-www-form-urlencoded

// Merge multiple sends
request
  .post('/api/users')
  .send({ name: 'John' })
  .send({ email: 'john@example.com' });
// Results in: {name: 'John', email: 'john@example.com'}

// Force content type
request
  .post('/api/users')
  .type('json')
  .send('{"name":"John"}');
```

### Multipart Form Data

Send multipart form data with fields and file attachments (Node.js only, browser uses FormData).

```javascript { .api }
/**
 * Add multipart form field(s)
 * Cannot be mixed with .send()
 * @param name - Field name or object with multiple fields
 * @param value - Field value (if name is string), can be string or boolean
 * @returns Request for chaining
 */
request.post(url).field(name: string, value: string | boolean): Request;
request.post(url).field(fields: object): Request;

/**
 * Attach file for multipart upload (Node.js only)
 * Cannot be mixed with .send()
 * @param field - Form field name
 * @param file - File path (string), Buffer, or Stream
 * @param options - Optional {filename: string, contentType: string}
 * @returns Request for chaining
 */
request.post(url).attach(
  field: string,
  file: string | Buffer | Stream,
  options?: {filename?: string, contentType?: string}
): Request;
```

**Usage Examples:**

```javascript
// Upload single file with fields
request
  .post('/upload')
  .field('title', 'My Photo')
  .field('description', 'Vacation picture')
  .attach('photo', '/path/to/image.jpg');

// Upload multiple files
request
  .post('/upload')
  .attach('photos', 'image1.jpg')
  .attach('photos', 'image2.jpg')
  .attach('photos', 'image3.jpg');

// Upload buffer with filename
const imageBuffer = fs.readFileSync('image.png');
request
  .post('/upload')
  .attach('photo', imageBuffer, { filename: 'photo.png', contentType: 'image/png' });

// Multiple fields at once
request
  .post('/upload')
  .field({
    title: 'Document',
    category: 'legal',
    private: true
  })
  .attach('document', 'contract.pdf');

// Array values create multiple fields
request
  .post('/upload')
  .field('tags', ['photo', 'vacation', '2024']);
// Creates: tags=photo, tags=vacation, tags=2024
```

### Query Parameters

Add query string parameters to URL.

```javascript { .api }
/**
 * Add query string parameter(s)
 * Multiple calls append to query string
 * @param params - Query string or object with parameters
 * @returns Request for chaining
 */
request.get(url).query(params: string | object): Request;

/**
 * Sort query string parameters
 * @param compareFn - Optional comparison function for sorting
 * @returns Request for chaining
 */
request.get(url).sortQuery(compareFn?: (a: string, b: string) => number): Request;
```

**Usage Examples:**

```javascript
// Add query parameters
request
  .get('/api/users')
  .query({ page: 1, limit: 10 });
// Results in: /api/users?page=1&limit=10

// String format
request
  .get('/api/users')
  .query('page=1&limit=10');

// Multiple calls append
request
  .get('/api/users')
  .query({ page: 1 })
  .query({ limit: 10 })
  .query({ sort: 'name' });
// Results in: /api/users?page=1&limit=10&sort=name

// Arrays in query
request
  .get('/api/users')
  .query({ tags: ['admin', 'active'] });
// Results in: /api/users?tags=admin&tags=active

// Sort query string
request
  .get('/api/users')
  .query('z=1&a=2&m=3')
  .sortQuery();
// Results in: /api/users?a=2&m=3&z=1

// Custom sort function
request
  .get('/api/users')
  .query('name=John&age=30&city=NYC')
  .sortQuery((a, b) => a.length - b.length);
// Sorts by parameter name length
```

### Content Type

Set Content-Type header with shortcuts.

```javascript { .api }
/**
 * Set Content-Type header
 * Supports MIME types and shortcuts
 * @param type - MIME type or shortcut ('json', 'xml', 'form', 'html', 'text')
 * @returns Request for chaining
 */
request.post(url).type(type: string): Request;
```

**Usage Examples:**

```javascript
// Shorthand types
request.post('/api/users').type('json');
// Sets: Content-Type: application/json

request.post('/api/users').type('form');
// Sets: Content-Type: application/x-www-form-urlencoded

request.post('/api/users').type('xml');
// Sets: Content-Type: application/xml

// Full MIME type
request.post('/api/users').type('application/vnd.api+json');

// Available shortcuts:
// - 'json' → 'application/json'
// - 'form' → 'application/x-www-form-urlencoded'
// - 'xml' → 'application/xml'
// - 'html' → 'text/html'
// - 'text' → 'text/plain'
```

### Accept Header

Set Accept header for response content negotiation.

```javascript { .api }
/**
 * Set Accept header
 * Supports MIME types and shortcuts
 * @param type - MIME type or shortcut ('json', 'xml', 'form', 'html', 'text')
 * @returns Request for chaining
 */
request.get(url).accept(type: string): Request;
```

**Usage Examples:**

```javascript
// Shorthand types
request.get('/api/users').accept('json');
// Sets: Accept: application/json

request.get('/api/data').accept('xml');
// Sets: Accept: application/xml

// Full MIME type
request.get('/api/data').accept('application/vnd.api+json');

// Multiple types (use full header)
request
  .get('/api/data')
  .set('Accept', 'application/json, text/plain, */*');
```

### Authentication

Set authentication credentials with Basic, Bearer, or auto modes.

```javascript { .api }
/**
 * Set authentication
 * @param user - Username (for basic/auto) or token (for bearer)
 * @param pass - Password (optional for bearer, required for basic)
 * @param options - {type: 'basic'|'bearer'|'auto'} (default: 'basic')
 * @returns Request for chaining
 */
request.get(url).auth(user: string, pass?: string, options?: {type: string}): Request;
```

**Usage Examples:**

```javascript
// Basic authentication (default)
request
  .get('/api/users')
  .auth('username', 'password');
// Sets: Authorization: Basic base64(username:password)

// Bearer token
request
  .get('/api/users')
  .auth('token123', { type: 'bearer' });
// Sets: Authorization: Bearer token123

// Auto mode (for redirects and 401 responses)
request
  .get('/api/users')
  .auth('username', 'password', { type: 'auto' });
// Sends credentials only when challenged

// Shorthand (user:pass format)
request
  .get('/api/users')
  .auth('username:password');
```

### CORS Credentials

Enable transmission of cookies with cross-domain requests (browser only).

```javascript { .api }
/**
 * Enable CORS credentials
 * Browser only - allows cookies with cross-origin requests
 * @param on - Enable flag (default: true)
 * @returns Request for chaining
 */
request.get(url).withCredentials(on?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable credentials
request
  .get('https://api.example.com/data')
  .withCredentials()
  .then(res => console.log(res.body));

// Explicit enable/disable
request
  .get('https://api.example.com/data')
  .withCredentials(true);

// Note: Server must set appropriate CORS headers:
// - Access-Control-Allow-Credentials: true
// - Access-Control-Allow-Origin: (specific origin, not *)
```

### Custom Parsers and Serializers

Override default response parsing and request serialization.

```javascript { .api }
/**
 * Set custom response body parser
 * @param parser - Function to parse response
 * @returns Request for chaining
 */
request.get(url).parse(parser: (res: Response, callback: (err: Error, body: any) => void) => void): Request;

/**
 * Set custom request body serializer
 * @param serializer - Function to serialize request data
 * @returns Request for chaining
 */
request.post(url).serialize(serializer: (obj: any) => string): Request;
```

**Usage Examples:**

```javascript
// Custom XML parser
request
  .get('/api/data.xml')
  .parse((res, callback) => {
    const xml = parseXML(res.text);
    callback(null, xml);
  })
  .then(res => console.log(res.body));

// Custom serializer
request
  .post('/api/data')
  .serialize(obj => convertToXML(obj))
  .send({ data: 'value' });

// CSV parser example
request
  .get('/api/export.csv')
  .parse((res, callback) => {
    const rows = res.text.split('\n').map(line => line.split(','));
    callback(null, rows);
  });
```

### Response Type

Set XHR responseType for binary data (browser only).

```javascript { .api }
/**
 * Set response type for binary data (browser only)
 * In Node.js, all responses are Buffers when appropriate
 * @param type - 'blob' or 'arraybuffer'
 * @returns Request for chaining
 */
request.get(url).responseType(type: 'blob' | 'arraybuffer'): Request;
```

**Usage Examples:**

```javascript
// Get blob (browser)
request
  .get('/api/download/image.png')
  .responseType('blob')
  .then(res => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    // Use blob URL
  });

// Get ArrayBuffer (browser)
request
  .get('/api/download/data.bin')
  .responseType('arraybuffer')
  .then(res => {
    const buffer = res.body;
    const view = new DataView(buffer);
    // Process binary data
  });
```

### Redirects

Set maximum number of redirects to follow (Node.js only).

```javascript { .api }
/**
 * Set maximum number of redirects (Node.js only)
 * Browser follows redirects automatically
 * @param count - Max redirects (default: 5 for GET/HEAD, 0 for others)
 * @returns Request for chaining
 */
request.get(url).redirects(count: number): Request;
```

**Usage Examples:**

```javascript
// Allow up to 10 redirects
request
  .get('/api/resource')
  .redirects(10);

// Disable redirects
request
  .get('/api/resource')
  .redirects(0);

// Default behavior:
// - GET/HEAD: 5 redirects
// - POST/PUT/etc: 0 redirects
```

### Response Size Limit

Set maximum response body size in bytes.

```javascript { .api }
/**
 * Set maximum response body size
 * @param bytes - Max size in bytes (default: 200MB)
 * @returns Request for chaining
 */
request.get(url).maxResponseSize(bytes: number): Request;
```

**Usage Examples:**

```javascript
// Limit to 5MB
request
  .get('/api/large-data')
  .maxResponseSize(5 * 1024 * 1024);

// Limit to 1MB
request
  .get('/api/data')
  .maxResponseSize(1048576);
```

### OK Status Check

Define custom "ok" response validation.

```javascript { .api }
/**
 * Define custom "ok" status check
 * By default, 2xx status codes are "ok"
 * @param callback - Function returning true for acceptable responses
 * @returns Request for chaining
 */
request.get(url).ok(callback: (res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Accept 404 as ok
request
  .get('/api/optional-resource')
  .ok(res => res.status === 404 || res.ok);

// Accept all status codes
request
  .get('/api/resource')
  .ok(res => true);

// Accept 200 and 304 only
request
  .get('/api/resource')
  .ok(res => res.status === 200 || res.status === 304);
```

### Request Conversion

Convert request to plain JavaScript object.

```javascript { .api }
/**
 * Convert request to plain object
 * Returns non-chainable object with request details
 * @returns Object with method, url, data, headers
 */
request.post(url).toJSON(): {method: string, url: string, data: any, headers: object};
```

**Usage Examples:**

```javascript
const req = request
  .post('/api/users')
  .send({ name: 'John' })
  .set('Authorization', 'Bearer token');

const reqObj = req.toJSON();
console.log(reqObj);
// {
//   method: 'POST',
//   url: '/api/users',
//   data: { name: 'John' },
//   headers: { authorization: 'Bearer token' }
// }
```

### Buffer Control (Node.js)

Enable or disable response buffering for streaming.

```javascript { .api }
/**
 * Enable/disable response buffering (Node.js only)
 * Must be false for streaming with .pipe()
 * @param enable - Enable buffering (default: true)
 * @returns Request for chaining
 */
request.get(url).buffer(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Disable buffering for streaming
request
  .get('/api/large-file')
  .buffer(false)
  .pipe(fs.createWriteStream('output.dat'));

// Explicit enable (default)
request
  .get('/api/data')
  .buffer(true)
  .then(res => console.log(res.body));

// Global buffer configuration
request.buffer['application/pdf'] = true;  // Always buffer PDFs
request.buffer['video/mp4'] = false;       // Never buffer videos
```

### HTTP Agent (Node.js)

Set HTTP agent for connection pooling and keep-alive.

```javascript { .api }
/**
 * Set HTTP agent for connection pooling (Node.js only)
 * @param agent - http.Agent instance or false to disable pooling
 * @returns Request for chaining
 */
request.get(url).agent(agent: http.Agent | false): Request;
```

**Usage Examples:**

```javascript
const http = require('http');

// Custom agent with keep-alive
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 10
});

request
  .get('/api/data')
  .agent(agent);

// Disable agent pooling
request
  .get('/api/data')
  .agent(false);
```
