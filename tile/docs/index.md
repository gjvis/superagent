# SuperAgent

SuperAgent is a progressive HTTP client library that provides a unified, fluent API for making HTTP requests in both browser and Node.js environments. It offers chainable method calls for building requests, supports modern promise-based workflows with `.then()` and `async/await` alongside traditional `.end()` callbacks, and handles various content types including JSON, form data, and multipart uploads. The library features automatic serialization/deserialization, cookie jar support (Node.js), built-in compression handling, timeout and retry capabilities, and a robust plugin system for extending functionality.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install superagent`
- **Repository**: https://github.com/visionmedia/superagent
- **Documentation**: https://visionmedia.github.io/superagent/

## Core Imports

ESM (ES6 Modules):

```javascript
import request from 'superagent';
```

CommonJS (Node.js):

```javascript
const request = require('superagent');
```

Named imports for advanced usage:

```javascript
// ESM
import request, { agent, serialize, parse } from 'superagent';

// CommonJS
const request = require('superagent');
const { agent } = require('superagent');
```

## Basic Usage

```javascript
// Simple GET request with promises
const res = await request.get('/api/users');
console.log(res.body); // parsed response body

// POST request with JSON body
await request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' })
  .set('Authorization', 'Bearer token123');

// Using callbacks (legacy)
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error(err);
    } else {
      console.log(res.body);
    }
  });

// Query parameters
const res = await request
  .get('/api/search')
  .query({ q: 'nodejs', limit: 10 });
```

## Architecture

SuperAgent is built around several key architectural components:

- **Dual Environment Support**: Unified API works in both Node.js and browser environments with platform-specific implementations automatically selected via package.json browser field
- **Fluent Interface**: Chainable method calls allow building complex requests with readable, declarative syntax
- **Request/Response Classes**: Separate classes for request configuration and response handling, both enhanced with mixins for shared functionality
- **Agent System**: Optional agent instances persist settings and cookies across multiple requests
- **Plugin Architecture**: `.use()` method enables middleware-style request modification and extension
- **Promise Integration**: Native promise support via `.then()` and `.catch()` for modern async/await workflows
- **Automatic Content Handling**: Intelligent serialization and parsing based on Content-Type headers

## Capabilities

### Making HTTP Requests

Core HTTP request methods for GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS requests with fluent chainable API.

```javascript { .api }
/**
 * Create an HTTP request
 * @param method - HTTP method (GET, POST, etc.)
 * @param url - URL string
 * @returns Request instance
 */
function request(method: string, url: string): Request;

// HTTP verb convenience methods
function get(url: string, data?: any, callback?: Function): Request;
function post(url: string, data?: any, callback?: Function): Request;
function put(url: string, data?: any, callback?: Function): Request;
function patch(url: string, data?: any, callback?: Function): Request;
function del(url: string, data?: any, callback?: Function): Request;
function delete(url: string, data?: any, callback?: Function): Request;
function head(url: string, data?: any, callback?: Function): Request;
function options(url: string, data?: any, callback?: Function): Request;
```

[Making HTTP Requests](./making-requests.md)

### Request Configuration

Configure request headers, query parameters, content types, and other request options using chainable methods.

```javascript { .api }
// Header management
set(field: string, value: string): Request;
set(fields: object): Request;
get(field: string): string;
unset(field: string): Request;
type(type: string): Request;
accept(type: string): Request;

// Query parameters
query(params: object | string): Request;
sortQuery(comparator?: Function): Request;

// Request body
send(data: any): Request;
```

[Request Configuration](./request-configuration.md)

### Response Handling

Access response data, status codes, headers, and handle response parsing with automatic content-type detection.

```javascript { .api }
interface Response {
  // Status information
  status: number;
  statusCode: number;
  statusType: number;
  ok: boolean;
  error: Error | false;
  clientError: boolean;
  serverError: boolean;

  // Content
  body: any;
  text: string;
  type: string;
  charset: string;
  header: object;
  headers: object;

  // Methods
  get(field: string): string;
}
```

[Response Handling](./response-handling.md)

### Authentication

Support for basic auth, bearer tokens, and custom authentication headers with flexible configuration options.

```javascript { .api }
/**
 * Set authorization credentials
 * @param user - Username or token
 * @param pass - Password (optional for bearer auth)
 * @param options - Auth options with type: 'basic', 'bearer', or 'auto'
 */
auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Request;
```

[Authentication](./authentication.md)

### File Uploads and Form Data

Upload files and send multipart/form-data with support for file attachments, form fields, and progress tracking.

```javascript { .api }
/**
 * Attach file for upload (Node.js only)
 * @param field - Form field name
 * @param file - File path, Buffer, or ReadStream
 * @param options - Options or filename string
 */
attach(field: string, file: string | Buffer | ReadStream, options?: string | object): Request;

/**
 * Add form field
 * @param name - Field name
 * @param value - Field value
 */
field(name: string, value: string | Blob | File | Buffer): Request;
field(fields: object): Request;
```

[File Uploads and Form Data](./file-uploads.md)

### Timeouts and Retries

Configure request timeouts, automatic retry logic, and handle timeout errors with granular control over timing behavior.

```javascript { .api }
/**
 * Set timeout for request
 * @param ms - Timeout in milliseconds, or object with response and deadline
 */
timeout(ms: number | { response?: number; deadline?: number }): Request;

/**
 * Enable automatic retry on failure
 * @param count - Number of retry attempts
 * @param callback - Optional callback to determine if retry should occur
 */
retry(count?: number, callback?: Function): Request;

/**
 * Abort the current request
 */
