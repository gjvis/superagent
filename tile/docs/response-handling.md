# Response Handling

The Response class provides comprehensive access to HTTP response data including status information, headers, body content, and parsed data. Every request made with SuperAgent returns a Response instance that contains all the information about the server's response, with automatic parsing based on content type and convenient boolean properties for checking response status.

## Response Object

The Response object is returned when a request completes successfully or encounters an HTTP error (4xx/5xx). It provides both the raw response data and parsed content, along with numerous convenience properties for checking status codes.

```javascript { .api }
/**
 * Response class representing an HTTP response
 * @constructor
 * @param {Request} request - The request that generated this response
 */
interface Response {
  // Status properties
  status: number;
  statusCode: number;
  statusType: number;
  ok: boolean;
  error: Error | false;
  clientError: boolean;
  serverError: boolean;
  info: boolean;
  redirect: boolean;

  // Status sugar properties
  created: boolean;
  accepted: boolean;
  noContent: boolean;
  badRequest: boolean;
  unauthorized: boolean;
  forbidden: boolean;
  notFound: boolean;
  notAcceptable: boolean;
  unprocessableEntity: boolean;

  // Content properties
  body: any;
  text: string;
  type: string;
  charset: string;
  header: object;
  headers: object;
  files: object;
  redirects: string[];
  links: object;

  // Request reference
  req: Request;
  request: Request;

  // Methods
  get(field: string): string;
  toError(): Error;
  toJSON(): object;
}
```

## Status Properties

Response status properties provide boolean flags and numeric values for understanding the HTTP response status code.

### Core Status Properties

```javascript { .api }
/**
 * HTTP status code
 * @type {number}
 * @example res.status // 200, 404, 500, etc.
 */
status: number;

/**
 * Alias for status
 * @type {number}
 */
statusCode: number;

/**
 * Status class (1-5 representing 1xx, 2xx, 3xx, 4xx, 5xx)
 * @type {number}
 * @example res.statusType // 2 for 200, 4 for 404, 5 for 500
 */
statusType: number;

/**
 * True for 2xx status codes (successful responses)
 * @type {boolean}
 * @example res.ok // true for 200-299
 */
ok: boolean;

/**
 * Error object for 4xx/5xx responses, false otherwise
 * @type {Error | false}
 * @example
 * if (res.error) {
 *   console.error(res.error.message);
 * }
 */
error: Error | false;
```

### Status Category Properties

```javascript { .api }
/**
 * True for 1xx informational responses
 * @type {boolean}
 * @example res.info // true for 100-199
 */
info: boolean;

/**
 * True for 3xx redirect responses
 * @type {boolean}
 * @example res.redirect // true for 300-399
 */
redirect: boolean;

/**
 * True for 4xx client error responses
 * @type {boolean}
 * @example res.clientError // true for 400-499
 */
clientError: boolean;

/**
 * True for 5xx server error responses
 * @type {boolean}
 * @example res.serverError // true for 500-599
 */
serverError: boolean;
```

### Status Sugar Properties

Convenient boolean properties for common HTTP status codes.

```javascript { .api }
/**
 * True for 201 Created
 * @type {boolean}
 */
created: boolean;

/**
 * True for 202 Accepted
 * @type {boolean}
 */
accepted: boolean;

/**
 * True for 204 No Content
 * @type {boolean}
 */
noContent: boolean;

/**
 * True for 400 Bad Request
 * @type {boolean}
 */
badRequest: boolean;

/**
 * True for 401 Unauthorized
 * @type {boolean}
 */
unauthorized: boolean;

/**
 * True for 403 Forbidden
 * @type {boolean}
 */
forbidden: boolean;

/**
 * True for 404 Not Found
 * @type {boolean}
 */
notFound: boolean;

/**
 * True for 406 Not Acceptable
 * @type {boolean}
 */
notAcceptable: boolean;

/**
 * True for 422 Unprocessable Entity
 * @type {boolean}
 */
unprocessableEntity: boolean;
```

## Content Properties

Properties providing access to the response body, headers, and metadata.

### Body Content

```javascript { .api }
/**
 * Parsed response body
 * Automatically parsed based on Content-Type header
 * - JSON responses parsed to objects
 * - Form-encoded responses parsed to objects
 * - Text responses returned as strings
 * - Binary responses returned as Buffer (Node.js) or Blob (browser)
 * @type {any}
 * @example
 * const res = await request.get('/api/user');
 * console.log(res.body.name); // Access parsed JSON
 */
body: any;

/**
 * Response body as string
 * Available for text-based responses
 * Null for binary responses or non-text response types
 * @type {string | null}
 * @example
 * const res = await request.get('/page.html');
 * console.log(res.text); // HTML content as string
 */
text: string | null;
```

