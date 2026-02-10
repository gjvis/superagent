# Request Configuration

Configure request headers, query parameters, body content, and other request options.

## Capabilities

### Setting Headers

Set request headers with case-insensitive field names.

```javascript { .api }
/**
 * Set a single request header
 * @param field - Header name (case-insensitive)
 * @param value - Header value
 * @returns Request instance for chaining
 */
Request.prototype.set(field: string, value: string): Request;

/**
 * Set multiple request headers from an object
 * @param headers - Object with header name-value pairs
 * @returns Request instance for chaining
 */
Request.prototype.set(headers: object): Request;
```

**Usage Examples:**

```javascript
// Set single header
request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer token123')
  .set('Accept', 'application/json');

// Set multiple headers
request.get('https://api.example.com/users')
  .set({
    'Authorization': 'Bearer token123',
    'Accept': 'application/json',
    'X-API-Key': 'secret'
  });
```

### Getting Headers

Retrieve request header values.

```javascript { .api }
/**
 * Get a request header value
 * @param field - Header name (case-insensitive)
 * @returns Header value or undefined
 */
Request.prototype.get(field: string): string | undefined;
```

**Usage Examples:**

```javascript
const req = request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer token123');

console.log(req.get('Authorization')); // 'Bearer token123'
console.log(req.get('authorization')); // 'Bearer token123' (case-insensitive)
```

### Removing Headers

Remove request headers.

```javascript { .api }
/**
 * Remove a request header
 * @param field - Header name (case-insensitive)
 * @returns Request instance for chaining
 */
Request.prototype.unset(field: string): Request;
```

**Usage Examples:**

```javascript
request.get('https://api.example.com/users')
  .set('X-Custom-Header', 'value')
  .unset('X-Custom-Header'); // Header removed
```

### Content-Type Header

Set the Content-Type header using convenience method.

```javascript { .api }
/**
 * Set Content-Type header
 * @param contentType - MIME type (e.g., 'json', 'xml', 'application/json')
 * @returns Request instance for chaining
 */
Request.prototype.type(contentType: string): Request;
```

**Usage Examples:**

```javascript
// Using shorthand
request.post('https://api.example.com/users')
  .type('json')  // Sets 'application/json'
  .send('{"name":"John"}');

// Using full MIME type
request.post('https://api.example.com/users')
  .type('application/json')
  .send('{"name":"John"}');

// Common shorthand types
request.post('/api').type('form');        // application/x-www-form-urlencoded
request.post('/api').type('json');        // application/json
request.post('/api').type('xml');         // application/xml
request.post('/api').type('html');        // text/html
request.post('/api').type('text');        // text/plain
```

### Accept Header

Set the Accept header using convenience method.

```javascript { .api }
/**
 * Set Accept header
 * @param acceptType - MIME type (e.g., 'json', 'xml', 'application/json')
 * @returns Request instance for chaining
 */
Request.prototype.accept(acceptType: string): Request;
```

**Usage Examples:**

```javascript
// Using shorthand
request.get('https://api.example.com/users')
  .accept('json');  // Sets 'application/json'

// Using full MIME type
request.get('https://api.example.com/users')
  .accept('application/json');
```

### Query Parameters

Add query string parameters to the request URL.

```javascript { .api }
/**
 * Add query parameters from an object
 * @param params - Object with key-value pairs
 * @returns Request instance for chaining
 */
Request.prototype.query(params: object): Request;

/**
 * Add query parameters from a string
 * @param queryString - URL-encoded query string
 * @returns Request instance for chaining
 */
Request.prototype.query(queryString: string): Request;
```

**Usage Examples:**

```javascript
// Using object
request.get('https://api.example.com/users')
  .query({ page: 1, limit: 10, sort: 'name' });
// Results in: /users?page=1&limit=10&sort=name

// Using string
request.get('https://api.example.com/users')
  .query('page=1&limit=10');

// Chaining multiple .query() calls merges parameters
request.get('https://api.example.com/users')
  .query({ page: 1 })
  .query({ limit: 10 })
  .query({ sort: 'name' });

// Arrays in query parameters
request.get('https://api.example.com/users')
  .query({ tags: ['javascript', 'nodejs'] });
// Results in: /users?tags=javascript&tags=nodejs

// Nested objects
request.get('https://api.example.com/users')
  .query({ filter: { status: 'active', role: 'admin' } });
// Results in: /users?filter[status]=active&filter[role]=admin
```

### Sorting Query Parameters

Sort query string parameters alphabetically.

```javascript { .api }
/**
 * Sort query parameters alphabetically
 * @param compareFn - Optional custom comparison function
 * @returns Request instance for chaining
 */
Request.prototype.sortQuery(compareFn?: (a: string, b: string) => number): Request;
```

**Usage Examples:**

```javascript
// Default alphabetical sort
request.get('https://api.example.com/users')
  .query({ z: 1, a: 2, m: 3 })
  .sortQuery();
// Results in: /users?a=2&m=3&z=1

// Custom sort function
request.get('https://api.example.com/users')
  .query({ z: 1, a: 2, m: 3 })
  .sortQuery((a, b) => b.localeCompare(a)); // Reverse order
```

### Sending Request Body

Send data as the request body with automatic serialization.

```javascript { .api }
/**
 * Send request body data
 * @param data - Request body (object, string, or Buffer)
 * @returns Request instance for chaining
 */
Request.prototype.send(data: object | string | Buffer): Request;
```

**Usage Examples:**

```javascript
// Send JSON (automatically sets Content-Type: application/json)
request.post('https://api.example.com/users')
  .send({ name: 'John', email: 'john@example.com' });

// Send string
request.post('https://api.example.com/data')
  .type('text')
  .send('plain text data');

// Send Buffer (Node.js)
const buffer = Buffer.from('binary data');
request.post('https://api.example.com/upload')
  .send(buffer);

// Multiple .send() calls for objects merge the data
request.post('https://api.example.com/users')
  .send({ name: 'John' })
  .send({ email: 'john@example.com' });
// Results in: { name: 'John', email: 'john@example.com' }

// Sending arrays
request.post('https://api.example.com/bulk')
  .send([{ id: 1 }, { id: 2 }, { id: 3 }]);
```

### Multipart Form Fields

Add form fields for multipart/form-data requests.

```javascript { .api }
/**
 * Add a single form field
 * @param name - Field name
 * @param value - Field value
 * @returns Request instance for chaining
 */
Request.prototype.field(name: string, value: any): Request;

/**
 * Add multiple form fields from an object
 * @param fields - Object with field name-value pairs
 * @returns Request instance for chaining
 */
Request.prototype.field(fields: object): Request;
```

**Usage Examples:**

```javascript
// Single field
request.post('https://api.example.com/upload')
  .field('name', 'John')
  .field('email', 'john@example.com')
  .attach('avatar', '/path/to/image.jpg');

// Multiple fields from object
request.post('https://api.example.com/upload')
  .field({
    name: 'John',
    email: 'john@example.com',
    age: 30
  })
  .attach('avatar', '/path/to/image.jpg');

// Array values (creates multiple fields with same name)
request.post('https://api.example.com/upload')
  .field('tags', ['javascript', 'nodejs', 'web']);
```

**Important Notes:**

- Cannot mix `.send()` with `.field()` or `.attach()` - use one approach per request
- Using `.field()` or `.attach()` automatically sets Content-Type to `multipart/form-data`
- Boolean values are converted to strings ('true', 'false')
