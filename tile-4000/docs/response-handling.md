# Response Handling

Access and interpret HTTP responses including status, headers, and parsed body content.

## Capabilities

### Response Status

Access HTTP status code and status-based boolean flags.

```javascript { .api }
interface Response {
  /** HTTP status code (200, 404, 500, etc.) */
  status: number;

  /** Alias for status */
  statusCode: number;

  /** Status class: 1 (info), 2 (success), 3 (redirect), 4 (client error), 5 (server error) */
  statusType: number;

  /** True if status is 2xx (success) */
  ok: boolean;

  /** Error object if 4xx/5xx, false otherwise */
  error: Error | false;

  /** True if status is 4xx (client error) */
  clientError: boolean;

  /** True if status is 5xx (server error) */
  serverError: boolean;

  /** True if status is 3xx (redirect) */
  redirect: boolean;

  /** True if status is 1xx (informational) */
  info: boolean;
}
```

**Usage Examples:**

```javascript
const res = await request.get('https://api.example.com/users');

console.log('Status:', res.status);           // 200
console.log('Status code:', res.statusCode);  // 200
console.log('Status type:', res.statusType);  // 2
console.log('OK:', res.ok);                   // true
console.log('Error:', res.error);             // false

// Status checking
if (res.ok) {
  console.log('Success');
}

if (res.clientError) {
  console.log('Client error (4xx)');
}

if (res.serverError) {
  console.log('Server error (5xx)');
}
```

### Status Sugar Properties

Convenient boolean properties for specific HTTP status codes.

```javascript { .api }
interface Response {
  /** True if status is 201 (Created) */
  created: boolean;

  /** True if status is 202 (Accepted) */
  accepted: boolean;

  /** True if status is 204 (No Content) */
  noContent: boolean;

  /** True if status is 400 (Bad Request) */
  badRequest: boolean;

  /** True if status is 401 (Unauthorized) */
  unauthorized: boolean;

  /** True if status is 403 (Forbidden) */
  forbidden: boolean;

  /** True if status is 404 (Not Found) */
  notFound: boolean;

  /** True if status is 406 (Not Acceptable) */
  notAcceptable: boolean;

  /** True if status is 422 (Unprocessable Entity) */
  unprocessableEntity: boolean;
}
```

**Usage Examples:**

```javascript
const res = await request.post('https://api.example.com/users')
  .send({ name: 'John' });

if (res.created) {
  console.log('User created successfully');
}

// Error handling
try {
  const res = await request.get('https://api.example.com/users/999');
} catch (err) {
  if (err.response && err.response.notFound) {
    console.log('User not found');
  }
}

// Checking specific statuses
if (res.unauthorized) {
  // Redirect to login
}

if (res.badRequest) {
  // Show validation errors
}

if (res.noContent) {
  // No body to parse
}
```

### Response Body

Parsed response body and raw text content.

```javascript { .api }
interface Response {
  /** Parsed response body (object for JSON, string for text, Buffer/Blob for binary) */
  body: any;

  /** Raw response body as string */
  text: string;
}
```

**Usage Examples:**

```javascript
// JSON response (automatically parsed)
const res = await request.get('https://api.example.com/users');
console.log('Users:', res.body);         // Array or object
console.log('Raw JSON:', res.text);      // String

// Text response
const res = await request.get('https://api.example.com/page.html');
console.log('HTML:', res.text);
console.log('Body:', res.body);          // Same as res.text for text/*

// Binary response (Node.js)
const res = await request.get('https://api.example.com/image.png');
console.log('Buffer:', res.body);        // Buffer
console.log('Length:', res.body.length); // Byte length

// Binary response (Browser)
const res = await request.get('https://api.example.com/image.png')
  .responseType('blob');
console.log('Blob:', res.body);          // Blob
console.log('Size:', res.body.size);     // Byte size

// Form-urlencoded response
const res = await request.get('https://api.example.com/data');
console.log('Parsed:', res.body);        // { key: 'value', ... }
console.log('Raw:', res.text);           // 'key=value&...'
```

### Response Headers

Access response headers with case-insensitive lookup.

```javascript { .api }
interface Response {
  /** Response headers object (lowercase keys) */
  header: object;

  /** Alias for header */
  headers: object;

  /**
   * Get response header value (case-insensitive)
   * @param field - Header name
   * @returns Header value or undefined
   */
  get(field: string): string | undefined;
}
```

**Usage Examples:**

```javascript
const res = await request.get('https://api.example.com/users');

// Access headers object
console.log('All headers:', res.headers);
console.log('Content-Type:', res.headers['content-type']);

// Get specific header (case-insensitive)
console.log('Content-Type:', res.get('Content-Type'));
console.log('Content-Type:', res.get('content-type')); // Same
console.log('ETag:', res.get('etag'));

// Common headers
console.log('Content-Length:', res.get('content-length'));
console.log('Cache-Control:', res.get('cache-control'));
console.log('Last-Modified:', res.get('last-modified'));

// Custom headers
console.log('X-RateLimit-Remaining:', res.get('x-ratelimit-remaining'));
console.log('X-Request-ID:', res.get('x-request-id'));
```

### Content Type

Access content type information from response headers.

```javascript { .api }
interface Response {
  /** Content-Type without parameters (e.g., 'application/json') */
  type: string;

  /** Character encoding from Content-Type (e.g., 'utf-8') */
  charset: string;
}
```

**Usage Examples:**