### Content Type Information

```javascript { .api }
/**
 * Content-Type without parameters
 * Extracted from Content-Type header, excluding charset and other parameters
 * @type {string}
 * @example
 * // For "Content-Type: application/json; charset=utf-8"
 * res.type // "application/json"
 */
type: string;

/**
 * Character encoding from Content-Type header
 * Extracted from charset parameter in Content-Type
 * @type {string}
 * @example
 * // For "Content-Type: text/html; charset=utf-8"
 * res.charset // "utf-8"
 */
charset: string;
```

### Headers

```javascript { .api }
/**
 * Response headers object
 * All header names normalized to lowercase
 * @type {object}
 * @example
 * res.header['content-type'] // "application/json"
 * res.header['x-custom-header'] // "custom-value"
 */
header: object;

/**
 * Alias for header
 * @type {object}
 */
headers: object;
```

### Additional Content Properties

```javascript { .api }
/**
 * Uploaded files (for multipart responses)
 * Contains information about files in multipart responses
 * @type {object}
 */
files: object;

/**
 * Array of redirect URLs followed
 * Contains the chain of URLs if request followed redirects
 * @type {string[]}
 * @example
 * res.redirects // ['/moved', '/moved-again', '/final']
 */
redirects: string[];

/**
 * Parsed Link header object
 * Parses RFC 5988 Link headers into an object
 * @type {object}
 * @example
 * // For Link: </page/2>; rel="next", </page/5>; rel="last"
 * res.links.next // "/page/2"
 * res.links.last // "/page/5"
 */
links: object;
```

## Response Methods

### Getting Header Values

```javascript { .api }
/**
 * Get response header value (case-insensitive)
 * @param {string} field - Header field name
 * @returns {string} Header value
 * @example
 * res.get('Content-Type') // "application/json; charset=utf-8"
 * res.get('content-type') // same result (case-insensitive)
 * res.get('X-RateLimit-Remaining') // "100"
 */
get(field: string): string;
```

### Converting to Error

```javascript { .api }
/**
 * Convert response to Error object
 * Creates an Error with response information
 * Used internally for 4xx/5xx responses
 * @returns {Error} Error object with status, method, and path/url
 * @example
 * const err = res.toError();
 * console.log(err.message); // "cannot GET /api/user (404)"
 * console.log(err.status); // 404
 */
toError(): Error;
```

### Converting to JSON

```javascript { .api }
/**
 * Convert response to plain object (Node.js only)
 * @returns {object} Object with req, header, status, and text
 * @example
 * const json = res.toJSON();
 * console.log(json.status); // 200
 * console.log(json.header); // headers object
 */
toJSON(): object;
```

## Usage with Promises

When using promises or async/await, you receive the Response object directly on success, or an Error with response attached on failure.

```javascript
// Successful response
try {
  const res = await request.get('/api/users');

  // Access response properties
  console.log(res.status);        // 200
  console.log(res.ok);            // true
  console.log(res.type);          // "application/json"
  console.log(res.body);          // Parsed JSON object
  console.log(res.body.users);    // Access parsed data
  console.log(res.get('Content-Length')); // Get header

} catch (err) {
  // HTTP error (4xx/5xx) or network error
  if (err.response) {
    // HTTP error - response received
    const res = err.response;
    console.log(res.status);      // 404, 500, etc.
    console.log(res.clientError); // true for 4xx
    console.log(res.serverError); // true for 5xx
    console.log(res.notFound);    // true for 404
    console.log(res.body);        // Error response body
    console.log(res.text);        // Error response text
  } else {
    // Network error - no response received
    console.log(err.message);     // "ECONNREFUSED", "ETIMEDOUT", etc.
  }
}
```

## Usage with Callbacks

When using callbacks, the Response object is passed as the second argument, while errors are passed as the first argument.

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      // HTTP error (4xx/5xx) or network error
      console.log(err.status);    // HTTP status if available
      if (err.response) {
        // HTTP error - response available on error object
        console.log(err.response.body);
        console.log(err.response.text);
      }
      return;
    }

    // Success - access response
    console.log(res.status);      // 200
    console.log(res.ok);          // true
    console.log(res.body);        // Parsed response
    console.log(res.text);        // Raw text
    console.log(res.get('Date')); // Response headers
  });
