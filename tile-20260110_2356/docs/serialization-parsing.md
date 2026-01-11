# Serialization and Parsing

Configure custom serializers for request bodies and parsers for response bodies with global and per-request settings.

## Capabilities

### Request Serialization

Control how request bodies are serialized based on Content-Type.

```javascript { .api }
/**
 * Set custom serializer for this request
 * @param {Function} serializer - Serializer function (data) => string
 * @returns {Request} Request instance for chaining
 */
Request.prototype.serialize = function(serializer);

/**
 * Global serializer map by content-type
 */
interface SerializeMap {
  [contentType: string]: (data: any) => string;
}
const request.serialize: SerializeMap;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Custom serializer for single request
request
  .post('/api/data')
  .type('application/json')
  .serialize(data => {
    // Custom JSON serialization
    return JSON.stringify(data, null, 2);
  })
  .send({ name: 'John' })
  .end(callback);

// XML serializer for single request
request
  .post('/api/data')
  .type('application/xml')
  .serialize(data => {
    return `<user><name>${data.name}</name></user>`;
  })
  .send({ name: 'John' })
  .end(callback);

// CSV serializer
request
  .post('/api/data')
  .type('text/csv')
  .serialize(data => {
    const headers = Object.keys(data[0]).join(',');
    const rows = data.map(row =>
      Object.values(row).join(',')
    ).join('\n');
    return `${headers}\n${rows}`;
  })
  .send([
    { name: 'John', age: 30 },
    { name: 'Jane', age: 25 }
  ])
  .end(callback);
```

### Global Serializers

Register global serializers for specific content types.

**Usage Examples:**

```javascript
// Add XML serializer globally
request.serialize['application/xml'] = function(obj) {
  // Simple XML serialization
  const xml = Object.keys(obj).map(key =>
    `<${key}>${obj[key]}</${key}>`
  ).join('');
  return `<?xml version="1.0"?><root>${xml}</root>`;
};

// Now all requests with XML content-type use this serializer
request
  .post('/api/data')
  .type('application/xml')
  .send({ name: 'John', email: 'john@example.com' })
  .end(callback);
// Sends: <?xml version="1.0"?><root><name>John</name><email>john@example.com</email></root>

// Add CSV serializer globally
request.serialize['text/csv'] = function(data) {
  if (!Array.isArray(data)) {
    data = [data];
  }
  const headers = Object.keys(data[0]).join(',');
  const rows = data.map(row =>
    Object.values(row).join(',')
  ).join('\n');
  return `${headers}\n${rows}`;
};

// Add YAML serializer globally (requires js-yaml)
const YAML = require('js-yaml');
request.serialize['application/yaml'] = function(obj) {
  return YAML.dump(obj);
};

// Add custom form serializer
request.serialize['application/x-www-form-urlencoded'] = function(obj) {
  return Object.keys(obj)
    .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(obj[key])}`)
    .join('&');
};
```

### Built-in Serializers

SuperAgent includes built-in serializers for common formats.

**Usage Examples:**

```javascript
// Built-in JSON serializer
// Content-Type: application/json
request.serialize['application/json'] = JSON.stringify;

// Built-in form serializer (Node.js)
// Content-Type: application/x-www-form-urlencoded
request.serialize['application/x-www-form-urlencoded'] = require('qs').stringify;

// Built-in form serializer (Browser)
// Content-Type: application/x-www-form-urlencoded
request.serialize['application/x-www-form-urlencoded'] = function(obj) {
  const pairs = [];
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      pairs.push(encodeURIComponent(key) + '=' + encodeURIComponent(obj[key]));
    }
  }
  return pairs.join('&');
};

// These are used automatically based on Content-Type
request
  .post('/api/data')
  .type('json')  // Uses JSON.stringify
  .send({ name: 'John' })
  .end(callback);

request
  .post('/api/form')
  .type('form')  // Uses form serializer
  .send({ name: 'John', email: 'john@example.com' })
  .end(callback);
