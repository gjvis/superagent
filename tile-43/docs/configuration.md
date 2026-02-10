# Global Configuration

This document covers global configuration options including custom serializers, parsers, buffering strategies, and other package-level settings.

## Global Objects

SuperAgent exposes several configuration objects on the main `request` export that can be modified to customize behavior globally.

```javascript { .api }
interface SuperAgent {
  // Serialization map
  serialize: {
    [contentType: string]: (data: any) => string;
  };

  // Parsing map
  parse: {
    [contentType: string]: (res: any, callback: (err: Error | null, body: any, files?: any) => void) => void;
  };

  // Buffering configuration map
  buffer: {
    [contentType: string]: boolean;
  };

  // Protocol handlers (Node.js only)
  protocols: {
    'http:': any;
    'https:': any;
    'http2:': any;
  };

  // MIME type shortcuts (Browser only)
  types?: {
    html: string;
    json: string;
    xml: string;
    urlencoded: string;
    form: string;
    'form-data': string;
  };

  // Utility functions (Browser only)
  getXHR?: () => XMLHttpRequest;
  serializeObject?: (obj: object) => string;
  parseString?: (str: string) => object;
}
```

## Capabilities

### Custom Serializers

Define how request bodies are serialized for specific content types.

```javascript { .api }
/**
 * Global serialization map
 * Maps Content-Type to serialization function
 */
request.serialize: {
  'application/x-www-form-urlencoded': (data: any) => string;
  'application/json': (data: any) => string;
  [contentType: string]: (data: any) => string;
};
```

**Default serializers:**
- `application/x-www-form-urlencoded` → `qs.stringify()` (Node.js) or custom serializer (browser)
- `application/json` → `JSON.stringify()`

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom XML serializer
request.serialize['application/xml'] = function(obj) {
  // Custom XML serialization logic
  return `<root>${Object.keys(obj).map(k => `<${k}>${obj[k]}</${k}>`).join('')}</root>`;
};

// Now requests with XML content type use custom serializer
request
  .post('/api/data')
  .type('application/xml')
  .send({ name: 'Alice', age: 30 })
  .then(res => console.log('Sent XML'));
// Request body: <root><name>Alice</name><age>30</age></root>

// Override JSON serializer
request.serialize['application/json'] = function(obj) {
  // Custom JSON serialization with indentation
  return JSON.stringify(obj, null, 2);
};

// Add YAML serializer
request.serialize['application/x-yaml'] = function(obj) {
  // Assuming you have a YAML library
  return require('js-yaml').dump(obj);
};
```

### Per-Request Serializers

Override the global serializer for a specific request.

```javascript { .api }
/**
 * Set custom serializer for this request only
 * @param serializer - Function to serialize request body
 * @returns Request instance for chaining
 */
serialize(serializer: (data: any) => string): Request;
```

**Usage Example:**

```javascript
request
  .post('/api/data')
  .type('application/json')
  .serialize(data => {
    // Custom serialization for this request only
    return JSON.stringify(data, null, 4);
  })
  .send({ name: 'Alice' });
```

### Custom Parsers

Define how response bodies are parsed for specific content types.

```javascript { .api }
/**
 * Global parsing map
 * Maps Content-Type to parser function
 */
