# Advanced Features

This document covers advanced features and extensibility patterns in SuperAgent, including plugins, custom parsers and serializers, streaming, HTTP/2 support, module-level exports, and event handling.

## Plugin System

### use() Method

The `use()` method enables middleware/plugin functionality, allowing you to extend or modify request behavior.

```typescript { .api }
/**
 * Apply a plugin function to the request
 *
 * @param {Function} fn - Plugin function that receives the request instance
 * @returns {Request} The request instance for chaining
 *
 * @example
 * function prefix(req) {
 *   req.url = 'https://api.example.com' + req.url;
 *   return req;
 * }
 *
 * request
 *   .get('/users')
 *   .use(prefix)
 *   .end(callback);
 */
RequestBase.prototype.use = function(fn: (req: Request) => void): Request;
```

### Plugin Examples

#### Authentication Plugin

```javascript
function bearerAuth(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

request
  .get('/api/protected')
  .use(bearerAuth('my-token-123'))
  .end(callback);
```

#### Base URL Plugin

```javascript
function baseUrl(url) {
  return function(req) {
    req.url = url + req.url;
  };
}

const api = baseUrl('https://api.example.com');

request
  .get('/users/123')
  .use(api)
  .end(callback);
```

#### UUID Plugin

```javascript
function uuid(req) {
  req.set('X-Request-ID', `${Date.now()}-${Math.random()}`);
}

request
  .post('/api/data')
  .use(uuid)
  .send(data)
  .end(callback);
```

#### Chaining Multiple Plugins

```javascript
request
  .get('/users')
  .use(baseUrl('https://api.example.com'))
  .use(bearerAuth('token-123'))
  .use(uuid)
  .end(callback);
```

## Custom Parsers

### parse() Method

Override the default response body parser with a custom parser function.

```typescript { .api }
/**
 * Override default response body parser
 *
 * This function will be called to convert incoming data into request.body
 *
 * @param {Function} fn - Parser function (res, callback)
 * @returns {Request} The request instance for chaining
 *
 * @example
 * request
 *   .get('/api/data')
 *   .parse((res, fn) => {
 *     res.text = '';
 *     res.on('data', chunk => { res.text += chunk; });
 *     res.on('end', () => fn(null, JSON.parse(res.text)));
 *   })
 *   .end(callback);
 */
RequestBase.prototype.parse = function(fn: (res: IncomingMessage, callback: Function) => void): Request;
```

### Module-Level parse Object

The `parse` object maps MIME types to parser functions. You can extend or override these globally.

```typescript { .api }
/**
 * Global parser registry mapping MIME types to parser functions
 *
 * @type {Object.<string, Function>}
 *
 * @example
 * // Add custom XML parser
 * request.parse['application/xml'] = function(res, fn) {
 *   res.text = '';
 *   res.setEncoding('utf8');
 *   res.on('data', chunk => { res.text += chunk; });
 *   res.on('end', () => {
 *     try {
 *       fn(null, parseXML(res.text));
 *     } catch (err) {
 *       fn(err);
 *     }
 *   });
 * };
 */
parse: {
  'application/json': Function,
  'application/x-www-form-urlencoded': Function,
  'text/plain': Function,
  'text/html': Function,
  'text/*': Function,
  'image/*': Function,
  'application/octet-stream': Function,
  'application/pdf': Function
}
```

### Node.js Default Parsers

```javascript
// JSON Parser
parse['application/json'] = function(res, fn) {
  res.text = '';
  res.setEncoding('utf8');
  res.on('data', chunk => { res.text += chunk; });
  res.on('end', () => {
    try {
      const body = res.text && JSON.parse(res.text);
      fn(null, body);
    } catch (err) {
      err.rawResponse = res.text || null;
      err.statusCode = res.statusCode;
      fn(err);
    }
  });
};

// URL-encoded Parser
parse['application/x-www-form-urlencoded'] = function(res, fn) {
  res.text = '';
  res.setEncoding('ascii');
  res.on('data', chunk => { res.text += chunk; });
  res.on('end', () => fn(null, qs.parse(res.text)));
};

// Binary/Image Parser
parse.image = function(res, fn) {
  const data = [];
  res.on('data', chunk => { data.push(chunk); });
  res.on('end', () => fn(null, Buffer.concat(data)));
};
```

### Custom Parser Examples

#### CSV Parser

