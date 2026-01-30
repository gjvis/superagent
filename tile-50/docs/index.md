# SuperAgent

SuperAgent is a progressive, universal HTTP request library that provides a fluent, chainable API for making HTTP requests in both browser and Node.js environments. It offers elegant request/response handling with support for promises and callbacks, includes built-in JSON parsing and form data serialization, supports file uploads and downloads, provides request/response middleware through plugins, handles cookies and authentication, and supports query string building and custom header management.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install superagent`

## Core Imports

```javascript
const request = require('superagent');
```

For ES modules:

```javascript
import request from 'superagent';
```

## Basic Usage

```javascript
const request = require('superagent');

// Simple GET request with promise
request
  .get('https://api.example.com/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });

// POST request with JSON body
request
  .post('https://api.example.com/users')
  .send({ name: 'John', email: 'john@example.com' })
  .set('Accept', 'application/json')
  .then(res => {
    console.log('User created:', res.body);
  });

// Using async/await
async function getUser(id) {
  const res = await request.get(`https://api.example.com/users/${id}`);
  return res.body;
}

// Using callback style
request
  .get('https://api.example.com/users')
  .end((err, res) => {
    if (err) {
      console.error(err);
    } else {
      console.log(res.body);
    }
  });
```

## Architecture

SuperAgent is built around several key components:

- **Request Factory**: Main `request()` function and HTTP verb shortcuts (`get`, `post`, `put`, etc.)
- **Fluent Interface**: Chainable methods for building requests declaratively
- **Dual Platform**: Separate implementations for Node.js (using http/https modules) and browser (using XMLHttpRequest)
- **Shared API**: RequestBase and ResponseBase mixins provide consistent interface across platforms
- **Agent System**: Persistent cookie handling and default request settings
- **Plugin System**: Extensibility through `.use()` method
- **Promise Support**: Native promise integration alongside callback support

## Capabilities

### Making HTTP Requests

Core request creation and execution using HTTP verb methods with fluent API for building requests.

```javascript { .api }
/**
 * Main request factory function
 * @param method - HTTP method (GET, POST, etc.)
 * @param url - Target URL
 * @returns Request instance
 */
function request(method: string, url: string): Request;

/**
 * HTTP verb shorthand methods
 */
request.get(url: string, callback?: Function): Request;
request.post(url: string, data?: any, callback?: Function): Request;
request.put(url: string, data?: any, callback?: Function): Request;
request.patch(url: string, data?: any, callback?: Function): Request;
request.delete(url: string, callback?: Function): Request;
request.del(url: string, callback?: Function): Request;
request.head(url: string, callback?: Function): Request;
request.options(url: string, callback?: Function): Request;
```

All HTTP methods return chainable Request instances. See [Request Methods](./request-methods.md) for detailed API.

[Request Methods](./request-methods.md)

### Request Execution and Control

Execute requests, handle responses with promises or callbacks, abort requests, configure timeouts and retries.

```javascript { .api }
/**
 * Execute the request
 * @param callback - Optional callback(err, res)
 * @returns Request instance
 */
request.get(url).end(callback?: (err: Error, res: Response) => void): Request;

/**
 * Promise support
 */
request.get(url).then(onFulfilled: (res: Response) => any, onRejected?: (err: Error) => any): Promise<any>;
request.get(url).catch(onRejected: (err: Error) => any): Promise<any>;

/**
 * Abort the request
 */
request.get(url).abort(): Request;

/**
 * Set timeout
 * @param ms - Milliseconds or {response, deadline} object
 */
request.get(url).timeout(ms: number | {response?: number, deadline?: number}): Request;

/**
 * Configure retry behavior
 * @param count - Number of retries
 * @param callback - Optional custom retry logic
 */
request.get(url).retry(count: number, callback?: (err: Error, res: Response) => boolean): Request;
```

[Request Execution](./request-execution.md)

### Request Configuration

Configure headers, request body, query parameters, content types, and authentication.

```javascript { .api }
/**
 * Set request headers
 */
request.get(url).set(field: string, value: string): Request;
request.get(url).set(headers: object): Request;

/**
 * Send request body
 * @param data - Request body (auto-detects JSON/form)
 */
request.post(url).send(data: any): Request;

/**
 * Add query parameters
 */
request.get(url).query(params: string | object): Request;

/**
 * Set Content-Type
 */
request.post(url).type(type: string): Request;

/**
 * Set Accept header
 */
request.get(url).accept(type: string): Request;

/**
 * Set authentication
 * @param user - Username or token
 * @param pass - Password (optional for bearer)
 * @param options - {type: 'basic'|'bearer'|'auto'}
 */
request.get(url).auth(user: string, pass?: string, options?: object): Request;
```

[Request Methods](./request-methods.md)

### Response Handling

Access response data, headers, status codes, and status flags.

```javascript { .api }
interface Response {
  // Response data
  body: any;              // Parsed response body
  text: string;           // Response as text
  header: object;         // Response headers (lowercase keys)
  type: string;           // Content-Type without params

  // Status information
  status: number;         // HTTP status code
  statusCode: number;     // Alias for status
  statusType: number;     // Status class (1-5)

  // Status flags
  ok: boolean;            // True for 2xx
  error: Error | false;   // Error object for 4xx/5xx
  clientError: boolean;   // True for 4xx
  serverError: boolean;   // True for 5xx

  // Specific status checks
  created: boolean;       // 201
  accepted: boolean;      // 202
  noContent: boolean;     // 204
  badRequest: boolean;    // 400
  unauthorized: boolean;  // 401
  forbidden: boolean;     // 403
  notFound: boolean;      // 404

  // Methods
  get(field: string): string;  // Get response header
}
```

[Response](./response.md)

### Agent and Cookies

Persistent agent for maintaining cookies across requests and setting default request configurations.

```javascript { .api }
/**
 * Create agent with persistent cookies (Node.js only)
 * @param options - Optional {ca, key, pfx, cert}
 * @returns Agent instance
 */
request.agent(options?: object): Agent;

interface Agent {
  // HTTP verb methods with cookie persistence
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;

  // Set defaults for all requests
  set(field: string, value: string): Agent;
  auth(user: string, pass: string, options?: object): Agent;
  timeout(ms: number): Agent;
  retry(count: number): Agent;
  // ... all Request configuration methods available
}
```

**Usage:**

```javascript
const agent = request.agent();

// Login and persist cookies
await agent
  .post('/login')
  .send({ username: 'user', password: 'pass' });

// Cookies automatically included in subsequent requests
const profile = await agent.get('/profile');
```

[Agent and Cookies](./agent.md)

### Multipart Forms and File Uploads

Upload files and send multipart form data (Node.js only).

```javascript { .api }
/**
 * Add form field
 */
request.post(url).field(name: string, value: string): Request;
request.post(url).field(fields: object): Request;

/**
 * Attach file
 * @param field - Form field name
 * @param file - File path, Buffer, or Stream
 * @param options - Optional {filename, contentType}
 */
request.post(url).attach(
  field: string,
  file: string | Buffer | Stream,
  options?: {filename?: string, contentType?: string}
): Request;
```

**Usage:**

```javascript
request
  .post('/upload')
  .field('name', 'Image Upload')
  .attach('photo', './image.jpg')
  .then(res => console.log('Uploaded'));
```

[Request Methods](./request-methods.md)

### Streaming (Node.js)

Stream request and response data for large files and real-time processing.

```javascript { .api }
/**
 * Pipe response to writable stream
 * @param stream - Writable stream destination
 * @param options - Stream pipe options
 * @returns Stream
 */
request.get(url).pipe(stream: WritableStream, options?: object): Stream;

/**
 * Write raw data to request
 * @param data - Data to write
 * @param encoding - Optional encoding
 * @returns Backpressure indicator
 */
request.post(url).write(data: string | Buffer, encoding?: string): boolean;
```

**Usage:**

```javascript
// Download file
const fs = require('fs');
request
  .get('https://example.com/large-file.zip')
  .pipe(fs.createWriteStream('file.zip'));

// Upload file
const stream = fs.createReadStream('upload.dat');
stream.pipe(request.post('https://example.com/upload'));
```

[Streaming](./streaming.md)

### TLS/SSL and HTTP/2 (Node.js)

Configure TLS certificates and enable HTTP/2 protocol.

```javascript { .api }
/**
 * Set CA certificate
 */
request.get(url).ca(cert: string | Buffer): Request;

/**
 * Set client private key
 */
request.get(url).key(cert: string | Buffer): Request;

/**
 * Set client certificate
 */
request.get(url).cert(cert: string | Buffer): Request;

/**
 * Set PFX/PKCS12 certificate
 */
request.get(url).pfx(cert: string | Buffer | {pfx: Buffer, passphrase: string}): Request;

/**
 * Enable HTTP/2 protocol
 */
request.get(url).http2(enable?: boolean): Request;
```

[TLS and HTTP/2](./tls-http2.md)

### Events and Plugins

Listen to request lifecycle events and extend functionality with plugins.

```javascript { .api }
/**
 * Apply plugin function
 * @param fn - Plugin function receiving request
 */
request.get(url).use(fn: (req: Request) => void): Request;

/**
 * Event listeners
 */
request.get(url).on(event: string, listener: Function): Request;
```

**Available Events:**
- `request` - Request starts
- `response` - Response received
- `error` - Error occurred
- `redirect` - Redirect followed (Node.js)
- `end` - Request completed
- `abort` - Request aborted
- `progress` - Upload/download progress (Browser)

**Plugin Example:**

```javascript
function timestamp(req) {
  req.set('X-Request-Time', Date.now().toString());
}

request
  .get('/api/data')
  .use(timestamp)
  .then(res => console.log(res.body));
```

[Events and Plugins](./events-plugins.md)

### Advanced Configuration

Custom parsers, serializers, response validation, and buffering control.

```javascript { .api }
/**
 * Set custom response parser
 */
request.get(url).parse(parser: (res: Response, callback: Function) => void): Request;

/**
 * Set custom request serializer
 */
request.post(url).serialize(serializer: (obj: any) => string): Request;

/**
 * Define custom "ok" status check
 */
request.get(url).ok(callback: (res: Response) => boolean): Request;

/**
 * Control response buffering (Node.js)
 */
request.get(url).buffer(enable?: boolean): Request;

/**
 * Set max response size
 */
request.get(url).maxResponseSize(bytes: number): Request;

/**
 * Set max redirects (Node.js)
 */
request.get(url).redirects(count: number): Request;

/**
 * Set response type (Browser)
 */
request.get(url).responseType(type: 'blob' | 'arraybuffer'): Request;
```

[Request Methods](./request-methods.md)

## Platform Differences

### Node.js Only
- File uploads with `.attach()`
- Streaming with `.pipe()` and `.write()`
- TLS/SSL configuration (`.ca()`, `.key()`, `.cert()`, `.pfx()`)
- HTTP/2 support with `.http2()`
- Redirect control with `.redirects()`
- Agent cookie persistence
- Automatic decompression (gzip, deflate, brotli)
- Buffer control with `.buffer()`

### Browser Only
- XHR response types with `.responseType()`
- CORS credentials with `.withCredentials()`
- Progress events for upload/download
- FormData for multipart uploads

### Both Platforms
- All RequestBase methods
- Promise and callback support
- Header management
- Query string handling
- Authentication
- Timeouts and retries
- Plugin system
- Event emitters

## Error Handling

SuperAgent provides detailed error information for failed requests:

```javascript
request
  .get('https://api.example.com/users')
  .then(res => {
    // Success
  })
  .catch(err => {
    console.error(err.message);   // Error message
    console.error(err.status);    // HTTP status code
    console.error(err.response);  // Response object (if available)
  });
```

Common error scenarios:
- Network errors (connection refused, timeout)
- HTTP errors (4xx, 5xx status codes)
- Parse errors (invalid JSON)
- Abort errors (request cancelled)

Automatic retry is available for:
- 5xx server errors (except 501)
- Network timeouts
- Connection errors (ECONNRESET, ETIMEDOUT, etc.)

## Global Configuration

SuperAgent provides global configuration objects and utility functions for customizing behavior.

### Custom Serializers

```javascript { .api }
/**
 * Object mapping MIME types to serializer functions
 * Available in both Node.js and browser
 */
request.serialize: {
  [mimeType: string]: (obj: any) => string
};
```

**Usage Examples:**

```javascript
// Add XML serializer
request.serialize['application/xml'] = function(obj) {
  return convertToXML(obj);
};

// Add CSV serializer
request.serialize['text/csv'] = function(data) {
  return data.map(row => row.join(',')).join('\n');
};

// Use custom serializer
request
  .post('/api/data')
  .type('xml')
  .send({ data: 'value' });
// Automatically uses the XML serializer
```

### Custom Parsers

```javascript { .api }
/**
 * Object mapping MIME types to parser functions
 * Available in both Node.js and browser
 */
request.parse: {
  [mimeType: string]: (res: Response, callback: (err: Error, body: any) => void) => void
};
```

**Usage Examples:**

```javascript
// Add XML parser (Node.js)
request.parse['application/xml'] = function(res, callback) {
  const parseXML = require('xml-parser');
  try {
    const parsed = parseXML(res.text);
    callback(null, parsed);
  } catch (err) {
    callback(err);
  }
};

// Add CSV parser (Node.js)
request.parse['text/csv'] = function(res, callback) {
  const rows = res.text.split('\n').map(line => line.split(','));
  callback(null, rows);
};

// Parser is automatically used
request
  .get('/api/data.xml')
  .then(res => {
    console.log(res.body);  // Parsed XML object
  });
```

### Buffer Configuration (Node.js)

```javascript { .api }
/**
 * Object mapping MIME types to buffering strategy
 * Node.js only
 */
request.buffer: {
  [mimeType: string]: boolean
};
```

**Usage Examples:**

```javascript
// Always buffer PDFs
request.buffer['application/pdf'] = true;

// Never buffer videos
request.buffer['video/mp4'] = false;
request.buffer['video/avi'] = false;

// Check default buffering strategy
console.log(request.buffer['application/json']);  // true
console.log(request.buffer['image/png']);          // false
```

### Type Shortcuts (Browser)

```javascript { .api }
/**
 * Object mapping short names to MIME types
 * Browser only
 */
request.types: {
  [shortName: string]: string
};
```

**Usage Examples:**

```javascript
// Add custom type shortcuts (browser)
request.types.xml = 'application/xml';
request.types.csv = 'text/csv';
request.types.yaml = 'application/x-yaml';

// Use shortcuts with .type() and .accept()
request
  .post('/api/data')
  .type('yaml')
  .accept('yaml');
// Sets Content-Type: application/x-yaml
// Sets Accept: application/x-yaml

// Built-in shortcuts:
// - html → text/html
// - json → application/json
// - xml → application/xml
// - urlencoded → application/x-www-form-urlencoded
// - form → application/x-www-form-urlencoded
// - form-data → application/x-www-form-urlencoded
```

### Protocol Handlers (Node.js)

```javascript { .api }
/**
 * Object mapping protocols to handler modules
 * Node.js only
 */
request.protocols: {
  'http:': typeof http,
  'https:': typeof https,
  'http2:': typeof http2
};
```

**Usage Examples:**

```javascript
// Node.js only
const http = require('http');
const https = require('https');

// Check protocol handlers
console.log(request.protocols['http:']);   // http module
console.log(request.protocols['https:']);  // https module
console.log(request.protocols['http2:']);  // http2 wrapper

// These are used internally by SuperAgent
// Modifying them is not recommended
```

### Utility Functions (Browser)

```javascript { .api }
/**
 * Serialize object to query string (browser only)
 * @param obj - Object to serialize
 * @returns Query string
 */
request.serializeObject(obj: object): string;

/**
 * Parse query string to object (browser only)
 * @param str - Query string to parse
 * @returns Parsed object
 */
request.parseString(str: string): object;

/**
 * Get XMLHttpRequest constructor (browser only)
 * @returns XMLHttpRequest constructor
 */
request.getXHR(): typeof XMLHttpRequest;
```

**Usage Examples:**

```javascript
// Browser only

// Serialize object to query string
const qs = request.serializeObject({ page: 1, limit: 10 });
console.log(qs);  // 'page=1&limit=10'

// Parse query string
const params = request.parseString('page=1&limit=10');
console.log(params);  // { page: '1', limit: '10' }

// Get XHR constructor
const XHR = request.getXHR();
const xhr = new XHR();
```

## Error Object

When requests fail, SuperAgent provides detailed error information through the Error object.

```javascript { .api }
interface Error {
  message: string;          // Error description
  status?: number;          // HTTP status code (if available)
  response?: Response;      // Response object (if available)
  method?: string;          // Request method (GET, POST, etc.)
  url?: string;             // Request URL
  timeout?: number;         // Timeout value (if timeout error)
  code?: string;            // Error code (e.g., 'ECONNABORTED', 'ETIMEDOUT')
  errno?: string;           // System error number (e.g., 'ETIME', 'ETIMEDOUT')
  retries?: number;         // Number of retries attempted
  crossDomain?: boolean;    // Cross-domain error flag (browser)
  parse?: boolean;          // Parser error flag
  original?: Error;         // Original error (if wrapped)
  rawResponse?: string;     // Raw response text (on parse errors)
}
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    // HTTP error (4xx or 5xx)
    if (err.status) {
      console.error('HTTP Error:', err.status);
      console.error('Response body:', err.response?.body);
      console.error('Request:', err.method, err.url);
    }

    // Timeout error
    if (err.timeout) {
      console.error('Request timed out:', err.timeout, 'ms');
      console.error('Error code:', err.code);  // 'ECONNABORTED'
      console.error('Errno:', err.errno);      // 'ETIME' or 'ETIMEDOUT'
    }

    // Network error
    if (err.code === 'ECONNRESET' || err.code === 'ECONNREFUSED') {
      console.error('Network error:', err.code);
    }

    // Parse error
    if (err.parse) {
      console.error('Parse error:', err.message);
      console.error('Raw response:', err.rawResponse);
    }

    // Cross-domain error (browser)
    if (err.crossDomain) {
      console.error('CORS error');
    }

    // Retry information
    if (err.retries) {
      console.error('Failed after', err.retries, 'retries');
    }

    // Original error
    if (err.original) {
      console.error('Original error:', err.original);
    }
  });
