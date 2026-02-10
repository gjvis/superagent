# Request Configuration

This document covers basic request configuration including headers, query parameters, request body, and content types.

## Capabilities

### Setting Request Headers

Set HTTP headers for the request using `.set()`. Headers can be set individually or as an object.

```javascript { .api }
/**
 * Set request header field to value
 * @param field - Header name (case-insensitive)
 * @param value - Header value
 * @returns Request instance for chaining
 */
set(field: string, value: string): Request;

/**
 * Set multiple headers from object
 * @param headers - Object with header name/value pairs
 * @returns Request instance for chaining
 */
set(headers: object): Request;
```

**Usage Examples:**

```javascript
// Set individual header
request
  .get('/api/users')
  .set('Accept', 'application/json')
  .set('X-API-Key', 'secret123');

// Set multiple headers
request
  .post('/api/users')
  .set({
    'Accept': 'application/json',
    'X-API-Key': 'secret123',
    'X-Request-ID': 'abc-123'
  });
```

### Getting Request Headers

Retrieve a previously set request header value.

```javascript { .api }
/**
 * Get request header value
 * @param field - Header name (case-insensitive)
 * @returns Header value or undefined
 */
get(field: string): string | undefined;
```

**Usage Example:**

```javascript
const req = request.get('/api/users').set('X-API-Key', 'secret');
console.log(req.get('x-api-key'));  // 'secret' (case-insensitive)
```

### Removing Request Headers

Remove a previously set header.

```javascript { .api }
/**
 * Remove request header
 * @param field - Header name (case-insensitive)
 * @returns Request instance for chaining
 */
unset(field: string): Request;
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .set('X-API-Key', 'secret')
  .unset('X-API-Key');  // Header removed
```

### Setting Content-Type

Set the Content-Type header using MIME types or shortcuts.

```javascript { .api }
/**
 * Set Content-Type header
 * @param type - MIME type or shortcut (json, xml, form, html, etc.)
 * @returns Request instance for chaining
 */
type(contentType: string): Request;
```

**Supported shortcuts:**
- `'json'` → `'application/json'`
- `'form'` → `'application/x-www-form-urlencoded'`
- `'xml'` → `'text/xml'` (browser) or resolved via mime library (Node.js)
- `'html'` → `'text/html'` (browser) or resolved via mime library (Node.js)
- Or use full MIME type: `'application/json'`, `'text/plain'`, etc.

**Usage Examples:**

```javascript
// Using shortcut
request
  .post('/api/users')
  .type('json')
  .send({ name: 'Alice' });

// Using full MIME type
request
  .post('/api/users')
  .type('application/json')
  .send({ name: 'Alice' });

// Form data
request
  .post('/api/users')
  .type('form')
  .send('name=Alice&email=alice@example.com');
```

### Setting Accept Header

Set the Accept header to indicate desired response format.

```javascript { .api }
/**
 * Set Accept header
 * @param type - MIME type or shortcut
 * @returns Request instance for chaining
 */
accept(acceptType: string): Request;
```

**Usage Examples:**

```javascript
// Using shortcut
request
  .get('/api/users')
  .accept('json');

// Using full MIME type
request
  .get('/api/users')
  .accept('application/json');
```

### Adding Query Parameters

Add query string parameters to the request URL. Can be called multiple times to build complex query strings.

```javascript { .api }
/**
 * Add query parameters to request
 * @param params - Query parameters as object or string
 * @returns Request instance for chaining
 */
query(params: object | string): Request;
```

**Usage Examples:**

```javascript
// Using object
request
  .get('/api/users')
  .query({ page: 2, limit: 20 });
// Results in: /api/users?page=2&limit=20

// Using string
request
  .get('/api/users')
  .query('page=2&limit=20');

// Multiple calls are cumulative
request
  .get('/api/users')
  .query({ page: 2 })
  .query({ limit: 20 });
// Results in: /api/users?page=2&limit=20

// For GET/HEAD with data parameter
request.get('/api/users', { page: 2, limit: 20 });
// Equivalent to .query({ page: 2, limit: 20 })
```

### Sorting Query Parameters

Sort query parameters alphabetically or with a custom function.

```javascript { .api }
/**
 * Sort query string parameters
 * @param compareFn - Optional comparison function for sorting
 * @returns Request instance for chaining
 */
sortQuery(compareFn?: (a: string, b: string) => number): Request;
```

**Usage Examples:**

```javascript
// Default alphabetical sort
request
  .get('/api/users')
  .query({ name: 'Alice', age: 25, city: 'NYC' })
  .sortQuery();
// Results in: /api/users?age=25&city=NYC&name=Alice

// Custom sort function
request
  .get('/api/users')
  .query({ name: 'Alice', age: 25 })
  .sortQuery((a, b) => a.length - b.length);
// Sort by parameter length
```

### Sending Request Body

Send data in the request body. Automatically sets Content-Type based on data type.

```javascript { .api }
/**
 * Send request body data
 * @param data - Data to send (object, string, Buffer, etc.)
 * @returns Request instance for chaining
 */
send(data: any): Request;
```

**Behavior:**
- **Objects/Arrays**: Automatically serialized to JSON and Content-Type set to `application/json`
- **Strings** (without Content-Type): Content-Type set to `application/x-www-form-urlencoded`
- **Strings** (with Content-Type): Sent as-is with specified Content-Type
- **Multiple calls**: Objects are merged; strings are concatenated

**Usage Examples:**

```javascript
// Send JSON object
request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' });
// Automatically sets Content-Type: application/json

// Send form data as string
request
  .post('/api/users')
  .send('name=Alice&email=alice@example.com');
// Automatically sets Content-Type: application/x-www-form-urlencoded

// Merge multiple objects
request
  .post('/api/users')
  .send({ name: 'Alice' })
  .send({ email: 'alice@example.com' });
// Results in: { name: 'Alice', email: 'alice@example.com' }

// Concatenate strings with form type
request
  .post('/api/users')
  .type('form')
  .send('name=Alice')
  .send('email=alice@example.com');
// Results in: name=Alice&email=alice@example.com

// Send Buffer (Node.js)
const buffer = Buffer.from('raw data');
request
  .post('/api/data')
  .send(buffer);

// Using data parameter in verb methods
request.post('/api/users', { name: 'Alice' });
// Equivalent to .send({ name: 'Alice' })
```

### Complete Example

```javascript
const request = require('superagent');

// Comprehensive request configuration
request
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('X-Request-ID', 'abc-123')
  .accept('json')
  .type('json')
  .query({ include: 'profile' })
  .send({
    name: 'Alice',
    email: 'alice@example.com',
    age: 30
  })
  .then(res => {
    console.log('User created:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```