```

### Response Parsing

Control how response bodies are parsed based on Content-Type.

```javascript { .api }
/**
 * Set custom parser for this request
 * @param {Function} parser - Parser function (res, callback) => void
 * @returns {Request} Request instance for chaining
 */
Request.prototype.parse = function(parser);

/**
 * Global parser map by content-type (Node.js)
 */
interface ParseMap {
  [contentType: string]: (res: Response, callback: (err: Error | null, body: any) => void) => void;
}
const request.parse: ParseMap;
```

**Usage Examples:**

```javascript
// Custom parser for single request
request
  .get('/api/data.xml')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      try {
        // Custom XML parsing
        const parsed = parseXML(data);
        callback(null, parsed);
      } catch (err) {
        callback(err);
      }
    });
  })
  .end((err, res) => {
    console.log('Parsed XML:', res.body);
  });

// CSV parser for single request
request
  .get('/api/data.csv')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      const lines = data.split('\n');
      const headers = lines[0].split(',');
      const rows = lines.slice(1).map(line => {
        const values = line.split(',');
        const obj = {};
        headers.forEach((header, i) => {
          obj[header] = values[i];
        });
        return obj;
      });
      callback(null, rows);
    });
  })
  .end((err, res) => {
    console.log('Parsed CSV:', res.body);
  });

// Binary parser
request
  .get('/api/image.png')
  .parse((res, callback) => {
    const chunks = [];
    res.on('data', chunk => {
      chunks.push(chunk);
    });
    res.on('end', () => {
      const buffer = Buffer.concat(chunks);
      callback(null, buffer);
    });
  })
  .end((err, res) => {
    console.log('Image buffer:', res.body);
  });
```

### Global Parsers (Node.js)

Register global parsers for specific content types in Node.js.

**Usage Examples:**

```javascript
const request = require('superagent');

// Add XML parser globally
const parseXML = require('xml2js').parseString;

request.parse['application/xml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    parseXML(data, callback);
  });
};

// Now all XML responses are automatically parsed
request
  .get('/api/data.xml')
  .end((err, res) => {
    console.log('Parsed XML:', res.body);
  });

// Add YAML parser globally
const YAML = require('js-yaml');

request.parse['application/yaml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    try {
      const parsed = YAML.load(data);
      callback(null, parsed);
    } catch (err) {
      callback(err);
    }
  });
};

// Add CSV parser globally
request.parse['text/csv'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    const lines = data.trim().split('\n');
    const headers = lines[0].split(',');
    const rows = lines.slice(1).map(line => {
      const values = line.split(',');
      const obj = {};
      headers.forEach((header, i) => {
        obj[header.trim()] = values[i] ? values[i].trim() : '';
      });
      return obj;
    });
    callback(null, rows);
  });
};

// Add MessagePack parser
const msgpack = require('msgpack-lite');

request.parse['application/msgpack'] = function(res, callback) {
  const chunks = [];
  res.on('data', chunk => {
    chunks.push(chunk);
  });
  res.on('end', () => {
    try {
      const buffer = Buffer.concat(chunks);
      const decoded = msgpack.decode(buffer);
      callback(null, decoded);
    } catch (err) {
      callback(err);
    }
  });
};
```

### Built-in Parsers

SuperAgent includes built-in parsers for common formats.

**Usage Examples:**

```javascript
// Node.js built-in parsers:

// JSON parser
request.parse['application/json'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    try {
      const parsed = JSON.parse(data);
      callback(null, parsed);
    } catch (err) {
      callback(err);
    }
  });
};

// Form data parser
request.parse['application/x-www-form-urlencoded'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    const parsed = require('qs').parse(data);
    callback(null, parsed);
  });
};

// Text parser
request.parse.text = function(res, callback) {
  let data = '';
  res.setEncoding('utf8');
  res.on('data', chunk => {
    data += chunk;
  });
  res.on('end', () => {
    callback(null, data);
  });
};

