# Global Configuration

SuperAgent provides global configuration objects for customizing default serializers, parsers, buffering behavior, and type mappings.

## Capabilities

### Global Serializers

Customize how request bodies are serialized for specific content types.

```javascript { .api }
/**
 * Global serializers map - Content-Type to serializer function
 */
request.serialize: {
  [contentType: string]: (data: any) => string;
};
```

**Default Serializers:**
- `'application/x-www-form-urlencoded'`: Uses `qs.stringify`
- `'application/json'`: Uses `JSON.stringify`

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom XML serializer
request.serialize['application/xml'] = function(obj) {
  // Custom XML serialization logic
  return `<root>${JSON.stringify(obj)}</root>`;
};

// Now all requests with type 'xml' use the custom serializer
request.post('https://api.example.com/data')
  .type('xml')
  .send({ name: 'John', age: 30 })
  .then(res => console.log(res.body));
// Body sent: <root>{"name":"John","age":30}</root>

// Add CSV serializer
request.serialize['text/csv'] = function(data) {
  if (Array.isArray(data)) {
    return data.map(row => row.join(',')).join('\n');
  }
  return String(data);
};

// Use CSV serializer
request.post('https://api.example.com/data')
  .type('text/csv')
  .send([
    ['Name', 'Age', 'City'],
    ['John', '30', 'New York'],
    ['Jane', '25', 'London']
  ]);
```

### Global Parsers (Node.js)

Customize how response bodies are parsed for specific content types.

```javascript { .api }
/**
 * Global parsers map - Content-Type to parser function (Node.js only)
 */
request.parse: {
  [contentType: string]: (res: Response, callback: (err: Error, body: any) => void) => void;
};
```

**Default Parsers (Node.js):**
- `'application/x-www-form-urlencoded'`: Parses URL-encoded data
- `'application/json'`: Parses JSON with `JSON.parse`
- `'text/*'`: Returns text string
- `'image/*'`: Returns Buffer
- `'application/octet-stream'`: Returns Buffer
- `'application/pdf'`: Returns Buffer

**Usage Examples:**

```javascript
const request = require('superagent');

// Add custom XML parser
request.parse['application/xml'] = function(res, callback) {
  let xml = '';

  res.on('data', chunk => {
    xml += chunk;
  });

  res.on('end', () => {
    try {
      // Use your XML parser (e.g., xml2js, fast-xml-parser)
      const parsed = parseXML(xml); // Your XML parsing function
      callback(null, parsed);
    } catch (err) {
      callback(err);
    }
  });

  res.on('error', err => {
    callback(err);
  });
};

// Now all XML responses are automatically parsed
request.get('https://api.example.com/data.xml')
  .then(res => {
    console.log('Parsed XML:', res.body);
  });

// Add YAML parser
request.parse['application/yaml'] = function(res, callback) {
  let yaml = '';

  res.on('data', chunk => {
    yaml += chunk;
  });

  res.on('end', () => {
    try {
      const parsed = require('js-yaml').load(yaml);
      callback(null, parsed);
    } catch (err) {
      callback(err);
    }
  });
};
```

### Global Buffering (Node.js)

Control whether responses are buffered by content type.

```javascript { .api }
/**
 * Global buffer map - Content-Type to buffering flag (Node.js only)
 */
request.buffer: {
  [contentType: string]: boolean;
};
```

**Default Buffering:**
- Text and JSON responses: `true` (buffered)
- Binary responses: Depends on content type

**Usage Examples:**

```javascript
const request = require('superagent');

// Disable buffering for large JSON files
request.buffer['application/json'] = false;

// Requests for large JSON will not be buffered
request.get('https://api.example.com/large-data.json')
  .pipe(fs.createWriteStream('output.json'));

// Enable buffering for specific binary type
request.buffer['application/octet-stream'] = true;

// Disable buffering for video streams
request.buffer['video/mp4'] = false;
request.buffer['video/*'] = false;

// Now video requests can be streamed
request.get('https://api.example.com/video.mp4')
  .pipe(fs.createWriteStream('video.mp4'));
```

### Type Shortcuts (Browser)

MIME type shortcuts for common content types.

```javascript { .api }
/**
 * Type shortcuts map (browser only)
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

**Usage Examples:**

```javascript
// These shortcuts are used internally
request.post('/api')
  .type('json')  // Translates to 'application/json'
  .send({ name: 'John' });

request.post('/api')
  .type('form')  // Translates to 'application/x-www-form-urlencoded'
  .send({ name: 'John' });

// Add custom type shortcut
request.types['yaml'] = 'application/yaml';

// Now you can use the shortcut
request.post('/api')
  .type('yaml')
  .send('name: John\nage: 30');
```

### Protocol Handlers (Node.js)

Access to Node.js protocol modules.

```javascript { .api }
/**
 * Protocol handlers (Node.js only)
 */