```

## Accessing Response Data

### Parsed vs Raw Response

SuperAgent automatically parses response bodies based on Content-Type:

```javascript
// JSON responses - automatically parsed
const res = await request.get('/api/user');
console.log(res.body);          // { id: 1, name: "Alice" }
console.log(res.body.name);     // "Alice"
console.log(res.text);          // '{"id":1,"name":"Alice"}'

// HTML/Text responses
const res = await request.get('/page.html');
console.log(res.text);          // "<html>...</html>"
console.log(res.body);          // Same as text for HTML

// Form-encoded responses - automatically parsed
const res = await request.get('/form-data');
console.log(res.body);          // { field1: "value1", field2: "value2" }
console.log(res.text);          // "field1=value1&field2=value2"
```

### Checking Response Status

Use the status properties to handle different response scenarios:

```javascript
const res = await request.get('/api/resource');

// Check for success
if (res.ok) {
  console.log('Success!', res.body);
}

// Check specific status codes
if (res.created) {
  console.log('Resource created at', res.get('Location'));
}

if (res.noContent) {
  console.log('No content to display');
}

// Handle errors
if (res.clientError) {
  if (res.unauthorized) {
    console.log('Please log in');
  } else if (res.notFound) {
    console.log('Resource not found');
  } else if (res.badRequest) {
    console.log('Invalid request:', res.body.message);
  }
}

if (res.serverError) {
  console.log('Server error:', res.status);
}

// Check status type
switch (res.statusType) {
  case 1: console.log('Informational'); break;
  case 2: console.log('Success'); break;
  case 3: console.log('Redirect'); break;
  case 4: console.log('Client error'); break;
  case 5: console.log('Server error'); break;
}
```

### Reading Headers

Access response headers using the `get()` method or directly from the `header` object:

```javascript
const res = await request.get('/api/data');

// Case-insensitive header access
console.log(res.get('Content-Type'));
console.log(res.get('content-type'));
console.log(res.get('X-RateLimit-Limit'));

// Direct header object access (lowercase keys)
console.log(res.header['content-type']);
console.log(res.header['x-ratelimit-limit']);

// Common headers
const contentType = res.get('Content-Type');
const contentLength = res.get('Content-Length');
const etag = res.get('ETag');
const lastModified = res.get('Last-Modified');
const location = res.get('Location');
```

### Working with Links

SuperAgent automatically parses RFC 5988 Link headers:

```javascript
// Server response includes:
// Link: </api/page/2>; rel="next", </api/page/10>; rel="last"

const res = await request.get('/api/page/1');

console.log(res.links.next);  // "/api/page/2"
console.log(res.links.last);  // "/api/page/10"
console.log(res.links.prev);  // undefined (not present)

// Pagination example
if (res.links.next) {
  const nextPage = await request.get(res.links.next);
  console.log(nextPage.body);
}
```

### Tracking Redirects

The `redirects` array contains all URLs visited if the request followed redirects:

```javascript
const res = await request
  .get('/old-url')
  .redirects(5); // Allow up to 5 redirects

console.log(res.redirects);
// ['/moved', '/moved-again', '/final-destination']

// Check if redirected
if (res.redirects.length > 0) {
  console.log('Request was redirected', res.redirects.length, 'times');
  console.log('Redirect chain:', res.redirects);
}
```

## Error Response Handling

When a request fails with a 4xx or 5xx status code, the error object contains a reference to the response:

```javascript
try {
  const res = await request
    .post('/api/users')
    .send({ name: '' }); // Invalid data

} catch (err) {
  if (err.status === 400) {
    // Access error response body
    console.log('Validation errors:', err.response.body.errors);
    console.log('Error message:', err.response.body.message);

    // Check error response properties
    console.log('Bad request:', err.response.badRequest); // true
    console.log('Status:', err.response.status);          // 400
  }

  if (err.status === 401) {
    console.log('Not authenticated');
    console.log('WWW-Authenticate:', err.response.get('WWW-Authenticate'));
  }

  if (err.status === 404) {
    console.log('Resource not found');
  }

  if (err.status >= 500) {
    console.log('Server error:', err.response.text);
  }
}
```

## Custom Success Handling

Use `.ok()` to customize what responses are considered successful:

```javascript
// By default, only 2xx status codes are successful
// Customize to treat 404 as success
const res = await request
  .get('/api/user/123')
  .ok(res => res.status < 500); // Treat 4xx as success too