```javascript
request.parse['text/csv'] = function(res, fn) {
  res.text = '';
  res.setEncoding('utf8');
  res.on('data', chunk => { res.text += chunk; });
  res.on('end', () => {
    try {
      const lines = res.text.split('\n');
      const headers = lines[0].split(',');
      const rows = lines.slice(1).map(line => {
        const values = line.split(',');
        return headers.reduce((obj, header, i) => {
          obj[header] = values[i];
          return obj;
        }, {});
      });
      fn(null, rows);
    } catch (err) {
      fn(err);
    }
  });
};

// Use the CSV parser
request
  .get('/data.csv')
  .end((err, res) => {
    console.log(res.body); // Array of objects
  });
```

#### XML Parser

```javascript
const xmlParser = require('xml2js').parseString;

request.parse['application/xml'] = function(res, fn) {
  res.text = '';
  res.setEncoding('utf8');
  res.on('data', chunk => { res.text += chunk; });
  res.on('end', () => {
    xmlParser(res.text, (err, result) => {
      fn(err, result);
    });
  });
};
```

#### Per-Request Parser Override

```javascript
// Override parser for a specific request
request
  .get('/api/data')
  .parse(request.parse['application/json'])
  .end((err, res) => {
    console.log(res.body);
  });

// Custom inline parser
request
  .get('/api/data')
  .buffer(false)
  .parse((res, fn) => {
    // Stream processing without buffering
    res.on('data', chunk => {
      processChunk(chunk);
    });
    res.on('end', () => fn(null, {}));
  })
  .end(callback);
```

## Custom Serializers

### serialize() Method

Override the default request body serializer with a custom function.

```typescript { .api }
/**
 * Override default request body serializer
 *
 * This function will be called to convert data set via .send() or .attach()
 * into the payload to send
 *
 * @param {Function} fn - Serializer function that transforms data
 * @returns {Request} The request instance for chaining
 *
 * @example
 * request
 *   .post('/api/data')
 *   .send({ foo: 123 })
 *   .serialize(data => JSON.stringify(data, null, 2))
 *   .end(callback);
 */
RequestBase.prototype.serialize = function(fn: (data: any) => string): Request;
```

### Module-Level serialize Object

The `serialize` object maps MIME types to serialization functions.

```typescript { .api }
/**
 * Global serializer registry mapping MIME types to serializer functions
 *
 * @type {Object.<string, Function>}
 *
 * @example
 * // Add custom XML serializer
 * request.serialize['application/xml'] = function(obj) {
 *   return convertToXML(obj);
 * };
 */
serialize: {
  'application/x-www-form-urlencoded': Function, // qs.stringify
  'application/json': Function                    // JSON.stringify
}
```

### Serializer Examples

#### Custom JSON Formatting

```javascript
request
  .post('/api/data')
  .type('json')
  .send({ name: 'John', age: 30 })
  .serialize(data => JSON.stringify(data, null, 2))
  .end(callback);
```

#### XML Serializer

```javascript
const js2xml = require('js2xmlparser');

request.serialize['application/xml'] = function(obj) {
  return js2xml.parse('root', obj);
};

request
  .post('/api/data')
  .type('xml')
  .send({ user: { name: 'John', age: 30 } })
  .end(callback);
```

#### YAML Serializer

```javascript
const yaml = require('js-yaml');

request.serialize['application/x-yaml'] = function(obj) {
  return yaml.dump(obj);
};

request
  .post('/api/data')
  .set('Content-Type', 'application/x-yaml')
  .send({ config: { host: 'localhost', port: 8080 } })
  .end(callback);
```

#### Per-Request Serializer Override

```javascript
request
  .post('/api/echo')
  .send({ foo: 123 })
  .serialize(data => '{"bar":456}')
  .end((err, res) => {
    console.log(res.body); // { bar: 456 }
  });
```

## Streaming (Node.js)

SuperAgent supports Node.js streams for both request and response bodies.

### pipe() Method

Pipe the response body to a writable stream.

```typescript { .api }
/**
 * Pipe the response to a stream
 *
 * @param {Stream} stream - Writable stream to pipe response to
 * @param {Object} options - Pipe options
 * @returns {Stream} The stream that was piped to
 *
 * @example
 * const fs = require('fs');
 *
 * request
 *   .get('http://example.com/image.jpg')
 *   .pipe(fs.createWriteStream('image.jpg'));
 */
Request.prototype.pipe = function(stream: WritableStream, options?: Object): WritableStream;
```