request.protocols: {
  'http:': typeof http;
  'https:': typeof https;
  'http2:': typeof http2 | undefined;
};
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Check available protocols
console.log('HTTP module:', request.protocols['http:']);
console.log('HTTPS module:', request.protocols['https:']);
console.log('HTTP/2 module:', request.protocols['http2:']);

// Check if HTTP/2 is available
if (request.protocols['http2:']) {
  console.log('HTTP/2 is supported');
  // Can use .http2() method
} else {
  console.log('HTTP/2 is not available in this Node.js version');
}

// Access protocol modules directly
const https = request.protocols['https:'];
```

## Configuration Patterns

### Custom Content Type Support

```javascript
const request = require('superagent');

// Add MessagePack support
request.serialize['application/msgpack'] = function(obj) {
  return require('msgpack').encode(obj);
};

request.parse['application/msgpack'] = function(res, callback) {
  const chunks = [];

  res.on('data', chunk => chunks.push(chunk));
  res.on('end', () => {
    try {
      const buffer = Buffer.concat(chunks);
      const decoded = require('msgpack').decode(buffer);
      callback(null, decoded);
    } catch (err) {
      callback(err);
    }
  });
};

// Usage
request.post('https://api.example.com/data')
  .type('application/msgpack')
  .send({ data: 'value' })
  .then(res => console.log(res.body));
```

### Protocol Buffer Support

```javascript
const request = require('superagent');
const protobuf = require('protobufjs');

// Load proto definition
const root = protobuf.loadSync('schema.proto');
const Message = root.lookupType('MyMessage');

// Add protobuf serializer
request.serialize['application/protobuf'] = function(obj) {
  const message = Message.create(obj);
  return Buffer.from(Message.encode(message).finish());
};

// Add protobuf parser
request.parse['application/protobuf'] = function(res, callback) {
  const chunks = [];

  res.on('data', chunk => chunks.push(chunk));
  res.on('end', () => {
    try {
      const buffer = Buffer.concat(chunks);
      const message = Message.decode(buffer);
      callback(null, Message.toObject(message));
    } catch (err) {
      callback(err);
    }
  });
};

// Usage
request.post('https://api.example.com/protobuf')
  .type('application/protobuf')
  .send({ field1: 'value1', field2: 123 });
```

### Custom Compression Support

```javascript
const request = require('superagent');
const zlib = require('zlib');

// Add brotli decompression parser
request.parse['application/brotli'] = function(res, callback) {
  const chunks = [];

  res.on('data', chunk => chunks.push(chunk));
  res.on('end', () => {
    const buffer = Buffer.concat(chunks);
    zlib.brotliDecompress(buffer, (err, decompressed) => {
      if (err) return callback(err);
      try {
        const json = JSON.parse(decompressed.toString());
        callback(null, json);
      } catch (parseErr) {
        callback(parseErr);
      }
    });
  });
};
```

### Streaming Large Responses

```javascript
const request = require('superagent');
const fs = require('fs');

// Disable buffering for large files
request.buffer['application/zip'] = false;
request.buffer['application/x-gzip'] = false;
request.buffer['application/octet-stream'] = false;

// Stream large file download
function downloadLargeFile(url, outputPath) {
  return new Promise((resolve, reject) => {
    request.get(url)
      .pipe(fs.createWriteStream(outputPath))
      .on('finish', resolve)
      .on('error', reject);
  });
}

// Usage
downloadLargeFile('https://api.example.com/large-file.zip', 'output.zip')
  .then(() => console.log('Download complete'))
  .catch(err => console.error('Download failed:', err));
```

## Important Notes

### Serializer Scope

- Global serializers apply to all requests using the specified content type
- Per-request `.serialize()` method overrides global serializers
- Serializers are synchronous functions that return a string or Buffer
- Serializers are called before the request body is sent

### Parser Scope (Node.js)

- Global parsers apply to all responses with the specified content type
- Per-request `.parse()` method overrides global parsers
- Parsers are asynchronous and use callback pattern
- Parsers receive the raw response stream
- Parser errors are caught and passed to request error handlers

### Buffer Control (Node.js)

- Buffering determines whether response body is collected in memory
- Disable buffering for large files to enable streaming
- When buffering is disabled, `.body` may be unavailable
- Use `.pipe()` for streaming when buffering is disabled

### Browser Limitations

- `request.parse` is not available in browsers (parsing happens in XMLHttpRequest)
- `request.buffer` is not available in browsers
- `request.protocols` is not available in browsers
- Only `request.serialize` and `request.types` are available in browsers

### Content-Type Matching

- Content types are matched exactly (e.g., `'application/json'`)
- Wildcards like `'text/*'` work for some parsers
- More specific types take precedence over wildcards
- Character set parameters (e.g., `; charset=utf-8`) are ignored in matching

### Thread Safety

- Global configuration is shared across all requests
- Modifications affect all subsequent requests
- Not recommended to change during concurrent requests
- Consider using per-request configuration for request-specific needs