// Binary parsers (images, PDFs, etc.)
request.parse.image = function(res, callback) {
  const chunks = [];
  res.on('data', chunk => {
    chunks.push(chunk);
  });
  res.on('end', () => {
    callback(null, Buffer.concat(chunks));
  });
};

request.parse['application/octet-stream'] = request.parse.image;
request.parse['application/pdf'] = request.parse.image;

// Browser built-in parsers:

// JSON parser (browser)
request.parse['application/json'] = function(res, callback) {
  try {
    callback(null, JSON.parse(res.text));
  } catch (err) {
    callback(err);
  }
};

// Form data parser (browser)
request.parse['application/x-www-form-urlencoded'] = function(res, callback) {
  const obj = {};
  const pairs = res.text.split('&');
  pairs.forEach(pair => {
    const [key, value] = pair.split('=');
    obj[decodeURIComponent(key)] = decodeURIComponent(value);
  });
  callback(null, obj);
};
```

### Buffer Control (Node.js)

Configure response buffering behavior.

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param {boolean} [enable] - Enable buffering (default: true)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.buffer = function(enable);

/**
 * Global buffer configuration by content-type
 */
interface BufferMap {
  [contentType: string]: boolean;
}
const request.buffer: BufferMap;
```

**Usage Examples:**

```javascript
// Disable buffering for streaming
request
  .get('/api/large-file')
  .buffer(false)
  .end((err, res) => {
    // res.body is undefined
    // Use streaming instead
  });

// Enable buffering (default)
request
  .get('/api/data')
  .buffer(true)
  .end((err, res) => {
    console.log('Buffered body:', res.body);
  });

// Configure global buffering by content-type
request.buffer['application/json'] = true;
request.buffer['text/plain'] = true;
request.buffer['application/octet-stream'] = true;
request.buffer['image/*'] = true;

// Disable buffering for video
request.buffer['video/*'] = false;
```

### Response Type (Browser)

Set expected response type for binary data in browsers.

```javascript { .api }
/**
 * Set response type (Browser)
 * @param {string} type - Response type ('blob', 'arraybuffer')
 * @returns {Request} Request instance for chaining
 */
Request.prototype.responseType = function(type);
```

**Usage Examples:**

```javascript
// Browser: Get Blob response
request
  .get('/api/image.png')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    document.getElementById('image').src = url;
  });

// Browser: Get ArrayBuffer response
request
  .get('/api/binary-data')
  .responseType('arraybuffer')
  .end((err, res) => {
    const buffer = res.body;
    const view = new DataView(buffer);
    console.log('First byte:', view.getUint8(0));
  });

// Node.js: responseType results in Buffer
request
  .get('/api/image.png')
  .responseType('blob')  // Any value results in Buffer
  .end((err, res) => {
    const buffer = res.body;  // Buffer instance
    fs.writeFileSync('image.png', buffer);
  });
```

### Type Shortcuts (Browser)

Map of content-type shortcuts in browser.

**Usage Examples:**

```javascript
// Browser type shortcuts
request.types = {
  html: 'text/html',
  json: 'application/json',
  xml: 'text/xml',
  urlencoded: 'application/x-www-form-urlencoded',
  form: 'application/x-www-form-urlencoded',
  'form-data': 'application/x-www-form-urlencoded'
};

// Use shortcuts
request
  .post('/api/data')
  .type('json')  // Sets Content-Type: application/json
  .send({ name: 'John' })
  .end(callback);

request
  .get('/api/page')
  .accept('html')  // Sets Accept: text/html
  .end(callback);

// Add custom shortcuts
request.types.yaml = 'application/yaml';
request.types.csv = 'text/csv';

request
  .post('/api/data')
  .type('yaml')
  .send(data)
  .end(callback);
```

### Complete Serialization/Parsing Example

Comprehensive example showing custom serializers and parsers.

