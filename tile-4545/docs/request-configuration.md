# Request Configuration

Configure request headers, content types, authentication, and other request properties.

## Capabilities

### Setting Headers

Set custom HTTP headers for the request.

```javascript { .api }
/**
 * Set request header(s)
 * @param field - Header name or object of headers
 * @param value - Header value (if field is a string)
 * @returns Request instance for chaining
 */
set(field: string, value: string): Request;
set(headers: object): Request;

/**
 * Get request header value
 * @param field - Header name (case-insensitive)
 * @returns Header value or undefined
 */
get(field: string): string | undefined;

/**
 * Get request header value (deprecated, use .get() instead)
 * @param field - Header name (case-insensitive)
 * @returns Header value or undefined
 * @deprecated Use .get() instead
 */
getHeader(field: string): string | undefined;

/**
 * Remove request header
 * @param field - Header name to remove
 * @returns Request instance for chaining
 */
unset(field: string): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Set single header
request
  .get('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('X-API-Key', 'abc123');

// Set multiple headers with object
request
  .get('/api/users')
  .set({
    'Authorization': 'Bearer token123',
    'X-API-Key': 'abc123',
    'User-Agent': 'MyApp/1.0'
  });

// Get header value
const req = request.get('/api/users').set('X-Custom', 'value');
console.log(req.get('X-Custom')); // 'value'

// Remove header
request
  .get('/api/users')
  .set('X-Temp', 'temp-value')
  .unset('X-Temp'); // header removed
```

### Content-Type

Set the Content-Type header using shortcuts or full MIME types.

```javascript { .api }
/**
 * Set Content-Type header
 * @param type - MIME type or shortcut ('json', 'xml', 'form', 'html', etc.)
 * @returns Request instance for chaining
 */
type(type: string): Request;
```

**Supported shortcuts:**
- `'json'` → `'application/json'`
- `'xml'` → `'application/xml'` or `'text/xml'`
- `'form'` → `'application/x-www-form-urlencoded'`
- `'form-data'` → `'multipart/form-data'`
- `'html'` → `'text/html'`
- File extensions: `'png'`, `'jpeg'`, etc.

**Usage Examples:**

```javascript
// Set JSON content-type
request
  .post('/api/users')
  .type('json')
  .send({ name: 'John' });

// Set form content-type
request
  .post('/api/login')
  .type('form')
  .send({ username: 'admin', password: 'secret' });

// Set custom MIME type
request
  .post('/api/data')
  .type('application/vnd.api+json')
  .send({ data: {...} });
```

### Accept Header

Set the Accept header to specify desired response format.

```javascript { .api }
/**
 * Set Accept header
 * @param type - MIME type or shortcut ('json', 'xml', 'text', etc.)
 * @returns Request instance for chaining
 */
accept(type: string): Request;
```

**Usage Examples:**

```javascript
// Request JSON response
request
  .get('/api/users')
  .accept('json');

// Request XML response
request
  .get('/api/data')
  .accept('xml');

// Request specific MIME type
request
  .get('/api/data')
  .accept('application/vnd.api+json');
```

### Authentication

Set authentication credentials for the request.

```javascript { .api }
/**
 * Set authentication
 * @param user - Username or token
 * @param pass - Password (optional for bearer auth)
 * @param options - Auth options
 * @param options.type - Auth type: 'basic', 'bearer', or 'auto' (default: 'basic')
 * @returns Request instance for chaining
 */
auth(user: string, pass?: string, options?: {type?: 'basic' | 'bearer' | 'auto'}): Request;
```

**Usage Examples:**

```javascript
// Basic authentication
request
  .get('/api/secure')
  .auth('username', 'password');

// Basic auth with explicit type
request
  .get('/api/secure')
  .auth('username', 'password', {type: 'basic'});

// Bearer token authentication
request
  .get('/api/secure')
  .auth('my-jwt-token', {type: 'bearer'});

// Auto-detect (default to basic)
request
  .get('/api/secure')
  .auth('username', 'password', {type: 'auto'});
```

### Query String Manipulation

Add or modify query string parameters.

```javascript { .api }
/**
 * Add query string parameters
 * @param params - Query parameters as object or string
 * @returns Request instance for chaining
 */
query(params: object | string): Request;

/**
 * Sort query string parameters
 * @param compareFn - Optional custom sort function
 * @returns Request instance for chaining
 */
sortQuery(compareFn?: (a: string, b: string) => number): Request;
```

**Usage Examples:**

```javascript
// Query with object
request
  .get('/api/users')
  .query({ role: 'admin', active: true });
// Result: /api/users?role=admin&active=true

// Query with string
request
  .get('/api/users')
  .query('role=admin&active=true');

// Multiple query calls (merged)
request
  .get('/api/users')
  .query({ role: 'admin' })
  .query({ active: true })
  .query({ limit: 10 });
// Result: /api/users?role=admin&active=true&limit=10

// Sort query parameters alphabetically
request
  .get('/api/users')
  .query({ z: 1, a: 2, m: 3 })
  .sortQuery();
// Result: /api/users?a=2&m=3&z=1

// Custom sort function
request
  .get('/api/users')
  .query({ z: 1, a: 2 })
  .sortQuery((a, b) => b.localeCompare(a)); // reverse order
```

### CORS Credentials

Enable sending credentials (cookies, authorization headers) with cross-origin requests.

```javascript { .api }
/**
 * Enable cross-origin credentials
 * @param enable - Enable/disable credentials (default: true)
 * @returns Request instance for chaining
 */
withCredentials(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable credentials for cross-origin request
request
  .get('https://api.example.com/data')
  .withCredentials()
  .end((err, res) => {
    console.log(res.body);
  });

// Explicitly disable
request
  .get('https://api.example.com/data')
  .withCredentials(false);
```

**Note**: This is primarily used in browser environments for CORS requests. The server must respond with appropriate `Access-Control-Allow-Credentials` headers.

### Custom Success Validation

Customize what response status codes are considered successful.

```javascript { .api }
/**
 * Custom success validation
 * @param callback - Function to determine if response is successful
 * @returns Request instance for chaining
 */
ok(callback: (res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Accept 404 as success
request
  .get('/api/users/123')
  .ok(res => res.status < 500)
  .then(res => {
    if (res.status === 404) {
      console.log('User not found, but not an error');
    }
  });

// Custom validation logic
request
  .get('/api/data')
  .ok(res => {
    // Consider 200-299 and 304 as success
    return (res.status >= 200 && res.status < 300) || res.status === 304;
  });
```

### Request Serialization

Customize how request body data is serialized.

```javascript { .api }
/**
 * Set custom request serializer
 * @param fn - Serializer function
 * @returns Request instance for chaining
 */
serialize(fn: (obj: any) => string): Request;
```

**Usage Examples:**

```javascript
// Custom serializer for special format
request
  .post('/api/data')
  .type('application/x-custom')
  .serialize(obj => {
    // Custom serialization logic
    return Object.keys(obj)
      .map(k => `${k}=${obj[k]}`)
      .join('&');
  })
  .send({ a: 1, b: 2 });
```
