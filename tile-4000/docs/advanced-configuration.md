# Advanced Configuration

Advanced features including custom parsers, serializers, plugins, and response validation.

## Capabilities

### Custom Response Parser

Override the default response body parser with a custom function.

```javascript { .api }
/**
 * Set custom response parser
 * @param parser - Function to parse response body
 * @returns Request instance for chaining
 */
Request.prototype.parse(parser: (res: Response, callback: (err: Error, body: any) => void) => void): Request;
```

**Usage Examples:**

```javascript
// Custom XML parser
request.get('https://api.example.com/data.xml')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      const xml = parseXML(data); // Your XML parser
      callback(null, xml);
    });
  })
  .then(res => {
    console.log('Parsed XML:', res.body);
  });

// Custom CSV parser
request.get('https://api.example.com/data.csv')
  .parse((res, callback) => {
    let csv = '';
    res.on('data', chunk => {
      csv += chunk;
    });
    res.on('end', () => {
      const rows = csv.split('\n').map(row => row.split(','));
      callback(null, rows);
    });
  })
  .then(res => {
    console.log('Parsed CSV:', res.body);
  });

// Skip parsing (get raw buffer)
request.get('https://api.example.com/data')
  .parse((res, callback) => {
    callback(null, res); // Return raw response
  })
  .then(res => {
    console.log('Raw response:', res.body);
  });
```

### Custom Request Serializer

Override the default request body serializer.

```javascript { .api }
/**
 * Set custom request body serializer
 * @param serializer - Function to serialize request body
 * @returns Request instance for chaining
 */
Request.prototype.serialize(serializer: (data: any) => string): Request;
```

**Usage Examples:**

```javascript
// Custom XML serializer
request.post('https://api.example.com/data')
  .type('xml')
  .serialize(data => {
    return `<root><name>${data.name}</name></root>`;
  })
  .send({ name: 'John' })
  .then(res => console.log(res.body));

// Custom CSV serializer
request.post('https://api.example.com/data')
  .type('text/csv')
  .serialize(data => {
    return data.map(row => row.join(',')).join('\n');
  })
  .send([[1, 2, 3], [4, 5, 6]])
  .then(res => console.log(res.body));

// Custom JSON serializer with formatting
request.post('https://api.example.com/data')
  .serialize(data => {
    return JSON.stringify(data, null, 2); // Pretty print
  })
  .send({ name: 'John', email: 'john@example.com' });
```

### Response Type

Set the expected response format (browser).

```javascript { .api }
/**
 * Set response type for browser XMLHttpRequest
 * @param type - Response type ('blob', 'arraybuffer', 'text', 'document', 'json')
 * @returns Request instance for chaining
 */
Request.prototype.responseType(type: string): Request;
```

**Usage Examples:**

```javascript
// Get binary data as Blob (browser)
request.get('https://api.example.com/image.png')
  .responseType('blob')
  .then(res => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    document.querySelector('img').src = url;
  });

// Get binary data as ArrayBuffer (browser)
request.get('https://api.example.com/data.bin')
  .responseType('arraybuffer')
  .then(res => {
    const buffer = res.body;
    const view = new DataView(buffer);
    console.log('First byte:', view.getUint8(0));
  });

// Get HTML as document (browser)
request.get('https://api.example.com/page.html')
  .responseType('document')
  .then(res => {
    const doc = res.body;
    console.log('Title:', doc.querySelector('title').textContent);
  });
```

### Plugin System

Extend requests using middleware functions.

```javascript { .api }
/**
 * Apply plugin/middleware function
 * @param plugin - Function receiving the request instance
 * @returns Request instance for chaining
 */
Request.prototype.use(plugin: (req: Request) => void): Request;
```

**Usage Examples:**

```javascript
// Simple plugin
function addTimestamp(req) {
  req.set('X-Timestamp', Date.now().toString());
}

request.get('https://api.example.com/users')
  .use(addTimestamp)
  .then(res => console.log(res.body));

// Plugin with configuration
function prefix(baseURL) {
  return function(req) {
    if (!req.url.startsWith('http')) {
      req.url = baseURL + req.url;
    }
  };
}

request.get('/users')
  .use(prefix('https://api.example.com'))
  .then(res => console.log(res.body));
// Actual URL: https://api.example.com/users

// Logging plugin
function logger(req) {
  const start = Date.now();
  console.log('Request:', req.method, req.url);

  req.on('response', res => {
    const duration = Date.now() - start;
    console.log('Response:', res.status, `(${duration}ms)`);
  });

  req.on('error', err => {
    console.error('Error:', err.message);
  });
}

request.get('https://api.example.com/users')
  .use(logger);

// Authentication plugin
function auth(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

const userToken = 'abc123';
request.get('https://api.example.com/profile')
  .use(auth(userToken))
  .then(res => console.log(res.body));

// Multiple plugins
request.get('https://api.example.com/users')
  .use(logger)
  .use(addTimestamp)
  .use(auth('token123'));
```

### Custom Response Validation