abort(): Request;
```

[Timeouts and Retries](./timeouts-retries.md)

### Agent and Cookie Management

Create agent instances to persist settings and cookies across multiple requests for session management and default configurations.

```javascript { .api }
/**
 * Create a new agent with persistent cookies (Node.js)
 * @param options - Agent configuration options
 */
function agent(options?: { ca?: any; key?: any; pfx?: any; cert?: any }): Agent;

interface Agent {
  // HTTP methods that create requests with agent defaults
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;
  del(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  options(url: string, callback?: Function): Request;

  // Configuration methods (returns agent for chaining)
  set(field: string, value: string): Agent;
  auth(user: string, pass: string): Agent;
  timeout(ms: number): Agent;
  redirects(n: number): Agent;
  // ... all Request configuration methods available
}
```

[Agent and Cookie Management](./agent-cookies.md)

### Advanced Features

Plugins, custom parsers/serializers, response streaming, HTTP/2 support, and other advanced capabilities for specialized use cases.

```javascript { .api }
/**
 * Apply plugin middleware
 * @param fn - Plugin function that receives the request instance
 */
use(fn: (req: Request) => void): Request;

/**
 * Custom response parser
 * @param fn - Parser function
 */
parse(fn: (res: Response, callback: Function) => void): Request;

/**
 * Custom request serializer
 * @param fn - Serializer function
 */
serialize(fn: (data: any) => string): Request;

/**
 * Enable/disable HTTP/2 (Node.js only)
 * @param enable - Enable HTTP/2 (default true)
 */
http2(enable?: boolean): Request;

/**
 * Pipe response to stream (Node.js only)
 * @param stream - Writable stream
 * @param options - Pipe options
 */
pipe(stream: WritableStream, options?: object): WritableStream;
```

[Advanced Features](./advanced-features.md)

### HTTPS and Certificates

Configure SSL/TLS certificates, custom HTTPS agents, and certificate validation for secure connections (Node.js only).

```javascript { .api }
/**
 * Set CA certificate
 * @param cert - Certificate Buffer or string
 */
ca(cert: Buffer | string): Request;

/**
 * Set client certificate key
 * @param cert - Key Buffer or string
 */
key(cert: Buffer | string): Request;

/**
 * Set client certificate
 * @param cert - Certificate Buffer or string
 */
cert(cert: Buffer | string): Request;

/**
 * Set PFX or PKCS12 certificate
 * @param cert - Certificate Buffer or string or object with pfx and passphrase
 */
pfx(cert: Buffer | string | { pfx: Buffer | string; passphrase: string }): Request;

/**
 * Set custom http.Agent for connection pooling
 * @param agent - Node.js http.Agent instance
 */
agent(agent: any): Request;
```

[HTTPS and Certificates](./https-certificates.md)

## Types

```javascript { .api }
interface Request {
  // Request properties
  method: string;
  url: string;
  header: object;
  cookies: string;

  // Promise interface
  then(resolve: Function, reject?: Function): Promise<Response>;
  catch(reject: Function): Promise<Response>;

  // Callback interface
  end(callback: (err: Error | null, res: Response) => void): void;

  // Configuration (see sub-docs for complete method signatures)
  set(field: string | object, value?: string): Request;
  get(field: string): string;
  unset(field: string): Request;
  query(params: object | string): Request;
  send(data: any): Request;
  type(type: string): Request;
  accept(type: string): Request;
  auth(user: string, pass?: string, options?: object): Request;
  timeout(ms: number | object): Request;
  retry(count?: number, callback?: Function): Request;
  redirects(n: number): Request;
  use(fn: Function): Request;
  ok(fn: Function): Request;
  abort(): Request;

  // ... see sub-docs for complete API
}

interface Response {
  // Status
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

  // Content
  body: any;
  text: string;
  type: string;
  charset: string;
  header: object;
  headers: object;
  files: object;
  redirects: string[];
  links: object;

  // Methods
  get(field: string): string;
}

interface Agent {
  // HTTP verb methods
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;
  del(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  options(url: string, callback?: Function): Request;

  // Configuration methods (same as Request)
  set(...args: any[]): Agent;
  auth(...args: any[]): Agent;
  timeout(...args: any[]): Agent;
  retry(...args: any[]): Agent;
  // ... all Request configuration methods
}
```

## Error Handling

Errors can be handled via callbacks or promise rejection:

```javascript
// Promise-based error handling
try {
  const res = await request.get('/api/users');
  console.log(res.body);
} catch (err) {
  console.error(err.status); // HTTP status code
  console.error(err.response); // Response object
  console.error(err.message); // Error message
}

// Callback-based error handling
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error(err);
      return;
    }
    console.log(res.body);
  });
```

Error objects contain:
- `.status` - HTTP status code (if response received)
- `.response` - Response object (if response received)
- `.message` - Error message
- `.timeout` - Timeout value (for timeout errors)
- `.code` - Error code (e.g., 'ECONNABORTED', 'ETIMEDOUT')

## Events

Request objects emit events that can be listened to:

```javascript
request
  .get('/api/data')
  .on('progress', (event) => {
    console.log('Progress:', event.percent);
  })
  .on('response', (res) => {
    console.log('Got response:', res.status);
  })
  .on('error', (err) => {
    console.error('Error:', err);
  })
  .on('abort', () => {
    console.log('Request aborted');
  })
  .end();
```

Available events:
- `request` - Emitted when request is initiated
- `response` - Emitted when response is received
- `redirect` - Emitted when following a redirect
- `error` - Emitted on request error
- `abort` - Emitted when request is aborted
- `end` - Emitted when request completes
- `progress` - Emitted during upload progress
- `drain` - Emitted when write buffer drains (Node.js only)
