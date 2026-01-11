# SuperAgent

SuperAgent is a progressive HTTP client library that provides a unified fluent API for making HTTP requests in both browser and Node.js environments. It offers a chainable interface for constructing requests with support for JSON and form data serialization, query string building, file uploads via multipart/form-data, cookie handling, request/response middleware through plugins, promise-based and callback-based patterns, automatic content-type detection and parsing, HTTP method helpers, and comprehensive error handling.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install superagent`

## Core Imports

```javascript
const request = require('superagent');
```

ES6/ESM:

```javascript
import request from 'superagent';
```

Browser (via script tag with browserified version):

```javascript
// superagent is available as global `superagent`
```

## Basic Usage

```javascript
const request = require('superagent');

// Simple GET request
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error(err);
    } else {
      console.log(res.body);
    }
  });

// POST with JSON body
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' })
  .set('accept', 'json')
  .end((err, res) => {
    console.log(res.body);
  });

// Using promises
request
  .get('/api/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });

// Using async/await
async function getUsers() {
  const res = await request.get('/api/users');
  return res.body;
}
```

## Architecture

SuperAgent is built around several key components:

- **Request Builder**: Fluent chainable API for constructing HTTP requests
- **HTTP Method Helpers**: Convenience functions for common HTTP methods (GET, POST, PUT, DELETE, etc.)
- **Agent**: Persistent configuration container that maintains settings and cookies across multiple requests
- **Plugin System**: Middleware-like extensions that can modify requests before they are sent
- **Promise Support**: Native Promise integration alongside traditional callback patterns
- **Automatic Parsing**: Content-type based response parsing with extensible parser registry
- **Retry Logic**: Built-in retry mechanism for failed requests with configurable conditions

## Capabilities

### Making HTTP Requests

Core request building functionality including HTTP method helpers, request configuration, and request execution.

```typescript { .api }
function request(method: string, url: string): Request;
function request(url: string): Request; // Defaults to GET
```

[HTTP Requests](./http-requests.md)

### Request Configuration

Configure request headers, body, query parameters, authentication, and other request options.

```typescript { .api }
interface Request {
  set(field: string, value: string): Request;
  set(headers: object): Request;
  type(type: string): Request;
  accept(type: string): Request;
  send(data: any): Request;
  query(params: object | string): Request;
  auth(user: string, pass?: string, options?: object): Request;
  timeout(ms: number | { response?: number, deadline?: number }): Request;
  retry(count?: number, callback?: Function): Request;
}
```

[Request Configuration](./request-configuration.md)

### File Uploads

Multipart form data support for uploading files with form fields.

```typescript { .api }
interface Request {
  attach(field: string, file: string | Buffer | Blob | File, options?: string | object): Request;
  field(name: string, value: string | Blob): Request;
}
```

[File Uploads](./file-uploads.md)

### Response Handling

Access response data, headers, status, and parse response body based on content type.

```typescript { .api }
interface Response {
  status: number;
  statusCode: number;
  header: object;
  headers: object;
  body: any;
  text: string;
  type: string;
  charset: string;
  ok: boolean;
  error: Error | false;
  clientError: boolean;
  serverError: boolean;
  get(field: string): string;
}
```

[Response Handling](./response-handling.md)

### Agent for Persistent Configuration

Agent maintains configuration and cookies across multiple requests, useful for test suites and authenticated sessions.

```typescript { .api }
function agent(): Agent;

interface Agent {
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  set(field: string, value?: string): Agent;
  auth(user: string, pass?: string, options?: object): Agent;
  use(plugin: Function): Agent;
}
```

[Agent](./agent.md)

### Promise and Async/Await Support

Native Promise integration for modern asynchronous JavaScript patterns.

```typescript { .api }
interface Request {
  then(resolve: (res: Response) => any, reject?: (err: Error) => any): Promise<any>;
  catch(reject: (err: Error) => any): Promise<any>;
}
```

[Promises](./promises.md)

### Plugin System

Extend SuperAgent functionality with plugins for caching, mocking, prefixes, and custom behaviors.

```typescript { .api }
interface Request {
  use(plugin: (req: Request) => void): Request;
}

interface Agent {
  use(plugin: (req: Request) => void): Agent;
}
```

[Plugins](./plugins.md)

### Error Handling and Retries

Comprehensive error handling with automatic retry logic for network failures and server errors.

```typescript { .api }
interface Request {
  retry(count?: number, callback?: (err: Error, res: Response) => boolean): Request;
  ok(callback: (res: Response) => boolean): Request;
}

interface SuperAgentError extends Error {
  status?: number;
  response?: Response;
  method?: string;
  url?: string;
  timeout?: number;
  code?: string;
  retries?: number;
}
```

[Error Handling](./error-handling.md)

### Serialization and Parsing

Configure custom serializers for request bodies and parsers for response bodies.

```typescript { .api }
interface Request {
  serialize(serializer: (data: any) => string): Request;
  parse(parser: (res: Response, callback: (err: Error | null, body: any) => void) => void): Request;
  buffer(enable?: boolean): Request;
  responseType(type: string): Request;
}

// Global configuration
request.serialize['application/xml'] = function(obj) { /* ... */ };
request.parse['application/xml'] = function(res, callback) { /* ... */ };
```

[Serialization and Parsing](./serialization-parsing.md)

### Streaming (Node.js)

Stream request and response data for large files and memory efficiency.

```typescript { .api }
interface Request {
  pipe(stream: Stream, options?: object): Stream;
  write(data: Buffer | string, encoding?: string): boolean;
}

interface Response {
  pipe(stream: Stream, options?: object): Stream;
  pause(): Response;
  resume(): Response;
}
```

[Streaming](./streaming.md)

### HTTPS and TLS (Node.js)

Configure SSL/TLS certificates for secure HTTPS requests.

```typescript { .api }
interface Request {
  ca(cert: Buffer | string | Array): Request;
  cert(cert: Buffer | string): Request;
  key(key: Buffer | string): Request;
  pfx(pfx: Buffer | string | { pfx: Buffer | string, passphrase: string }): Request;
}
```

[HTTPS and TLS](./https-tls.md)

### HTTP/2 Support (Node.js)

Enable HTTP/2 protocol for improved performance.

```typescript { .api }
interface Request {
  http2(enable?: boolean): Request;
}
```

[HTTP/2](./http2.md)

## Global Configuration

SuperAgent provides global configuration objects for serializers, parsers, and type mappings.

```typescript { .api }
// Serializers
interface SerializeMap {
  [contentType: string]: (data: any) => string;
}
const serialize: SerializeMap;

// Parsers (Node.js)
interface ParseMap {
  [contentType: string]: (res: Response, callback: (err: Error | null, body: any) => void) => void;
}
const parse: ParseMap;

// Type shortcuts (Browser)
interface TypeMap {
  [shortcut: string]: string;
}
const types: TypeMap;

// Buffer configuration (Node.js)
interface BufferMap {
  [contentType: string]: boolean;
}
const buffer: BufferMap;

// Protocol map (Node.js)
interface ProtocolMap {
  'http:': typeof http;
  'https:': typeof https;
  'http2:'?: typeof http2;
}
const protocols: ProtocolMap;
```

## Browser Utilities

Browser-specific utility functions for working with objects and query strings.

```typescript { .api }
/**
 * Get XMLHttpRequest instance (Browser only)
 * @returns {Function} XHR constructor
 */
function getXHR(): Function;

/**
 * Serialize object to query string (Browser only)
 * @param {object} obj - Object to serialize
 * @returns {string} URL-encoded query string
 */
function serializeObject(obj: object): string;

/**
 * Parse query string to object (Browser only)
 * @param {string} str - URL-encoded query string
 * @returns {object} Parsed object
 */
function parseString(str: string): object;
```

## Events

SuperAgent requests emit events that can be listened to for monitoring and debugging.

```typescript { .api }
interface Request {
  on(event: 'request', listener: (req: Request) => void): Request;
  on(event: 'response', listener: (res: Response) => void): Request;
  on(event: 'redirect', listener: (res: Response) => void): Request; // Node.js only
  on(event: 'error', listener: (err: Error) => void): Request;
  on(event: 'end', listener: () => void): Request;
  on(event: 'abort', listener: () => void): Request;
  on(event: 'drain', listener: () => void): Request; // Node.js only - write buffer drained
  on(event: 'progress', listener: (event: ProgressEvent) => void): Request;
}

interface ProgressEvent {
  direction: 'upload' | 'download';
  loaded: number;
  total: number;
  percent?: number; // Browser only
}
```

## Types

```typescript { .api }
interface Request {
  method: string;
  url: string;
  header: object;
  cookies: string; // Node.js only
  qs: object;
  qsRaw: string[]; // Deprecated, use ._query
  username: string; // For 'auto' auth type
  password: string; // For 'auto' auth type
  protocol: string; // Node.js only
  host: string; // Node.js only
}

interface Response {
  status: number;
  statusCode: number;
  statusType: number; // 1-5 representing 1xx-5xx
  statusText: string; // Browser only
  header: object;
  headers: object;
  body: any;
  text: string;
  type: string;
  charset: string;
  links: object;
  files: object; // Node.js only - multipart file uploads
  redirects: string[]; // Node.js only
  buffered: boolean; // Node.js only
  req: Request;
  request: Request; // Node.js only
  res: any; // Node.js only - native response object
  xhr: XMLHttpRequest; // Browser only

  // Status flags
  ok: boolean;
  error: Error | false;
  info: boolean;
  redirect: boolean;
  clientError: boolean;
  serverError: boolean;
  created: boolean;
  accepted: boolean;
  noContent: boolean;
  badRequest: boolean;
  unauthorized: boolean;
  forbidden: boolean;
  notFound: boolean;
  notAcceptable: boolean;
  unprocessableEntity: boolean;
}

interface Agent {
  jar: CookieJar; // Node.js only
}

interface SuperAgentError extends Error {
  status?: number;
  response?: Response;
  method?: string;
  url?: string;
  timeout?: number;
  code?: string;
  retries?: number;
  crossDomain?: boolean; // Browser only
  parse?: boolean;
  original?: Error;
  rawResponse?: string; // Browser only
}
```
