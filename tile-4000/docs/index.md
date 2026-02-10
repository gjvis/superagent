# SuperAgent

SuperAgent is a progressive HTTP client library that provides a fluent, chainable API for making HTTP requests in both browser and Node.js environments. It features a unified interface for AJAX requests and server-side HTTP calls, with built-in support for promises, streams, file uploads, authentication, and automatic response parsing.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript (ES6+)
- **Installation**: `npm install superagent`
- **Environments**: Node.js (6+) and modern browsers

## Core Imports

```javascript
const request = require('superagent');
```

ES6 modules:

```javascript
import request from 'superagent';
```

Browser (via script tag with browserify/webpack):

```javascript
// Available as global 'superagent'
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
  .set('Authorization', 'Bearer token123')
  .then(res => {
    console.log('Created:', res.body);
  });

// Async/await
async function fetchUser(id) {
  const res = await request.get(`https://api.example.com/users/${id}`);
  return res.body;
}

// Legacy callback style
request
  .get('https://api.example.com/users')
  .end((err, res) => {
    if (err) return console.error(err);
    console.log(res.body);
  });
```

## Architecture

SuperAgent provides a dual-build architecture with:

- **Unified API**: Same interface across Node.js and browser environments
- **Request Factory**: Main `request()` function creates chainable Request instances
- **HTTP Method Helpers**: Convenience methods (`.get()`, `.post()`, etc.)
- **Agent System**: Manages request defaults and cookie persistence
- **Plugin Support**: Extensible via middleware functions
- **Promise Integration**: Native promise support with `.then()` and `.catch()`
- **Auto-Parsing**: Automatic content-type detection and response parsing
- **Retry Logic**: Built-in retry mechanism for failed requests

## Capabilities

### Request Creation

Factory function and HTTP method helpers for creating requests.

```javascript { .api }
function request(method: string, url: string): Request;
function request(url: string): Request;
function request(url: string, callback: (err: Error, res: Response) => void): Request;

function request.get(url: string, data?: object, callback?: Function): Request;
function request.post(url: string, data?: object, callback?: Function): Request;
function request.put(url: string, data?: object, callback?: Function): Request;
function request.patch(url: string, data?: object, callback?: Function): Request;
function request.head(url: string, data?: object, callback?: Function): Request;
function request.delete(url: string, data?: object, callback?: Function): Request;
function request.del(url: string, data?: object, callback?: Function): Request;
function request.options(url: string, data?: object, callback?: Function): Request;
```

[Request Creation](./request-creation.md)

### Request Configuration

Configure request headers, query parameters, body content, and other request options.

```javascript { .api }
// Headers
Request.prototype.set(field: string, value: string): Request;
Request.prototype.set(headers: object): Request;
Request.prototype.get(field: string): string;
Request.prototype.unset(field: string): Request;
Request.prototype.type(contentType: string): Request;
Request.prototype.accept(acceptType: string): Request;

// Query parameters
Request.prototype.query(params: object | string): Request;
Request.prototype.sortQuery(compareFn?: Function): Request;

// Request body
Request.prototype.send(data: object | string | Buffer): Request;
Request.prototype.field(name: string, value: any): Request;
Request.prototype.field(fields: object): Request;
```

[Request Configuration](./request-configuration.md)

### File Uploads

Multipart form data and file attachment support (Node.js and browser).

```javascript { .api }
// Node.js
Request.prototype.attach(field: string, file: string | Buffer | ReadStream, options?: object | string): Request;

// Browser
Request.prototype.attach(field: string, file: Blob | File, filename?: string): Request;
```

[File Uploads](./file-uploads.md)

### Authentication

HTTP authentication methods including Basic, Bearer, and auto authentication.

```javascript { .api }
Request.prototype.auth(user: string, pass: string, options?: object): Request;

// Node.js SSL/TLS
Request.prototype.ca(cert: string | Buffer | Array): Request;
Request.prototype.cert(cert: string | Buffer): Request;
Request.prototype.key(key: string | Buffer): Request;
Request.prototype.pfx(pfx: string | Buffer | object): Request;
```

[Authentication](./authentication.md)

### Request Execution

Execute requests using callbacks, promises, or async/await.

```javascript { .api }
Request.prototype.end(callback?: (err: Error, res: Response) => void): Request;
Request.prototype.then(resolve: (res: Response) => any, reject?: (err: Error) => any): Promise;
Request.prototype.catch(reject: (err: Error) => any): Promise;
```

[Request Execution](./request-execution.md)

### Request Control

Control request behavior including timeouts, retries, redirects, and aborting.

```javascript { .api }
Request.prototype.timeout(ms: number): Request;
Request.prototype.timeout(options: { response?: number, deadline?: number }): Request;
Request.prototype.retry(count?: number, callback?: Function): Request;
Request.prototype.abort(): Request;
Request.prototype.redirects(count: number): Request;
Request.prototype.maxResponseSize(bytes: number): Request;
```

[Request Control](./request-control.md)

### Response Handling

Access and interpret HTTP responses including status, headers, and body.

```javascript { .api }
interface Response {
  // Status
  status: number;
  statusCode: number;
  statusType: number;
  ok: boolean;
  error: Error | false;
  clientError: boolean;
  serverError: boolean;

  // Body
  body: any;
  text: string;
  type: string;
  charset: string;

