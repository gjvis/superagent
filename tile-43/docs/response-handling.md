# Response Handling

This document covers accessing and working with HTTP response data, including status codes, headers, body parsing, and convenient status checking properties.

## Response Object

The Response object is passed to callbacks and promise handlers, containing all response data and metadata.

```javascript { .api }
interface Response {
  // Status information
  status: number;
  statusCode: number;
  statusType: number;

  // Body and text
  body: any;
  text: string;

  // Headers
  header: object;
  headers: object;
  type: string;
  charset: string;
  links: object;

  // Status flags
  ok: boolean;
  error: Error | false;
  redirect: boolean;
  clientError: boolean;
  serverError: boolean;
  info: boolean;

  // Status code sugar properties
  created: boolean;
  accepted: boolean;
  noContent: boolean;
  badRequest: boolean;
  unauthorized: boolean;
  forbidden: boolean;
  notFound: boolean;
  notAcceptable: boolean;
  unprocessableEntity: boolean;

  // Node.js specific
  files?: object;
  redirects?: string[];
  req?: any;
  res?: any;
  buffered?: boolean;

  // Methods
  get(field: string): string;
  toError(): Error;
}
```

## Capabilities

### Status Code

Access the HTTP status code.

```javascript { .api }
/**
 * HTTP status code (e.g., 200, 404, 500)
 */
status: number;
statusCode: number;  // Alias for status
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log('Status:', res.status);  // 200
    console.log('Status:', res.statusCode);  // 200 (same)
  });
```

### Status Type

Get the status code class (1xx, 2xx, 3xx, 4xx, or 5xx).

```javascript { .api }
/**
 * Status code class (1-5)
 * 1: Informational, 2: Success, 3: Redirection, 4: Client Error, 5: Server Error
 */
statusType: number;
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    if (res) {
      console.log('Status type:', res.statusType);
      // 200-299 → 2
      // 404 → 4
      // 500 → 5
    }
  });
```

### Response Body

Access the parsed response body.

```javascript { .api }
/**
 * Parsed response body
 * - JSON responses: JavaScript object/array
 * - Form-encoded: JavaScript object
 * - Text: String
 * - Binary: Buffer (Node.js) or Blob/ArrayBuffer (browser)
 */
body: any;
```

**Automatic parsing based on Content-Type:**
- `application/json` → Parsed JSON object/array
- `application/x-www-form-urlencoded` → Parsed object
- `text/*` → String (also available in `.text`)
- `image/*`, `application/octet-stream` → Buffer (Node.js) or binary (browser)
- `multipart/*` → Parsed with fields in `.body` and files in `.files` (Node.js)

**Usage Examples:**

```javascript
// JSON response
request
  .get('/api/users')
  .then(res => {
    console.log('Users:', res.body);  // [{ id: 1, name: 'Alice' }, ...]
    console.log('First user:', res.body[0].name);
  });

// Form-encoded response
request
  .get('/api/data')
  .then(res => {
    console.log('Data:', res.body);  // { key: 'value', ... }
  });

// Text response
request
  .get('/api/message')
  .then(res => {
    console.log('Message:', res.body);  // String content
  });
```

### Response Text

Access the response as raw text string.

```javascript { .api }
/**
 * Response body as text string
 * Available regardless of Content-Type
 */
text: string;
```

**Usage Example:**

```javascript
request
  .get('/api/data')
  .then(res => {
    console.log('Raw text:', res.text);  // '{"key":"value"}'
    console.log('Parsed:', res.body);    // { key: 'value' }
  });
```

### Response Headers

Access response headers.

```javascript { .api }
/**
 * Response headers object (case-insensitive keys)
 */
header: object;
headers: object;  // Alias for header
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log('All headers:', res.header);
    console.log('Content-Type:', res.header['content-type']);
    console.log('Content-Type:', res.headers['content-type']);  // Same
  });
```

### Get Header Value

Get a specific header value with case-insensitive lookup.

```javascript { .api }
/**
 * Get response header value
 * @param field - Header name (case-insensitive)
 * @returns Header value
 */
get(field: string): string;
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log(res.get('Content-Type'));  // 'application/json'
    console.log(res.get('content-type'));  // 'application/json' (case-insensitive)
    console.log(res.get('X-RateLimit-Remaining'));  // '99'
  });
```

### Content Type

Get the response Content-Type without charset.

```javascript { .api }
/**
 * Content-Type MIME type (without charset)
 * e.g., "application/json" for "application/json; charset=utf-8"
 */
type: string;
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log('Type:', res.type);  // 'application/json'
    // Even if full header is 'application/json; charset=utf-8'
  });
```

### Character Set

Get the character set from Content-Type header.

```javascript { .api }
/**
 * Character set from Content-Type header
 * e.g., "utf-8" for "text/html; charset=utf-8"
 */
charset: string;
```

**Usage Example:**

```javascript
request
  .get('/api/page')
  .then(res => {
    console.log('Charset:', res.charset);  // 'utf-8'
  });
```

### Link Header Parsing

Access parsed Link header values.

```javascript { .api }
/**
 * Parsed Link header as object
 * Each link relation maps to its URL
 */
links: object;
```

**Usage Example:**

```javascript
// Response with Link header:
// Link: <https://api.example.com/users?page=2>; rel="next",
//       <https://api.example.com/users?page=5>; rel="last"

request
  .get('/api/users')
  .then(res => {
    console.log(res.links.next);  // 'https://api.example.com/users?page=2'
    console.log(res.links.last);  // 'https://api.example.com/users?page=5'
  });
```