**Usage Examples:**

```javascript
const request = require('superagent');
const xml2js = require('xml2js');
const YAML = require('js-yaml');

// Configure global XML serializer
request.serialize['application/xml'] = function(obj) {
  const builder = new xml2js.Builder();
  return builder.buildObject(obj);
};

// Configure global XML parser
request.parse['application/xml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    xml2js.parseString(data, callback);
  });
};

// Configure global YAML serializer
request.serialize['application/yaml'] = function(obj) {
  return YAML.dump(obj);
};

// Configure global YAML parser
request.parse['application/yaml'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    try {
      const parsed = YAML.load(data);
      callback(null, parsed);
    } catch (err) {
      callback(err);
    }
  });
};

// Configure global CSV parser
request.parse['text/csv'] = function(res, callback) {
  let data = '';
  res.on('data', chunk => {
    data += chunk.toString();
  });
  res.on('end', () => {
    const lines = data.trim().split('\n');
    const headers = lines[0].split(',').map(h => h.trim());
    const rows = lines.slice(1).map(line => {
      const values = line.split(',').map(v => v.trim());
      const obj = {};
      headers.forEach((header, i) => {
        obj[header] = values[i] || '';
      });
      return obj;
    });
    callback(null, rows);
  });
};

// Usage with different formats

// Send and receive XML
async function xmlExample() {
  const userData = {
    user: {
      name: 'John',
      email: 'john@example.com',
      age: 30
    }
  };

  const res = await request
    .post('/api/users.xml')
    .type('application/xml')
    .accept('application/xml')
    .send(userData);

  console.log('Response:', res.body);
}

// Send and receive YAML
async function yamlExample() {
  const config = {
    database: {
      host: 'localhost',
      port: 5432,
      name: 'mydb'
    }
  };

  const res = await request
    .post('/api/config.yaml')
    .type('application/yaml')
    .accept('application/yaml')
    .send(config);

  console.log('Response:', res.body);
}

// Receive CSV
async function csvExample() {
  const res = await request
    .get('/api/users.csv')
    .accept('text/csv');

  console.log('Users:', res.body);
  // [
  //   { name: 'John', email: 'john@example.com', age: '30' },
  //   { name: 'Jane', email: 'jane@example.com', age: '25' }
  // ]
}

// Custom per-request parser
async function customParserExample() {
  const res = await request
    .get('/api/custom-format')
    .parse((res, callback) => {
      let data = '';
      res.on('data', chunk => {
        data += chunk.toString();
      });
      res.on('end', () => {
        // Custom parsing logic
        const parsed = data
          .split('\n')
          .map(line => line.trim())
          .filter(line => line.length > 0);
        callback(null, parsed);
      });
    });

  console.log('Parsed data:', res.body);
}

// Binary data handling
async function binaryExample() {
  // Download image
  const imgRes = await request
    .get('/api/image.png')
    .responseType('blob');

  const imageBuffer = imgRes.body;

  // Upload image
  await request
    .post('/api/upload')
    .attach('image', imageBuffer, 'image.png');
}
```

### Important Notes

- **Content-Type Based**: Serializers and parsers are automatically selected based on the Content-Type header.
- **Global vs Request**: Global serializers/parsers apply to all requests. Request-level serializers/parsers override global settings.
- **Node.js vs Browser**: Parser implementation differs between Node.js (stream-based) and browser (text-based).
- **Error Handling**: Parser errors should be passed to the callback. They will be available as `err.parse`.
- **Built-in Support**: JSON and form data are supported out of the box. Add custom serializers/parsers for other formats.
- **Buffering**: Response buffering must be enabled (default) for parsers to work. Disable buffering only for streaming.
- **Response Type**: Use `responseType()` in browsers for binary data (Blob, ArrayBuffer). In Node.js, it results in Buffer.
- **Type Shortcuts**: Browser provides shortcuts for common MIME types. These can be extended with custom mappings.