### write() Method

Write raw data to the request socket for streaming uploads.

```typescript { .api }
/**
 * Write raw data to the socket
 *
 * @param {Buffer|String} data - Data to write
 * @param {String} encoding - Character encoding (optional)
 * @returns {Boolean} Returns false if the stream wishes for the calling code
 *                     to wait for the 'drain' event
 *
 * @example
 * const fs = require('fs');
 * const stream = fs.createReadStream('large-file.json');
 *
 * const req = request.post('http://example.com/upload');
 * req.type('json');
 *
 * stream.on('data', chunk => {
 *   req.write(chunk);
 * });
 *
 * stream.on('end', () => {
 *   req.end();
 * });
 */
Request.prototype.write = function(data: Buffer | string, encoding?: string): boolean;
```

### Streaming Examples

#### Download File

```javascript
const fs = require('fs');

request
  .get('http://example.com/large-file.zip')
  .on('error', err => console.error(err))
  .pipe(fs.createWriteStream('download.zip'));
```

#### Upload File Stream

```javascript
const fs = require('fs');

const stream = fs.createReadStream('large-file.json');
const req = request.post('http://example.com/upload');

req.type('json');

req.on('response', res => {
  console.log('Upload complete:', res.body);
});

stream.pipe(req);
```

#### Manual Streaming Upload

```javascript
const fs = require('fs');
const stream = fs.createReadStream('data.txt');

const req = request.post('http://example.com/upload');
req.type('text');

stream.on('data', chunk => {
  req.write(chunk);
});

stream.on('end', () => {
  req.end();
});
```

#### Streaming with Progress

```javascript
const fs = require('fs');

request
  .get('http://example.com/large-file.zip')
  .on('progress', event => {
    console.log(`Downloaded ${event.loaded} of ${event.total} bytes`);
  })
  .on('error', err => console.error(err))
  .pipe(fs.createWriteStream('download.zip'));
```

#### Pipe with Redirect Handling

```javascript
const fs = require('fs');

request
  .get('http://example.com/redirected-file')
  .on('redirect', res => {
    console.log('Redirecting to:', res.headers.location);
  })
  .pipe(fs.createWriteStream('output.dat'));
```

## HTTP/2 Support (Node.js)

### http2() Method

Enable or disable HTTP/2 protocol for requests.

```typescript { .api }
/**
 * Enable or disable HTTP/2 support
 *
 * @param {Boolean} enable - true to enable HTTP/2, false to disable (default: true)
 * @returns {Request} The request instance for chaining
 * @throws {Error} If HTTP/2 is not supported by the Node.js version
 *
 * @example
 * // Enable HTTP/2
 * request
 *   .get('https://example.com/api')
 *   .http2()
 *   .end(callback);
 *
 * // Explicitly enable
 * request
 *   .get('https://example.com/api')
 *   .http2(true)
 *   .end(callback);
 *
 * // Disable HTTP/2
 * request
 *   .get('https://example.com/api')
 *   .http2(false)
 *   .end(callback);
 */
Request.prototype.http2 = function(enable?: boolean): Request;
```

### protocols Object

The `protocols` object maps protocol strings to their respective Node.js modules.

```typescript { .api }
/**
 * Protocol handlers for different HTTP protocols
 *
 * @type {Object.<string, Module>}
 *
 * @example
 * // Access protocol modules
 * const https = request.protocols['https:'];
 * const http2 = request.protocols['http2:'];
 */
protocols: {
  'http:': typeof import('http'),
  'https:': typeof import('https'),
  'http2:': typeof import('http2')
}
```

### HTTP/2 Examples

#### Basic HTTP/2 Request

```javascript
request
  .get('https://http2.example.com/api/data')
  .http2()
  .then(res => {
    console.log(res.body);
  });
```

#### HTTP/2 with Custom Options

```javascript
request
  .get('https://http2.example.com/api')
  .http2()
  .set('Accept', 'application/json')
  .query({ limit: 100 })
  .end((err, res) => {
    if (err) throw err;
    console.log(res.body);
  });
```

#### Conditional HTTP/2

```javascript
const useHttp2 = process.env.USE_HTTP2 === 'true';

request
  .get('https://example.com/api')
  .http2(useHttp2)
  .end(callback);
```

## Module-Level Exports

SuperAgent exports several objects that can be used to configure global behavior.

### buffer Object