// Now 404 won't throw an error
if (res.notFound) {
  console.log('User not found, creating new user...');
} else {
  console.log('User exists:', res.body);
}
```

## Platform Differences

### Node.js Specific Features

```javascript
// Node.js Response extends Stream
const fs = require('fs');
const request = require('superagent');

// Stream response to file
request
  .get('/api/large-file')
  .pipe(fs.createWriteStream('output.bin'));

// Listen to streaming events
request
  .get('/api/data')
  .on('data', chunk => console.log('Received chunk:', chunk.length))
  .on('end', () => console.log('Stream complete'))
  .pipe(destination);

// Response streaming methods
response.pause();   // Pause the response stream
response.resume();  // Resume the response stream
response.destroy(); // Destroy the response stream

// toJSON() method available
const json = res.toJSON();
console.log(json.status, json.header, json.text);
```

### Browser Specific Features

```javascript
// Browser Response properties
console.log(res.xhr);        // XMLHttpRequest object
console.log(res.statusText); // Status text like "OK", "Not Found"

// Response types for binary data
await request
  .get('/image.png')
  .responseType('blob');

console.log(res.body); // Blob object in browser
```

## Examples

### Basic Response Handling

```javascript
// Simple GET request
const res = await request.get('/api/users');
console.log(res.body); // Array of users
console.log(res.status); // 200
console.log(res.ok); // true

// POST request with response
const res = await request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' });

console.log(res.created); // true
console.log(res.status); // 201
console.log(res.body.id); // New user ID
console.log(res.get('Location')); // URL of created resource
```

### Comprehensive Error Handling

```javascript
async function fetchUser(userId) {
  try {
    const res = await request.get(`/api/users/${userId}`);

    if (res.ok) {
      return res.body;
    }

  } catch (err) {
    const res = err.response;

    if (!res) {
      // Network error
      console.error('Network error:', err.message);
      throw new Error('Failed to connect to server');
    }

    // HTTP error - response available
    switch (res.status) {
      case 400:
        throw new Error('Invalid user ID');
      case 401:
        throw new Error('Authentication required');
      case 403:
        throw new Error('Access denied');
      case 404:
        return null; // User not found
      case 500:
      case 503:
        throw new Error('Server error, please try again later');
      default:
        throw new Error(`Request failed: ${res.status}`);
    }
  }
}
```

### Processing Different Content Types

```javascript
// JSON response
const jsonRes = await request
  .get('/api/data.json')
  .accept('application/json');
console.log(jsonRes.body.items); // Parsed JSON

// HTML response
const htmlRes = await request.get('/page.html');
console.log(htmlRes.text); // HTML string
console.log(htmlRes.type); // "text/html"

// Binary response (Node.js)
const binRes = await request.get('/file.pdf');
console.log(Buffer.isBuffer(binRes.body)); // true
fs.writeFileSync('output.pdf', binRes.body);

// Form-encoded response
const formRes = await request.get('/form-data');
console.log(formRes.body); // Parsed object
console.log(formRes.text); // Raw form-encoded string
```

### Working with Headers and Metadata

```javascript
const res = await request.get('/api/data');

// Content negotiation
console.log('Content-Type:', res.type);
console.log('Character set:', res.charset);

// Caching headers
console.log('ETag:', res.get('ETag'));
console.log('Last-Modified:', res.get('Last-Modified'));
console.log('Cache-Control:', res.get('Cache-Control'));

// Rate limiting
console.log('Rate limit:', res.get('X-RateLimit-Limit'));
console.log('Remaining:', res.get('X-RateLimit-Remaining'));
console.log('Reset:', res.get('X-RateLimit-Reset'));

// Custom headers
console.log('Request ID:', res.get('X-Request-ID'));
console.log('Response time:', res.get('X-Response-Time'));
```

### Pagination with Link Headers

```javascript
async function fetchAllPages(url) {
  const allData = [];
  let currentUrl = url;

  while (currentUrl) {
    const res = await request.get(currentUrl);
    allData.push(...res.body.items);

    // Follow next link if present
    currentUrl = res.links.next;

    console.log(`Fetched page, total items: ${allData.length}`);
    if (res.links.next) {
      console.log('Next page:', res.links.next);
    }
  }

  return allData;
}

const allUsers = await fetchAllPages('/api/users?page=1');
console.log(`Total users: ${allUsers.length}`);
```
