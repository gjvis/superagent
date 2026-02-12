# SuperAgent

SuperAgent is a comprehensive HTTP client library that provides a unified, fluent API for making HTTP requests in both browser and Node.js environments. It offers a progressive, chainable interface for building HTTP requests with features including promise-based and callback-based patterns, automatic content-type detection, JSON/form data serialization, query string manipulation, file uploads with multipart/form-data, cookie handling, cross-domain request support, and extensibility through a robust plugin system.

## Package Information

- **Package Name**: superagent
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install superagent`
- **Environments**: Node.js 6+ and modern browsers (Chrome, Firefox, Safari, IE10+)

## Core Imports

```javascript
const request = require('superagent');
```

ES6 modules:

```javascript
import request from 'superagent';
```

## Basic Usage

```javascript
const request = require('superagent');

// Simple GET request with promises
request
  .get('/api/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });

// POST request with JSON body
request
  .post('/api/pet')
  .send({ name: 'Manny', species: 'cat' })
  .set('X-API-Key', 'foobar')
  .set('accept', 'json')
  .end((err, res) => {
    if (err) {
      console.error(err);
    } else {
      console.log(res.body);
    }
  });

// Using async/await
async function getUser(id) {
  try {
    const res = await request.get(`/api/users/${id}`);
    return res.body;
  } catch (err) {
    console.error(err);
  }
}
```

## Architecture

SuperAgent is built around several key components:

- **Request Builder**: Chainable interface for constructing HTTP requests with method, URL, headers, body, and configuration
- **Response Handler**: Automatic parsing of JSON, form data, and text responses with status code helpers
- **Agent**: Stateful request manager that maintains cookies and default configuration across multiple requests
- **Parser/Serializer System**: Extensible content-type handling for custom formats
- **Plugin System**: Middleware-style plugins via `.use()` for cross-cutting concerns
- **Platform Abstraction**: Unified API across Node.js (http/https modules) and browser (XMLHttpRequest)

## Capabilities

### Making HTTP Requests

Core functionality for creating and sending HTTP requests using various HTTP methods.

```javascript { .api }
function request(method: string, url: string): Request;
function request(url: string): Request; // defaults to GET

// HTTP verb shortcuts
request.get(url: string, data?: object, callback?: Function): Request;
request.post(url: string, data?: object, callback?: Function): Request;
request.put(url: string, data?: object, callback?: Function): Request;
request.patch(url: string, data?: object, callback?: Function): Request;
request.delete(url: string, data?: object, callback?: Function): Request;
request.del(url: string, data?: object, callback?: Function): Request; // alias for delete
request.head(url: string, data?: object, callback?: Function): Request;
request.options(url: string, data?: object, callback?: Function): Request;
```

[HTTP Requests](./http-requests.md)

### Request Configuration

Configure request headers, content types, authentication, and other request properties.

```javascript { .api }
interface Request {
  // Headers
  set(field: string, value: string): Request;
  set(headers: object): Request;
  get(field: string): string;
  unset(field: string): Request;

  // Content-Type and Accept
  type(type: string): Request;
  accept(type: string): Request;

  // Authentication
  auth(user: string, pass: string, options?: {type: 'basic' | 'bearer' | 'auto'}): Request;
}
```

[Request Configuration](./request-configuration.md)

### Sending Request Data

Send request bodies in various formats including JSON, form data, and multipart uploads.

```javascript { .api }
interface Request {
  // Send request body
  send(data: any): Request;

  // Query parameters
  query(params: object | string): Request;

  // Form fields and file uploads
  field(name: string, value: string): Request;
  attach(field: string, file: string | Buffer | Blob, options?: {filename?: string, contentType?: string}): Request;
}
```

[Sending Data](./sending-data.md)

### Response Handling

Parse and process HTTP responses with automatic content-type detection and status helpers.

```javascript { .api }
interface Response {
  // Status properties
  status: number;
  statusCode: number;
  statusType: number; // 1-5
  ok: boolean; // 2xx
  error: Error | false;
  clientError: boolean; // 4xx
  serverError: boolean; // 5xx

  // Status code helpers
  created: boolean; // 201
  accepted: boolean; // 202
  noContent: boolean; // 204
  badRequest: boolean; // 400
  unauthorized: boolean; // 401
  forbidden: boolean; // 403
  notFound: boolean; // 404
  notAcceptable: boolean; // 406
  unprocessableEntity: boolean; // 422

  // Content
  header: object;
  headers: object;
  body: any; // parsed response body
  text: string; // raw response text
  type: string; // content-type without params
  charset: string;

  // Methods
  get(field: string): string;
  toError(): Error;
}
```

[Response Handling](./response-handling.md)

### Timeout and Retry

Configure request timeouts and automatic retry logic for failed requests.

```javascript { .api }
interface Request {
  // Timeout configuration
  timeout(ms: number): Request;
  timeout(options: {response?: number, deadline?: number}): Request;
  clearTimeout(): Request;

  // Retry configuration
  retry(count?: number, callback?: Function): Request;
}
```

[Timeout and Retry](./timeout-retry.md)

### Agent for Stateful Requests

Create agents that maintain state (cookies, default headers) across multiple requests.

```javascript { .api }
function request.agent(options?: object): Agent;

interface Agent {
  // HTTP methods
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;
  del(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  options(url: string, callback?: Function): Request;

  // Configuration methods (set defaults for all requests)
  set(field: string, value: string): Agent;
  auth(user: string, pass: string, options?: object): Agent;
  type(type: string): Agent;
  accept(type: string): Agent;
  timeout(ms: number): Agent;
  retry(count: number): Agent;
  use(plugin: Function): Agent;

  // Node.js only
  jar: CookieJar; // cookie management
}
```

[Agent](./agent.md)

### Redirects and Request Control

Configure redirect behavior and control request lifecycle.

```javascript { .api }
interface Request {
  redirects(count: number): Request;
  abort(): Request;
  maxResponseSize(bytes: number): Request;
}
```

[Redirects and Control](./redirects-control.md)

### Custom Parsers and Serializers

Extend SuperAgent with custom content-type parsers and request serializers.

```javascript { .api }
interface Request {
  parse(fn: (res: Response, callback: Function) => void): Request;
  serialize(fn: (obj: any) => string): Request;
  buffer(enable: boolean): Request; // Node.js only
  responseType(type: string): Request; // Browser: 'blob' | 'arraybuffer'
}

// Global parser configuration
request.parse: {[contentType: string]: Function};
request.serialize: {[contentType: string]: Function};
request.buffer: {[contentType: string]: boolean}; // Node.js only
```

[Parsers and Serializers](./parsers-serializers.md)

### Plugin System

Extend request functionality using plugins.

```javascript { .api }
interface Request {
  use(plugin: (req: Request) => void): Request;
}
```

[Plugins](./plugins.md)

### Promise Support

Use promises or async/await instead of callbacks.

```javascript { .api }
interface Request {
  then(onFulfilled: (res: Response) => any, onRejected?: (err: Error) => any): Promise<any>;
  catch(onRejected: (err: Error) => any): Promise<any>;
  end(callback?: (err: Error, res: Response) => void): void;
}
```

[Promises and Async](./promises-async.md)

### Node.js Specific Features

Node.js-only features including streams, SSL/TLS, HTTP/2, and custom agents.

```javascript { .api }
interface Request {
  // Streaming
  pipe(stream: any, options?: object): stream;
  write(data: string | Buffer, encoding?: string): boolean;

  // SSL/TLS
  ca(cert: string | Buffer | Array): Request;
  cert(cert: string | Buffer): Request;
  key(key: string | Buffer): Request;
  pfx(pfx: string | Buffer): Request;

  // HTTP agent
  agent(agent: http.Agent): Request;

  // HTTP/2
  http2(enable?: boolean): Request;
}
```

[Node.js Features](./nodejs-features.md)

### Browser Specific Features

Browser-only features including CORS credentials and XHR access.

```javascript { .api }
interface Request {
  withCredentials(enable?: boolean): Request;
}

// Browser utilities
request.getXHR(): XMLHttpRequest;
request.serializeObject(obj: object): string;
request.parseString(str: string): object;
```

[Browser Features](./browser-features.md)

## Types

```javascript { .api }
// Request class
class Request {
  method: string;
  url: string;
  header: object;
  // All methods listed in capabilities above
}

// Response class
class Response {
  status: number;
  statusCode: number;
  header: object;
  headers: object;
  body: any;
  text: string;
  type: string;
  charset: string;
  // All properties and methods listed above
}

// Agent class
class Agent {
  jar: CookieJar; // Node.js only
  // All methods listed in Agent capability
}
```