```

### Common Error Codes

```javascript
// Network errors
'ECONNRESET'       // Connection reset by peer
'ECONNREFUSED'     // Connection refused
'ETIMEDOUT'        // Connection timed out
'ENOTFOUND'        // DNS lookup failed
'ECONNABORTED'     // Request aborted (timeout or manual)

// Parse errors
err.parse === true // Response parsing failed

// Timeout errors
err.timeout        // Timeout value in milliseconds
err.errno === 'ETIME' || err.errno === 'ETIMEDOUT'

// HTTP errors
err.status >= 400  // Client or server error
```

### Error Handling Patterns

```javascript
// Handle specific error types
async function makeRequest(url) {
  try {
    const res = await request.get(url);
    return res.body;
  } catch (err) {
    if (err.status === 404) {
      console.log('Resource not found');
      return null;
    }

    if (err.status === 401) {
      console.log('Unauthorized, redirecting to login');
      redirectToLogin();
      return;
    }

    if (err.timeout) {
      console.log('Request timed out, retrying...');
      return makeRequest(url);  // Retry
    }

    if (err.status >= 500) {
      console.error('Server error:', err.status);
      throw err;
    }

    // Unknown error
    console.error('Unexpected error:', err);
    throw err;
  }
}

// Extract error details
function getErrorDetails(err) {
  return {
    type: err.timeout ? 'timeout' :
          err.parse ? 'parse' :
          err.crossDomain ? 'cors' :
          err.status ? 'http' : 'network',
    status: err.status,
    code: err.code,
    message: err.message,
    url: err.url,
    method: err.method,
    retries: err.retries
  };
}

request
  .get('/api/data')
  .catch(err => {
    const details = getErrorDetails(err);
    console.error('Error details:', details);
    logToMonitoring(details);
  });
```

## Exported Classes

SuperAgent exports Request and Response class constructors for advanced usage.

```javascript { .api }
/**
 * Request class constructor
 * Advanced: Typically use request() factory instead
 */
request.Request: typeof Request;

/**
 * Response class constructor
 * Advanced: Typically responses are created automatically
 */
request.Response: typeof Response;
```

**Usage Examples:**

```javascript
// Access Request constructor
const Request = request.Request;
console.log(Request);  // [Function: Request]

// Access Response constructor
const Response = request.Response;
console.log(Response);  // [Function: Response]

// Typical usage: Use factory functions instead of constructors
const req = request.get('/api/users');  // Preferred
// Not: const req = new request.Request('GET', '/api/users');

// Check instance types
const req = request.get('/api/users');
console.log(req instanceof request.Request);  // true

request
  .get('/api/users')
  .then(res => {
    console.log(res instanceof request.Response);  // true
  });
```

## TypeScript Support

SuperAgent v4.0.0 does not include built-in TypeScript definitions. Install type definitions separately:

```bash
npm install --save-dev @types/superagent
```