### Status Flags

Boolean properties indicating the response status category.

```javascript { .api }
/**
 * Status flag properties
 */
ok: boolean;          // true for 2xx status codes
error: Error | false; // Error object for 4xx/5xx, false otherwise
redirect: boolean;    // true for 3xx status codes
clientError: boolean; // true for 4xx status codes
serverError: boolean; // true for 5xx status codes
info: boolean;        // true for 1xx status codes
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    if (res && res.ok) {
      console.log('Success:', res.body);
    }

    if (res && res.clientError) {
      console.log('Client error (4xx):', res.status);
    }

    if (res && res.serverError) {
      console.log('Server error (5xx):', res.status);
    }

    if (res && res.error) {
      console.log('Error occurred:', res.error.message);
    }
  });
```

### Status Code Sugar Properties

Convenient boolean properties for common HTTP status codes.

```javascript { .api }
/**
 * Common status code properties
 */
created: boolean;              // 201 Created
accepted: boolean;             // 202 Accepted
noContent: boolean;            // 204 No Content
badRequest: boolean;           // 400 Bad Request
unauthorized: boolean;         // 401 Unauthorized
forbidden: boolean;            // 403 Forbidden
notFound: boolean;             // 404 Not Found
notAcceptable: boolean;        // 406 Not Acceptable
unprocessableEntity: boolean;  // 422 Unprocessable Entity
```

**Usage Examples:**

```javascript
// Check for 201 Created
request
  .post('/api/users')
  .send({ name: 'Alice' })
  .then(res => {
    if (res.created) {
      console.log('User created successfully');
    }
  });

// Check for 404 Not Found
request
  .get('/api/users/999')
  .ok(res => res.ok || res.notFound)  // Don't throw on 404
  .then(res => {
    if (res.notFound) {
      console.log('User not found');
    } else {
      console.log('User:', res.body);
    }
  });

// Check for 204 No Content
request
  .delete('/api/users/123')
  .then(res => {
    if (res.noContent) {
      console.log('User deleted (no content returned)');
    }
  });

// Check for authorization errors
request
  .get('/api/admin')
  .end((err, res) => {
    if (res && res.unauthorized) {
      console.log('Not authenticated');
    } else if (res && res.forbidden) {
      console.log('Not authorized');
    }
  });
```

### Node.js Specific Properties

Additional properties available only in Node.js environment.

```javascript { .api }
/**
 * Multipart form files (Node.js only)
 * Available when response Content-Type is multipart/*
 */
files: object;

/**
 * Array of redirect URLs followed (Node.js only)
 * Populated when request followed redirects
 */
redirects: string[];

/**
 * Internal Node.js request object (Node.js only)
 */
req: http.ClientRequest;

/**
 * Internal Node.js response object (Node.js only)
 */
res: http.IncomingMessage;

/**
 * Whether response body was buffered (Node.js only)
 */
buffered: boolean;
```

**Usage Examples:**

```javascript
// Access redirect history (Node.js)
request
  .get('/api/redirect-test')
  .then(res => {
    console.log('Redirects followed:', res.redirects);
    // ['http://example.com/step1', 'http://example.com/step2']
  });

// Access low-level Node.js response (Node.js)
request
  .get('/api/users')
  .then(res => {
    console.log('HTTP version:', res.res.httpVersion);
    console.log('Raw headers:', res.res.rawHeaders);
  });

// Check buffering status (Node.js)
request
  .get('/api/data')
  .buffer(true)
  .then(res => {
    console.log('Was buffered:', res.buffered);  // true
  });
```

### Convert Response to Error

Convert response to an Error object (used internally).

```javascript { .api }
/**
 * Convert response to Error object
 * Used internally for 4xx/5xx responses
 * @returns Error object with status and response properties
 */
toError(): Error;
```

**Usage Example:**

```javascript
request
  .get('/api/users')
  .then(res => {
    if (res.status >= 400) {
      const error = res.toError();
      console.log(error.message);  // HTTP status message
      console.log(error.status);   // Status code
      console.log(error.response); // Response object
    }
  });
```

### Complete Example

```javascript
const request = require('superagent');

request
  .get('/api/users')
  .query({ page: 1, limit: 10 })
  .then(res => {
    // Status information
    console.log('Status:', res.status);        // 200
    console.log('OK:', res.ok);                // true
    console.log('Status type:', res.statusType); // 2

    // Body and content
    console.log('Users:', res.body);           // Parsed JSON
    console.log('Raw text:', res.text);        // Raw response string
    console.log('Content-Type:', res.type);    // 'application/json'

    // Headers
    console.log('Content-Length:', res.get('Content-Length'));
    console.log('Rate limit:', res.get('X-RateLimit-Remaining'));

    // Pagination links
    if (res.links.next) {
      console.log('Next page:', res.links.next);
    }

    // Process data
    res.body.forEach(user => {
      console.log(`User: ${user.name} (${user.email})`);
    });
  })
  .catch(err => {
    // Error handling
    if (err.response) {
      console.log('Error status:', err.status);
      console.log('Error body:', err.response.body);

      if (err.response.unauthorized) {
        console.log('Authentication required');
      } else if (err.response.notFound) {
        console.log('Resource not found');
      } else if (err.response.serverError) {
        console.log('Server error occurred');
      }
    } else {
      console.log('Network error:', err.message);
    }
  });
```