Configure buffering behavior by MIME type.

```typescript { .api }
/**
 * Buffering configuration by MIME type
 *
 * By default, this is an empty object. Set properties to control buffering
 * behavior for specific content types.
 *
 * @type {Object.<string, boolean>}
 *
 * @example
 * // Enable buffering for XML
 * request.buffer['application/xml'] = true;
 *
 * // Disable buffering for large JSON responses
 * request.buffer['application/json'] = false;
 */
buffer: {
  [mimeType: string]: boolean
}
```

### Complete Export Structure (Node.js)

```typescript { .api }
/**
 * SuperAgent module exports for Node.js
 */
interface SuperAgentModule {
  // Main request function
  (method: string, url: string): Request;

  // Classes
  Request: typeof Request;
  Response: typeof Response;

  // Agent factory
  agent: (options?: AgentOptions) => Agent;

  // Protocol handlers
  protocols: {
    'http:': typeof import('http');
    'https:': typeof import('https');
    'http2:': typeof import('http2');
  };

  // Serialization functions
  serialize: {
    'application/x-www-form-urlencoded': (obj: any) => string;
    'application/json': (obj: any) => string;
  };

  // Parser functions
  parse: {
    'application/json': (res: IncomingMessage, fn: Function) => void;
    'application/x-www-form-urlencoded': (res: IncomingMessage, fn: Function) => void;
    'text/plain': (res: IncomingMessage, fn: Function) => void;
    'text/html': (res: IncomingMessage, fn: Function) => void;
    image: (res: IncomingMessage, fn: Function) => void;
    [mimeType: string]: (res: IncomingMessage, fn: Function) => void;
  };

  // Buffering configuration
  buffer: {
    [mimeType: string]: boolean;
  };

  // HTTP verb methods
  get: (url: string, data?: any, fn?: Function) => Request;
  post: (url: string, data?: any, fn?: Function) => Request;
  put: (url: string, data?: any, fn?: Function) => Request;
  patch: (url: string, data?: any, fn?: Function) => Request;
  delete: (url: string, data?: any, fn?: Function) => Request;
  del: (url: string, data?: any, fn?: Function) => Request;
  head: (url: string, data?: any, fn?: Function) => Request;
  options: (url: string, data?: any, fn?: Function) => Request;
}
```

### Complete Export Structure (Browser)

```typescript { .api }
/**
 * SuperAgent module exports for Browser
 */
interface SuperAgentBrowserModule {
  // Main request function
  (method: string, url: string): Request;

  // Classes
  Request: typeof Request;
  Response: typeof Response;

  // Agent factory
  agent: () => Agent;

  // Helper functions
  getXHR: () => XMLHttpRequest;
  serializeObject: (obj: any) => string;
  parseString: (str: string) => any;

  // MIME type shortcuts
  types: {
    html: 'text/html';
    json: 'application/json';
    xml: 'text/xml';
    urlencoded: 'application/x-www-form-urlencoded';
    form: 'application/x-www-form-urlencoded';
    'form-data': 'application/x-www-form-urlencoded';
  };

  // Serialization functions
  serialize: {
    'application/x-www-form-urlencoded': (obj: any) => string;
    'application/json': (obj: any) => string;
  };

  // Parser functions
  parse: {
    'application/x-www-form-urlencoded': (str: string) => any;
    'application/json': (str: string) => any;
  };

  // HTTP verb methods
  get: (url: string, data?: any, fn?: Function) => Request;
  post: (url: string, data?: any, fn?: Function) => Request;
  put: (url: string, data?: any, fn?: Function) => Request;
  patch: (url: string, data?: any, fn?: Function) => Request;
  delete: (url: string, data?: any, fn?: Function) => Request;
  del: (url: string, data?: any, fn?: Function) => Request;
  head: (url: string, data?: any, fn?: Function) => Request;
  options: (url: string, data?: any, fn?: Function) => Request;
}
```

### Usage Examples

```javascript
// Configure global buffering
const request = require('superagent');

// Don't buffer images (for streaming)
request.buffer['image/jpeg'] = false;
request.buffer['image/png'] = false;

// Add custom XML parser globally
request.parse['application/xml'] = xmlParserFunction;

// Add custom YAML serializer globally
request.serialize['application/x-yaml'] = yamlSerializerFunction;

// Access HTTP/2 module directly
const http2 = request.protocols['http2:'];
```

## Event Handling