request.parse: {
  'application/json': (res: any, callback: (err: Error | null, body: any) => void) => void;
  'application/x-www-form-urlencoded': (res: any, callback: (err: Error | null, body: any) => void) => void;
  'text/*': (res: any, callback: (err: Error | null, body: any) => void) => void;
  'image/*': (res: any, callback: (err: Error | null, body: any) => void) => void;
  [contentType: string]: (res: any, callback: (err: Error | null, body: any) => void) => void;
};
```

**Default parsers (Node.js):**
- `application/json` → JSON parser
- `application/x-www-form-urlencoded` → URL-encoded parser
- `text/*` → Text parser (returns string)
- `image/*`, `application/octet-stream`, `application/pdf` → Binary parser (returns Buffer)
- `multipart/*` → Multipart parser using formidable

**Default parsers (Browser):**
- `application/json` → `JSON.parse()`
- `application/x-www-form-urlencoded` → Custom URL-encoded parser

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom XML parser (Node.js)
request.parse['application/xml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    try {
      // Assuming you have an XML parser library
      const xml2js = require('xml2js');
      xml2js.parseString(data, (err, result) => {
        callback(err, result);
      });
    } catch (err) {
      callback(err);
    }
  });
};

// Now XML responses are automatically parsed
request
  .get('/api/data.xml')
  .then(res => {
    console.log('Parsed XML:', res.body);  // JavaScript object
  });

// Add YAML parser
request.parse['application/x-yaml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    try {
      const yaml = require('js-yaml');
      const obj = yaml.load(data);
      callback(null, obj);
    } catch (err) {
      callback(err);
    }
  });
};

// Add CSV parser
request.parse['text/csv'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    try {
      // Simple CSV parsing (use a library for production)
      const rows = data.split('\n').map(row => row.split(','));
      callback(null, rows);
    } catch (err) {
      callback(err);
    }
  });
};
```

### Per-Request Parsers

Override the global parser for a specific request.

```javascript { .api }
/**
 * Set custom parser for this request only
 * @param parser - Function to parse response body
 * @returns Request instance for chaining
 */
parse(parser: (res: any, callback: (err: Error | null, body: any, files?: any) => void) => void): Request;
```

**Usage Example:**

```javascript
// Custom parser for a single request (Node.js)
request
  .get('/api/data')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      // Custom parsing logic
      try {
        const parsed = customParseLogic(data);
        callback(null, parsed);
      } catch (err) {
        callback(err);
      }
    });
  })
  .then(res => {
    console.log('Custom parsed body:', res.body);
  });

// Disable parsing for a request
request
  .get('/api/raw-data')
  .parse((res, callback) => {
    // Return raw response without parsing
    callback(null, res);
  })
  .then(res => {
    console.log('Raw response:', res.body);
  });
```

### Buffering Configuration

Control which content types are buffered by default.

```javascript { .api }
/**
 * Global buffering configuration map
 * Maps Content-Type to buffering boolean
 */
request.buffer: {
  [contentType: string]: boolean;
};
```

**Default buffering:**
- Most content types are buffered by default
- Can be customized per content type globally
- Can be overridden per request with `.buffer(true/false)`

**Usage Examples:**

```javascript
const request = require('superagent');

// Disable buffering for large JSON responses by default
request.buffer['application/json'] = false;

// Enable buffering for a specific binary type
request.buffer['application/octet-stream'] = true;

// Now requests inherit these defaults
request
  .get('/api/large-data.json')
  .then(res => {
    // Not buffered (need to use .pipe() or .buffer(true))
  });

// Override per request
request
  .get('/api/large-data.json')
  .buffer(true)  // Force buffering for this request
  .then(res => {
    console.log('Buffered:', res.body);
  });
```

### Protocol Handlers (Node.js only)

Access or customize protocol handlers for HTTP, HTTPS, and HTTP/2.

```javascript { .api }
/**
 * Protocol handlers map (Node.js only)
 */
request.protocols: {
  'http:': typeof http;
  'https:': typeof https;
  'http2:': any;
};
```

**Usage Example:**

```javascript
const request = require('superagent');
const http = require('http');
const https = require('https');

// Access protocol modules
console.log('HTTP module:', request.protocols['http:']);
console.log('HTTPS module:', request.protocols['https:']);
console.log('HTTP/2 module:', request.protocols['http2:']);

// These are the built-in Node.js modules used for requests
// Advanced: Could theoretically be replaced with custom implementations
```

### MIME Type Shortcuts (Browser only)

Predefined MIME type shortcuts for convenience.

```javascript { .api }
/**
 * MIME type shortcuts (Browser only)
 */
request.types: {
  html: 'text/html';
  json: 'application/json';
  xml: 'text/xml';
  urlencoded: 'application/x-www-form-urlencoded';
  form: 'application/x-www-form-urlencoded';
  'form-data': 'application/x-www-form-urlencoded';
};
```

**Usage Example:**

```javascript
// Browser only
console.log(request.types.json);  // 'application/json'
console.log(request.types.xml);   // 'text/xml'

