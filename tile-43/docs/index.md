# SuperAgent

SuperAgent is a progressive HTTP client library that provides a fluent, chainable API for making HTTP requests in both browser and Node.js environments. It offers high-level features including method chaining for request configuration, support for JSON/form data/multipart uploads, automatic serialization and parsing of request/response bodies, promise-based and callback-based interfaces, plugin extensibility, and comprehensive error handling.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install superagent`

## Core Imports

```javascript { .api }
const request = require('superagent');
```

ES6/TypeScript:

```javascript { .api }
import request from 'superagent';
```

Browser (via script tag):

```javascript { .api }
// Available as global 'superagent'
const request = window.superagent;
```

## Basic Usage

```javascript
const request = require('superagent');

// Simple GET request with promises
request
  .get('https://api.example.com/users')
  .then(res => {
    console.log(res.status, res.body);
  })
  .catch(err => {
    console.error(err);
  });

// POST request with JSON data
request
  .post('https://api.example.com/users')
  .send({ name: 'Alice', email: 'alice@example.com' })
  .set('Authorization', 'Bearer token123')
  .then(res => {
    console.log('Created user:', res.body);
  });

// Using callbacks instead of promises
request.get('https://api.example.com/users', (err, res) => {
  if (err) {
    console.error(err);
  } else {
    console.log(res.body);
  }
});
```

## Core Concepts

### Request Methods

SuperAgent provides convenience methods for all HTTP verbs:

```javascript { .api }
request.get(url, [data], [callback])
request.post(url, [data], [callback])
request.put(url, [data], [callback])
request.patch(url, [data], [callback])
request.delete(url, [data], [callback])
request.del(url, [data], [callback])  // Alias for delete
request.head(url, [data], [callback])
request.options(url, [data], [callback])
```

For GET and HEAD requests, `data` is used as query parameters. For other methods, `data` is sent as the request body.

### Fluent Chainable API

All request configuration methods return the request instance, allowing for method chaining:

```javascript
request
  .post('/api/users')
  .set('Accept', 'application/json')
  .send({ name: 'Alice' })
  .timeout({ response: 5000 })
  .retry(2)
  .end(callback);
```

### Promise and Callback Support

SuperAgent supports both callback-based and promise-based workflows:

```javascript
// Callback
request.get('/api/users', (err, res) => { /* ... */ });

// Promise
request.get('/api/users').then(res => { /* ... */ }).catch(err => { /* ... */ });

// Async/await
const res = await request.get('/api/users');
```

## Capabilities

### Making HTTP Requests

Create and execute HTTP requests using the main request function or convenience methods.

```javascript { .api }
/**
 * Create a new HTTP request
 * @param method - HTTP method (GET, POST, etc.)
 * @param url - Request URL
 * @returns Request instance for chaining
 */
function request(method: string, url: string): Request;

/**
 * Create GET request (shorthand for url-only parameter)
 * @param url - Request URL
 * @returns Request instance for chaining
 */
function request(url: string): Request;

/**
 * Create GET request with callback
 * @param url - Request URL
 * @param callback - Callback function (err, res) => void
 * @returns Request instance
 */