Override the default response validation logic.

```javascript { .api }
/**
 * Set custom response validator
 * @param validator - Function to validate response (return true for OK)
 * @returns Request instance for chaining
 */
Request.prototype.ok(validator: (res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Accept only 200 status
request.get('https://api.example.com/users')
  .ok(res => res.status === 200)
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    // Any non-200 status will reject
    console.error('Not OK:', err.status);
  });

// Accept 2xx and 3xx
request.get('https://api.example.com/users')
  .ok(res => res.status < 400)
  .then(res => console.log(res.body));

// Accept specific error codes as valid
request.get('https://api.example.com/users/123')
  .ok(res => res.status === 200 || res.status === 404)
  .then(res => {
    if (res.status === 404) {
      console.log('User not found');
    } else {
      console.log('User:', res.body);
    }
  });

// Custom business logic validation
request.get('https://api.example.com/users')
  .ok(res => {
    if (!res.ok) return false;
    if (!res.body || !Array.isArray(res.body)) return false;
    return true;
  })
  .then(res => console.log('Valid array:', res.body))
  .catch(err => console.error('Invalid response'));

// Always succeed (never reject on status)
request.get('https://api.example.com/users')
  .ok(() => true)
  .then(res => {
    // Always resolves, even on 4xx/5xx
    console.log('Status:', res.status);
    console.log('Body:', res.body);
  });
```

## Global Configuration

### Default Serializers

Customize global serializers for specific content types.

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom serializer for XML
request.serialize['application/xml'] = function(obj) {
  return `<root>${JSON.stringify(obj)}</root>`;
};

// Now all XML requests use the custom serializer
request.post('https://api.example.com/data')
  .type('xml')
  .send({ name: 'John' });
// Body: <root>{"name":"John"}</root>
```

### Default Parsers

Customize global parsers for specific content types (Node.js).

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom parser for XML
request.parse['application/xml'] = function(res, callback) {
  let xml = '';
  res.on('data', chunk => {
    xml += chunk;
  });
  res.on('end', () => {
    const parsed = parseXML(xml);
    callback(null, parsed);
  });
};

// Now all XML responses use the custom parser
request.get('https://api.example.com/data.xml')
  .then(res => {
    console.log('Parsed XML:', res.body);
  });
```

### Buffer Control (Node.js)

Control response buffering per content type.

**Usage Examples:**

```javascript
const request = require('superagent');

// Disable buffering for large files
request.buffer['application/zip'] = false;

// Request large file without buffering
request.get('https://api.example.com/large-file.zip')
  .pipe(fs.createWriteStream('output.zip'));

// Enable buffering for specific type
request.buffer['application/custom'] = true;
```

## Plugin Examples

### Cache Plugin

```javascript
const cache = {};

function cachePlugin(req) {
  const key = `${req.method}:${req.url}`;

  // Check cache
  if (req.method === 'GET' && cache[key]) {
    console.log('Returning cached response');
    req.abort(); // Cancel request
    return Promise.resolve(cache[key]);
  }

  // Store in cache on response
  req.on('response', res => {
    if (req.method === 'GET' && res.ok) {
      cache[key] = res;
    }
  });
}

// Usage
request.get('https://api.example.com/users')
  .use(cachePlugin);
```

### Retry Plugin with Backoff

```javascript
function retryWithBackoff(maxRetries = 3) {
  return function(req) {
    let attempt = 0;

    function retry() {
      if (attempt >= maxRetries) {
        return;
      }

      req.on('error', err => {
        if (err.status >= 500 || err.timeout) {
          attempt++;
          const delay = Math.pow(2, attempt) * 1000;
          console.log(`Retry attempt ${attempt} after ${delay}ms`);

          setTimeout(() => {
            req.retry();
          }, delay);
        }
      });
    }

    retry();
  };
}

// Usage
request.get('https://api.example.com/users')
  .use(retryWithBackoff(3));
```

### Request ID Plugin

```javascript
function requestIdPlugin(req) {
  const requestId = Math.random().toString(36).substring(7);
  req.set('X-Request-ID', requestId);

  console.log(`[${requestId}] Starting request to ${req.url}`);

  req.on('response', res => {
    console.log(`[${requestId}] Response: ${res.status}`);
  });

  req.on('error', err => {
    console.error(`[${requestId}] Error: ${err.message}`);
  });
}

// Usage
request.get('https://api.example.com/users')
  .use(requestIdPlugin);
```

## Important Notes

### Parser and Serializer Scope

- `.parse()` and `.serialize()` apply to individual requests
- Global `request.parse` and `request.serialize` apply to all requests
- Per-request configuration overrides global configuration

### Plugin Execution Order

- Plugins are executed in the order they are added with `.use()`
- Plugins execute before the request is sent
- Event listeners in plugins execute at appropriate times

### Response Validation

- Default behavior: 2xx status codes are OK, others throw errors
- Custom `.ok()` validator overrides default behavior
- Validator receives the full Response object
- Return `true` for successful responses, `false` to reject