// Add custom type shortcut
request.types.yaml = 'application/x-yaml';

// Use in requests
request
  .post('/api/data')
  .type('yaml')  // Uses custom shortcut
  .send(data);
```

### Utility Functions (Browser only)

Helper functions available in browser builds.

```javascript { .api }
/**
 * Get XMLHttpRequest constructor (Browser only)
 * @returns XMLHttpRequest constructor
 */
request.getXHR(): XMLHttpRequest;

/**
 * Serialize object to query string (Browser only)
 * @param obj - Object to serialize
 * @returns URL-encoded query string
 */
request.serializeObject(obj: object): string;

/**
 * Parse URL-encoded string to object (Browser only)
 * @param str - URL-encoded string
 * @returns Parsed object
 */
request.parseString(str: string): object;
```

**Usage Examples:**

```javascript
// Browser only

// Get XHR constructor
const XHR = request.getXHR();
const xhr = new XHR();

// Serialize object
const query = request.serializeObject({ name: 'Alice', age: 30 });
console.log(query);  // 'name=Alice&age=30'

// Parse string
const obj = request.parseString('name=Alice&age=30');
console.log(obj);  // { name: 'Alice', age: '30' }
```

### Complete Configuration Example

```javascript
const request = require('superagent');
const yaml = require('js-yaml');
const xml2js = require('xml2js');

// Configure custom serializers
request.serialize['application/x-yaml'] = function(obj) {
  return yaml.dump(obj);
};

request.serialize['application/xml'] = function(obj) {
  const builder = new xml2js.Builder();
  return builder.buildObject(obj);
};

// Configure custom parsers
request.parse['application/x-yaml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => { data += chunk; });
  res.on('end', () => {
    try {
      callback(null, yaml.load(data));
    } catch (err) {
      callback(err);
    }
  });
};

request.parse['application/xml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => { data += chunk; });
  res.on('end', () => {
    xml2js.parseString(data, callback);
  });
};

// Configure buffering
request.buffer['application/x-yaml'] = true;
request.buffer['application/xml'] = true;

// Now YAML and XML are fully supported
request
  .post('/api/data')
  .type('application/x-yaml')
  .send({ name: 'Alice', items: [1, 2, 3] })
  .then(res => console.log('YAML sent'));

request
  .get('/api/data.xml')
  .then(res => {
    console.log('Parsed XML:', res.body);
  });
```

### Plugin System

Extend SuperAgent functionality with plugins using the `.use()` method.

```javascript { .api }
/**
 * Apply plugin to request
 * @param plugin - Function that receives and modifies the request
 * @returns Request instance for chaining
 */
use(plugin: (req: Request) => void): Request;
```

**Usage Examples:**

```javascript
// Define a plugin
function customPlugin(req) {
  // Modify request
  req.set('X-Custom-Header', 'value');
  req.timeout(5000);

  // Can also attach event listeners
  req.on('response', res => {
    console.log('Response received:', res.status);
  });
}

// Use plugin on a single request
request
  .get('/api/users')
  .use(customPlugin)
  .then(res => console.log(res.body));

// Apply plugin to all requests via agent
const agent = request.agent()
  .use(customPlugin);

agent.get('/api/users').then(res => {
  // Plugin automatically applied
  console.log(res.body);
});

// Plugin with configuration
function authPlugin(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

request
  .get('/api/users')
  .use(authPlugin('my-token-123'))
  .then(res => console.log(res.body));

// Logging plugin
function loggingPlugin(req) {
  const start = Date.now();

  req.on('request', () => {
    console.log(`[${req.method}] ${req.url}`);
  });

  req.on('response', res => {
    const duration = Date.now() - start;
    console.log(`[${res.status}] ${req.url} (${duration}ms)`);
  });
}

request
  .get('/api/users')
  .use(loggingPlugin)
  .then(res => console.log(res.body));
```