  // Headers
  header: object;
  headers: object;
  get(field: string): string;

  // Methods
  toError(): Error;
  toJSON(): object;
}
```

[Response Handling](./response-handling.md)

### Agent

Manage request defaults and cookie persistence across multiple requests.

```javascript { .api }
function request.agent(options?: object): Agent;

interface Agent {
  // Configuration methods (set defaults for all requests)
  set(field: string, value: string): Agent;
  query(params: object): Agent;
  type(contentType: string): Agent;
  accept(acceptType: string): Agent;
  timeout(ms: number): Agent;
  retry(count?: number): Agent;

  // HTTP methods
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;

  // Node.js only
  jar: CookieJar;
}
```

[Agent](./agent.md)

### Advanced Configuration

Advanced features including custom parsers, serializers, plugins, and response type control.

```javascript { .api }
Request.prototype.parse(parser: Function): Request;
Request.prototype.serialize(serializer: Function): Request;
Request.prototype.responseType(type: string): Request;
Request.prototype.use(plugin: Function): Request;
Request.prototype.ok(validator: (res: Response) => boolean): Request;
```

[Advanced Configuration](./advanced-configuration.md)

### Node.js Specific Features

Node.js-only capabilities including HTTP/2, streams, and buffer control.

```javascript { .api }
Request.prototype.http2(enable?: boolean): Request;
Request.prototype.agent(agent: http.Agent | https.Agent): Request;
Request.prototype.buffer(enable?: boolean): Request;
Request.prototype.pipe(stream: WritableStream, options?: object): Request;
Request.prototype.write(data: string | Buffer, encoding?: string): boolean;
```

[Node.js Features](./nodejs-features.md)

### Browser Specific Features

Browser-only capabilities including CORS credentials and XHR access.

```javascript { .api }
Request.prototype.withCredentials(enable?: boolean): Request;
function request.getXHR(): XMLHttpRequest;
```

[Browser Features](./browser-features.md)

### Events and Progress Tracking

Request lifecycle events and upload/download progress monitoring.

```javascript { .api }
Request.prototype.on(event: 'request' | 'response' | 'error' | 'abort' | 'redirect' | 'drain' | 'end' | 'progress', listener: Function): Request;

interface ProgressEvent {
  direction: 'upload' | 'download';
  percent: number;
  loaded: number;
  total: number;
  lengthComputable: boolean;
}
```

[Events and Progress](./events-and-progress.md)

### Global Configuration

Global serializers, parsers, buffering, and protocol handlers.

```javascript { .api }
request.serialize: { [contentType: string]: (data: any) => string };
request.parse: { [contentType: string]: (res: Response, callback: Function) => void };
request.buffer: { [contentType: string]: boolean };
request.types: { [shortcut: string]: string };
request.protocols: { 'http:': typeof http; 'https:': typeof https; 'http2:': typeof http2 };
```

[Global Configuration](./global-configuration.md)

## Default Behavior

### Content-Type Handling

SuperAgent automatically sets Content-Type headers based on the data sent:
- Objects/arrays: `application/json`
- Strings: `application/x-www-form-urlencoded` (for `.query()`) or `text/plain` (for `.send()`)
- Buffers/Blobs: `application/octet-stream`
- Using `.field()` or `.attach()`: `multipart/form-data`

### Response Parsing

Responses are automatically parsed based on Content-Type:
- `application/json`: Parsed as JSON into `.body`
- `application/x-www-form-urlencoded`: Parsed into `.body` object
- `text/*`: Available as `.text` string
- `image/*`, `application/octet-stream`: Binary data in `.body` (Buffer in Node.js, Blob in browser)

### Error Handling

Errors are thrown/rejected for:
- Network errors (connection failed, timeout, etc.)
- HTTP 4xx and 5xx status codes (unless custom `.ok()` validator is set)

### Retries

By default, requests are NOT retried. When enabled with `.retry()`, requests are retried on:
- Network errors (ECONNRESET, ETIMEDOUT, EADDRINFO, ESOCKETTIMEDOUT)
- HTTP 5xx status codes (except 501)
- Timeout errors

## Types

```javascript { .api }
interface Request {
  method: string;
  url: string;
  header: object;
  cookies: string;
}

interface Response {
  status: number;
  statusCode: number;
  statusType: number;
  ok: boolean;
  error: Error | false;
  clientError: boolean;
  serverError: boolean;
  redirect: boolean;
  info: boolean;
  created: boolean;
  accepted: boolean;
  noContent: boolean;
  badRequest: boolean;
  unauthorized: boolean;
  forbidden: boolean;
  notAcceptable: boolean;
  notFound: boolean;
  unprocessableEntity: boolean;
  body: any;
  text: string;
  type: string;
  charset: string;
  header: object;
  headers: object;
  links: object;
  redirects: string[];
}

interface Agent {
  jar: CookieJar;  // Node.js only
}

interface TimeoutOptions {
  response?: number;
  deadline?: number;
}

interface AuthOptions {
  type?: 'basic' | 'bearer' | 'auto';
}

interface AttachOptions {
  filename?: string;
  contentType?: string;
}

interface AgentOptions {
  ca?: string | Buffer | Array;
  key?: string | Buffer;
  pfx?: string | Buffer | object;
  cert?: string | Buffer;
}
```