```javascript
const res = await request.get('https://api.example.com/users');

console.log('Type:', res.type);         // 'application/json'
console.log('Charset:', res.charset);   // 'utf-8'

// Full Content-Type header
console.log('Full:', res.get('content-type')); // 'application/json; charset=utf-8'

// Type checking
if (res.type === 'application/json') {
  console.log('JSON response:', res.body);
}

if (res.type.startsWith('text/')) {
  console.log('Text response:', res.text);
}

if (res.type.startsWith('image/')) {
  console.log('Image response');
}
```

### Link Headers

Parsed Link header for pagination and related resources.

```javascript { .api }
interface Response {
  /** Parsed Link header as object with rel as keys */
  links: object;
}
```

**Usage Examples:**

```javascript
// API response with Link header:
// Link: <https://api.example.com/users?page=2>; rel="next",
//       <https://api.example.com/users?page=5>; rel="last"

const res = await request.get('https://api.example.com/users?page=1');

console.log('Links:', res.links);
// {
//   next: 'https://api.example.com/users?page=2',
//   last: 'https://api.example.com/users?page=5'
// }

// Pagination
if (res.links.next) {
  const nextPage = await request.get(res.links.next);
  console.log('Next page:', nextPage.body);
}

// Common link relations
console.log('Previous:', res.links.prev);
console.log('First:', res.links.first);
console.log('Last:', res.links.last);
console.log('Self:', res.links.self);
```

### Redirects

List of URLs in the redirect chain.

```javascript { .api }
interface Response {
  /** Array of redirect URLs (Node.js only) */
  redirects: string[];
}
```

**Usage Examples:**

```javascript
// Node.js only
const res = await request.get('https://api.example.com/redirect');

console.log('Final URL:', res.req.url);
console.log('Redirect chain:', res.redirects);
// ['https://api.example.com/moved', 'https://api.example.com/final']

// Check if request was redirected
if (res.redirects && res.redirects.length > 0) {
  console.log('Request was redirected', res.redirects.length, 'times');
}
```

### Additional Response Properties

Additional properties available on Response objects.

```javascript { .api }
interface Response {
  /** Original Node.js request object */
  req: object;

  /** SuperAgent Request instance that created this response */
  request: Request;

  /** XMLHttpRequest object (browser only) */
  xhr: XMLHttpRequest;

  /** Uploaded files in multipart response (Node.js only) */
  files: object;

  /** Whether response was buffered */
  buffered: boolean;

  /** HTTP status text (e.g., 'OK', 'Not Found') */
  statusText: string;
}
```

**Usage Examples:**

```javascript
const res = await request.get('https://api.example.com/users');

// Access original request
console.log('Request method:', res.request.method);
console.log('Request URL:', res.request.url);

// Status text
console.log('Status text:', res.statusText); // 'OK'

// Check if buffered
console.log('Buffered:', res.buffered); // true

// Browser: Access XMLHttpRequest
if (res.xhr) {
  console.log('XHR object:', res.xhr);
}

// Node.js: Access files from multipart response
if (res.files) {
  console.log('Uploaded files:', res.files);
}
```

### Response Utilities

Utility methods for response objects.

```javascript { .api }
/**
 * Convert response to an Error object
 * @returns Error object with response details
 */
Response.prototype.toError(): Error;

/**
 * Serialize response to plain object
 * @returns Object with status, headers, and body
 */
Response.prototype.toJSON(): object;
```

**Usage Examples:**

```javascript
// Convert response to error
const res = await request.get('https://api.example.com/users')
  .ok(res => res.status === 200); // Custom validation

if (!res.ok) {
  const err = res.toError();
  console.error('Error:', err.message);
  console.error('Status:', err.status);
}

// Serialize response
const res = await request.get('https://api.example.com/users');
const json = res.toJSON();
console.log(json);
// {
//   status: 200,
//   header: { ... },
//   body: [ ... ],
//   text: '[ ... ]'
// }

// Log response for debugging
console.log('Response:', JSON.stringify(res.toJSON(), null, 2));
```

## Response Patterns

### Conditional Processing

```javascript
const res = await request.get('https://api.example.com/users');

if (res.ok) {
  // Process successful response
  console.log('Users:', res.body);
} else if (res.notFound) {
  console.log('No users found');
} else if (res.serverError) {
  console.log('Server error, please try again');
}
```

### Extract Specific Data

```javascript
async function getUsername(userId) {
  const res = await request.get(`https://api.example.com/users/${userId}`);
  return res.body.name;
}

async function getHeaderValue(url, headerName) {
  const res = await request.head(url);
  return res.get(headerName);
}
```

### Response Validation

```javascript
async function fetchValidated(url) {
  const res = await request.get(url);

  if (!res.ok) {
    throw new Error(`HTTP ${res.status}: ${res.statusText}`);
  }

  if (res.type !== 'application/json') {
    throw new Error(`Unexpected content type: ${res.type}`);
  }

  return res.body;
}
```

### Pagination Handling

```javascript
async function fetchAllPages(url) {
  const allResults = [];
  let currentUrl = url;

  while (currentUrl) {
    const res = await request.get(currentUrl);
    allResults.push(...res.body);

    // Check for next page link
    currentUrl = res.links.next || null;
  }

  return allResults;
}
```

### Rate Limit Handling

```javascript
async function fetchWithRateLimit(url) {
  const res = await request.get(url);

  const remaining = res.get('x-ratelimit-remaining');
  const reset = res.get('x-ratelimit-reset');

  console.log(`Rate limit: ${remaining} requests remaining`);
  console.log(`Resets at: ${new Date(reset * 1000)}`);

  if (remaining < 10) {
    console.warn('Approaching rate limit!');
  }

  return res.body;
}
```