SuperAgent requests inherit from EventEmitter and emit various events throughout the request lifecycle.

### Request Events

```typescript { .api }
/**
 * Event: 'request'
 * Emitted when the request is initiated
 *
 * @event request
 * @param {Request} request - The request instance
 */

/**
 * Event: 'response'
 * Emitted when the response is received
 *
 * @event response
 * @param {Response} response - The response instance
 */

/**
 * Event: 'redirect'
 * Emitted when following a redirect
 *
 * @event redirect
 * @param {IncomingMessage} res - The redirect response
 */

/**
 * Event: 'error'
 * Emitted when an error occurs
 *
 * @event error
 * @param {Error} error - The error object
 */

/**
 * Event: 'abort'
 * Emitted when the request is aborted
 *
 * @event abort
 */

/**
 * Event: 'end'
 * Emitted when the request completes
 *
 * @event end
 */

/**
 * Event: 'drain'
 * Emitted when the write buffer drains (Node.js streaming)
 *
 * @event drain
 */

/**
 * Event: 'progress'
 * Emitted during upload/download progress
 *
 * @event progress
 * @param {Object} event - Progress event
 * @param {string} event.direction - 'upload' or 'download'
 * @param {boolean} event.lengthComputable - Whether total is known
 * @param {number} event.loaded - Bytes transferred
 * @param {number} event.total - Total bytes (if known)
 */
```

### Event Examples

#### Error Handling

```javascript
request
  .get('http://example.com/api')
  .on('error', err => {
    console.error('Request failed:', err.message);
    console.error('Status:', err.status);
    console.error('Response:', err.response);
  })
  .end();
```

#### Progress Tracking

```javascript
request
  .post('http://example.com/upload')
  .attach('file', 'large-video.mp4')
  .on('progress', event => {
    if (event.direction === 'upload') {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      console.log(`Upload progress: ${percent}%`);
    }
  })
  .end((err, res) => {
    console.log('Upload complete');
  });
```

#### Redirect Tracking

```javascript
request
  .get('http://example.com/redirect-chain')
  .on('redirect', res => {
    console.log('Redirecting to:', res.headers.location);
  })
  .end((err, res) => {
    console.log('Final URL:', res.request.url);
    console.log('Redirect chain:', res.redirects);
  });
```

#### Response Streaming Events

```javascript
request
  .get('http://example.com/large-file')
  .on('response', res => {
    console.log('Status:', res.status);
    console.log('Headers:', res.headers);
  })
  .on('end', () => {
    console.log('Download complete');
  })
  .pipe(fs.createWriteStream('output.dat'));
```

#### Request Lifecycle Monitoring

```javascript
request
  .post('http://example.com/api')
  .send({ data: 'value' })
  .on('request', req => {
    console.log('Request started:', req.method, req.url);
  })
  .on('response', res => {
    console.log('Response received:', res.status);
  })
  .on('end', () => {
    console.log('Request completed');
  })
  .on('error', err => {
    console.error('Request failed:', err);
  })
  .end();
```

## Extending SuperAgent

### Subclassing Request

You can create custom Request classes to add functionality.

```javascript
const request = require('superagent');
const OriginalRequest = request.Request;

function CustomRequest(method, url) {
  OriginalRequest.call(this, method, url);
  // Add custom initialization
  this.customProperty = 'value';
}

// Inherit from OriginalRequest
CustomRequest.prototype = Object.create(OriginalRequest.prototype);
CustomRequest.prototype.constructor = CustomRequest;

// Add custom methods
CustomRequest.prototype.customMethod = function() {
  this.set('X-Custom-Header', 'value');
  return this;
};

// Override existing methods
CustomRequest.prototype.end = function(fn) {
  console.log('Custom request ending');
  return OriginalRequest.prototype.end.call(this, fn);
};

// Replace the Request class
request.Request = CustomRequest;

// Now all requests use CustomRequest
request
  .get('/api/data')
  .customMethod()
  .end(callback);
```

### Creating Request Wrappers