function request(url: string, callback: (err: Error, res: Response) => void): Request;
```

HTTP verb methods:

```javascript { .api }
function get(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function post(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function put(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function patch(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function delete(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function del(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function head(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
function options(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

### Request Configuration

Configure request headers, query parameters, body data, and content types. Essential for customizing HTTP requests with proper headers and data formatting.

```javascript { .api }
interface Request {
  set(field: string, value: string): Request;
  set(headers: object): Request;
  get(field: string): string;
  unset(field: string): Request;
  query(params: object | string): Request;
  send(data: any): Request;
  type(contentType: string): Request;
  accept(acceptType: string): Request;
}
```

[Request Configuration](./request-configuration.md)

### Advanced Request Options

Configure authentication, SSL/TLS certificates, timeouts, retries, and redirect behavior. Critical for production use cases requiring security, reliability, and error handling.

```javascript { .api }
interface Request {
  auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Request;
  timeout(ms: number): Request;
  timeout(options: { response?: number, deadline?: number }): Request;
  retry(count?: number, callback?: (err: Error, res: Response) => boolean): Request;
  redirects(count: number): Request;
  ca(cert: Buffer | string): Request;  // Node.js only
  key(cert: Buffer | string): Request;  // Node.js only
  cert(cert: Buffer | string): Request;  // Node.js only
  pfx(cert: Buffer | string | { pfx: Buffer | string, passphrase: string }): Request;  // Node.js only
}
```

[Advanced Request Options](./request-advanced.md)

### File Uploads

Upload files using multipart/form-data in Node.js. Supports file paths, Buffers, and Streams with automatic content-type detection.

```javascript { .api }
interface Request {
  attach(field: string, file: string | Buffer | Stream, options?: string | { filename?: string }): Request;
  field(name: string, value: string | Blob | File | Buffer): Request;
  field(fields: object): Request;
}
```

[File Uploads](./file-upload.md)

### Response Handling

Access response status, headers, and body data with convenient status code checking properties and automatic parsing.

```javascript { .api }
interface Response {
  status: number;
  statusCode: number;
  statusType: number;  // 1-5
  body: any;
  text: string;
  header: object;
  headers: object;
  type: string;
  charset: string;

  // Status checks
  ok: boolean;  // 2xx
  error: Error | false;  // 4xx or 5xx
  clientError: boolean;  // 4xx
  serverError: boolean;  // 5xx
  redirect: boolean;  // 3xx
  info: boolean;  // 1xx

  // Common status codes
  created: boolean;  // 201
  accepted: boolean;  // 202
  noContent: boolean;  // 204
  badRequest: boolean;  // 400
  unauthorized: boolean;  // 401
  forbidden: boolean;  // 403
  notFound: boolean;  // 404
  notAcceptable: boolean;  // 406
  unprocessableEntity: boolean;  // 422

  get(field: string): string;
}
```

[Response Handling](./response-handling.md)

### Agent with Cookie Persistence

Create agent instances to persist configuration and cookies across multiple requests. Useful for maintaining sessions and applying common settings to request groups.

```javascript { .api }
interface Agent {
  get(url: string, callback?: (err: Error, res: Response) => void): Request;
  post(url: string, callback?: (err: Error, res: Response) => void): Request;
  put(url: string, callback?: (err: Error, res: Response) => void): Request;
  patch(url: string, callback?: (err: Error, res: Response) => void): Request;
  delete(url: string, callback?: (err: Error, res: Response) => void): Request;
  del(url: string, callback?: (err: Error, res: Response) => void): Request;
  head(url: string, callback?: (err: Error, res: Response) => void): Request;
  options(url: string, callback?: (err: Error, res: Response) => void): Request;

  // Default configuration methods
  set(field: string, value: string): Agent;
  auth(user: string, pass: string, options?: object): Agent;
  timeout(ms: number): Agent;
  retry(count: number): Agent;
  // ... all Request configuration methods available

  jar: CookieJar;  // Node.js only
}

function agent(options?: { ca?: any, key?: any, pfx?: any, cert?: any }): Agent;
```

[Agent](./agent.md)

### Streaming and Piping

Stream request and response data in Node.js for memory-efficient handling of large payloads.

```javascript { .api }
interface Request {
  pipe(destination: Stream, options?: object): Stream;  // Node.js only
  write(data: string | Buffer, encoding?: string): boolean;  // Node.js only
}
```

[Streaming](./streaming.md)

### Request Execution

Execute requests using callbacks or promises. All requests must be explicitly executed via `.end()`, `.then()`, or by providing a callback to the verb method.

```javascript { .api }
interface Request {
  end(callback?: (err: Error | null, res: Response) => void): Request;
  then(resolve: (res: Response) => void, reject?: (err: Error) => void): Promise<Response>;
  catch(reject: (err: Error) => void): Promise<Response>;
}
```

### Response Parsing and Serialization

Customize how request bodies are serialized and response bodies are parsed, or use built-in parsers for common content types.

```javascript { .api }
interface Request {
  parse(parser: (res: any, callback: (err: Error | null, body: any, files?: any) => void) => void): Request;
  serialize(serializer: (data: any) => string): Request;
  buffer(enable?: boolean): Request;
  responseType(type: string): Request;  // 'blob', 'arraybuffer', etc.
}
```

[Global Configuration](./configuration.md)

### Events

SuperAgent requests are EventEmitters, allowing you to listen for various events during the request lifecycle.

```javascript { .api }
interface Request {
  /**
   * Listen for event
   * @param event - Event name
   * @param handler - Event handler function
   * @returns Request instance for chaining
   */
  on(event: string, handler: Function): Request;

  /**
   * Listen for event once
   * @param event - Event name
   * @param handler - Event handler function
   * @returns Request instance for chaining
   */
  once(event: string, handler: Function): Request;
}
```

**Available events:**

- `'request'` - Emitted when request is initiated (passes Request instance)
- `'response'` - Emitted when response is received (passes Response instance)
- `'error'` - Emitted on error (passes Error object)
- `'redirect'` - Emitted on redirect (passes Response instance)
- `'abort'` - Emitted when request is aborted
- `'progress'` - Emitted during upload/download (passes `{ direction: 'upload' | 'download', loaded: number, total: number, percent?: number }`)
- `'end'` - Emitted when request completes
- `'drain'` - Emitted when request buffer is drained (Node.js only)

**Usage Examples:**

```javascript
// Listen for response
request
  .get('/api/users')
  .on('response', res => {
    console.log('Response received:', res.status);
  })
  .on('error', err => {
    console.error('Error:', err.message);
  });

// Monitor progress
request
  .post('/api/upload')
  .attach('file', 'large-file.zip')
  .on('progress', event => {
    if (event.direction === 'upload') {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      console.log('Upload progress:', percent + '%');
    }
  })
  .then(res => {
    console.log('Upload complete');
  });

// Listen for redirects
request
  .get('/api/redirect-test')
  .on('redirect', res => {
    console.log('Redirecting to:', res.headers.location);
  })
  .then(res => {
    console.log('Final destination');
  });
```

### Utility Methods

Additional request utilities for plugins, custom validation, aborting, and conversion.

```javascript { .api }
interface Request {
  use(plugin: (req: Request) => void): Request;
  ok(validator: (res: Response) => boolean): Request;
  abort(): Request;
  clearTimeout(): Request;
  sortQuery(compareFn?: (a: string, b: string) => number): Request;
  maxResponseSize(bytes: number): Request;
  toJSON(): { method: string, url: string, data: any, headers: object };
  withCredentials(enable?: boolean): Request;
}
```

## Error Handling

SuperAgent errors include additional properties for debugging:

```javascript
request.get('/api/endpoint')
  .end((err, res) => {
    if (err) {
      console.log(err.status);      // HTTP status code (if available)
      console.log(err.response);    // Response object (if available)
      console.log(err.retries);     // Number of retries attempted
      console.log(err.timeout);     // Timeout value (for timeout errors)
      console.log(err.code);        // Error code ('ECONNABORTED', etc.)
    }
  });
```

## Browser vs Node.js Differences

Most of the API is identical across environments, but some features are platform-specific:

**Node.js only:**
- File uploads (`.attach()`, `.field()`)
- SSL/TLS certificate methods (`.ca()`, `.key()`, `.cert()`, `.pfx()`)
- HTTP agent configuration (`.agent()`)
- HTTP/2 support (`.http2()`)
- Streaming (`.pipe()`, `.write()`)
- Cookie jar in Agent (`.jar`)
- Response `.files` property

**Browser only:**
- `request.getXHR()` - Get XMLHttpRequest constructor
- `request.types` - MIME type shortcuts
- `request.serializeObject()` - Serialize objects to query strings
- `request.parseString()` - Parse URL-encoded strings

## Exported Classes and Objects

```javascript { .api }
const request: {
  // Main function
  (method: string, url: string): Request;
  (url: string): Request;

  // HTTP verbs
  get: (url: string, data?: any, callback?: Function) => Request;
  post: (url: string, data?: any, callback?: Function) => Request;
  put: (url: string, data?: any, callback?: Function) => Request;
  patch: (url: string, data?: any, callback?: Function) => Request;
  delete: (url: string, data?: any, callback?: Function) => Request;
  del: (url: string, data?: any, callback?: Function) => Request;
  head: (url: string, data?: any, callback?: Function) => Request;
  options: (url: string, data?: any, callback?: Function) => Request;

  // Classes
  Request: typeof Request;
  Response: typeof Response;

  // Agent
  agent: (options?: object) => Agent;

  // Configuration
  serialize: { [contentType: string]: (data: any) => string };
  parse: { [contentType: string]: (res: any, callback: Function) => void };
  buffer: { [contentType: string]: boolean };
  protocols: { 'http:': any, 'https:': any, 'http2:': any };

  // Browser only
  types?: { html: string, json: string, xml: string, urlencoded: string, form: string, 'form-data': string };
  getXHR?: () => XMLHttpRequest;
  serializeObject?: (obj: object) => string;
  parseString?: (str: string) => object;
};
```