```javascript
class APIClient {
  constructor(baseURL, apiKey) {
    this.baseURL = baseURL;
    this.apiKey = apiKey;
  }

  request(method, path) {
    return request(method, this.baseURL + path)
      .set('Authorization', `Bearer ${this.apiKey}`)
      .set('User-Agent', 'MyApp/1.0');
  }

  get(path) {
    return this.request('GET', path);
  }

  post(path, data) {
    return this.request('POST', path).send(data);
  }

  put(path, data) {
    return this.request('PUT', path).send(data);
  }

  delete(path) {
    return this.request('DELETE', path);
  }
}

// Usage
const api = new APIClient('https://api.example.com', 'my-api-key');

api.get('/users/123').then(res => {
  console.log(res.body);
});

api.post('/users', { name: 'John' }).then(res => {
  console.log('Created:', res.body);
});
```

### Agent with Default Settings

```javascript
const request = require('superagent');

// Create agent with default settings
const agent = request.agent()
  .set('User-Agent', 'MyApp/1.0')
  .set('Accept', 'application/json')
  .timeout(5000)
  .retry(2);

// All requests from this agent inherit defaults
agent.get('http://example.com/api/users').end(callback);
agent.post('http://example.com/api/users').send(data).end(callback);
```

### Plugin for Common Patterns

```javascript
// Retry with exponential backoff
function retryWithBackoff(retries = 3) {
  let attempt = 0;

  return function(req) {
    req.retry(retries, (err, res) => {
      if (err) {
        attempt++;
        const delay = Math.pow(2, attempt) * 1000;
        return delay;
      }
    });
  };
}

// Timeout with custom error message
function timeout(ms, message) {
  return function(req) {
    req.timeout(ms).on('error', err => {
      if (err.timeout) {
        err.message = message || `Request timeout after ${ms}ms`;
      }
    });
  };
}

// Usage
request
  .get('/api/slow-endpoint')
  .use(retryWithBackoff(3))
  .use(timeout(5000, 'API took too long to respond'))
  .end(callback);
```

### Global Request Interceptor

```javascript
const request = require('superagent');
const OriginalRequest = request.Request;

function InterceptedRequest(method, url) {
  OriginalRequest.call(this, method, url);
}

InterceptedRequest.prototype = Object.create(OriginalRequest.prototype);

InterceptedRequest.prototype.end = function(fn) {
  // Log all requests
  console.log(`[${this.method}] ${this.url}`);

  return OriginalRequest.prototype.end.call(this, (err, res) => {
    // Log all responses
    if (err) {
      console.error(`[ERROR] ${this.method} ${this.url}:`, err.message);
    } else {
      console.log(`[${res.status}] ${this.method} ${this.url}`);
    }

    if (fn) fn(err, res);
  });
};

request.Request = InterceptedRequest;
```

## Best Practices

### Error Handling

Always handle errors, especially with promises:

```javascript
// With promises
try {
  const res = await request.get('/api/data');
  console.log(res.body);
} catch (err) {
  console.error('Request failed:', err.message);
  console.error('Status:', err.status);
}

// With callbacks
request
  .get('/api/data')
  .end((err, res) => {
    if (err) {
      console.error('Request failed:', err);
      return;
    }
    console.log(res.body);
  });
```

### Plugin Organization

Keep plugins small and focused:

```javascript
// plugins/auth.js
module.exports = function(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
};

// plugins/retry.js
module.exports = function(retries = 3) {
  return function(req) {
    req.retry(retries);
  };
};

// Usage
const auth = require('./plugins/auth');
const retry = require('./plugins/retry');

request
  .get('/api/data')
  .use(auth(process.env.API_TOKEN))
  .use(retry(3))
  .end(callback);
```

### Streaming Performance

Use streaming for large files:

```javascript
// DON'T: Buffer large files in memory
const res = await request.get('http://example.com/large-video.mp4');
fs.writeFileSync('video.mp4', res.body);

// DO: Stream directly to disk
request
  .get('http://example.com/large-video.mp4')
  .pipe(fs.createWriteStream('video.mp4'));
```

### Custom Parser Performance

For large responses, use streaming parsers:

```javascript
// DON'T: Buffer entire response
request
  .get('/api/huge-dataset')
  .parse((res, fn) => {
    res.text = '';
    res.on('data', chunk => { res.text += chunk; });
    res.on('end', () => fn(null, JSON.parse(res.text)));
  });

// DO: Use streaming JSON parser for huge datasets
const JSONStream = require('JSONStream');

request
  .get('/api/huge-dataset')
  .buffer(false)
  .parse((res, fn) => {
    const parser = JSONStream.parse('*');
    res.pipe(parser);

    const results = [];
    parser.on('data', data => results.push(data));
    parser.on('end', () => fn(null, results));
  });
```
