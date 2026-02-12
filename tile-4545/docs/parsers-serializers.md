# Parsers and Serializers

Extend SuperAgent with custom content-type parsers and request serializers.

## Capabilities

### Custom Response Parsers

Override automatic parsing for specific requests or globally.

```javascript { .api }
/**
 * Set custom response parser for this request
 * @param fn - Parser function
 * @returns Request instance for chaining
 */
parse(fn: (res: Response, callback: (err: Error | null, body: any) => void) => void): Request;

// Global parser configuration
request.parse: {
  [contentType: string]: (res: Response, callback: (err: Error | null, body: any) => void) => void
};
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Custom parser for single request
request
  .get('/api/data.xml')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk.toString();
    });
    res.on('end', () => {
      try {
        // Custom XML parsing logic
        const parsed = parseXML(data);
        callback(null, parsed);
      } catch (err) {
        callback(err, null);
      }
    });
  })
  .end((err, res) => {
    console.log(res.body); // Parsed XML
  });

// Custom CSV parser
request
  .get('/api/data.csv')
  .parse((res, callback) => {
    let csv = '';
    res.setEncoding('utf8');
    res.on('data', chunk => csv += chunk);
    res.on('end', () => {
      const lines = csv.split('\n');
      const headers = lines[0].split(',');
      const data = lines.slice(1).map(line => {
        const values = line.split(',');
        return headers.reduce((obj, header, i) => {
          obj[header] = values[i];
          return obj;
        }, {});
      });
      callback(null, data);
    });
  })
  .end((err, res) => {
    console.log(res.body); // Array of objects
  });
```

### Global Parser Registration

Register parsers globally for specific content types.

**Usage Examples:**

```javascript
// Register global XML parser
request.parse['application/xml'] = function(res, fn) {
  let data = '';
  res.on('data', chunk => data += chunk.toString());
  res.on('end', () => {
    try {
      fn(null, parseXML(data));
    } catch (err) {
      fn(err);
    }
  });
};

// Now all XML responses use this parser
request
  .get('/api/data.xml')
  .end((err, res) => {
    console.log(res.body); // Parsed using custom XML parser
  });

// Register CSV parser
request.parse['text/csv'] = function(res, fn) {
  let csv = '';
  res.setEncoding('utf8');
  res.on('data', chunk => csv += chunk);
  res.on('end', () => {
    const parsed = parseCSV(csv);
    fn(null, parsed);
  });
};

// Register YAML parser
request.parse['application/yaml'] = function(res, fn) {
  let yaml = '';
  res.on('data', chunk => yaml += chunk);
  res.on('end', () => {
    try {
      fn(null, YAML.parse(yaml));
    } catch (err) {
      fn(err);
    }
  });
};
```

### Built-in Parsers

SuperAgent includes default parsers for common content types.

**Node.js default parsers:**

```javascript
{
  'application/json': jsonParser,
  'application/x-www-form-urlencoded': urlencodedParser,
  'text/plain': textParser,
  'application/octet-stream': imageParser,
  'application/pdf': imageParser,
  'image/*': imageParser
}
```

**Browser default parsers:**

```javascript
{
  'application/json': JSON.parse,
  'application/x-www-form-urlencoded': parseString
}
```

### Custom Request Serializers

Override how request body data is serialized.

```javascript { .api }
/**
 * Set custom request serializer for this request
 * @param fn - Serializer function
 * @returns Request instance for chaining
 */
serialize(fn: (obj: any) => string): Request;

// Global serializer configuration
request.serialize: {
  [contentType: string]: (obj: any) => string
};
```

**Usage Examples:**

```javascript
// Custom serializer for single request
request
  .post('/api/data')
  .type('application/x-custom')
  .serialize(obj => {
    // Custom serialization logic
    return Object.keys(obj)
      .map(k => `${k}:${obj[k]}`)
      .join('|');
  })
  .send({ a: 1, b: 2, c: 3 })
  .end((err, res) => {
    // Sends: "a:1|b:2|c:3"
  });

// YAML serializer
request
  .post('/api/config')
  .type('application/yaml')
  .serialize(obj => {
    return YAML.stringify(obj);
  })
  .send({ config: { port: 8080, host: 'localhost' } });
```

### Global Serializer Registration

Register serializers globally for specific content types.

**Usage Examples:**

```javascript
// Register global XML serializer
request.serialize['application/xml'] = function(obj) {
  return convertToXML(obj);
};

// Now all requests with XML content-type use this serializer
request
  .post('/api/data')
  .type('application/xml')
  .send({ user: { name: 'John', email: 'john@example.com' } })
  .end((err, res) => {
    // Sends XML representation
  });

// Register YAML serializer
request.serialize['application/yaml'] = function(obj) {
  return YAML.stringify(obj);
};

// Register custom format
request.serialize['application/x-msgpack'] = function(obj) {
  return msgpack.encode(obj);
};
```

### Built-in Serializers

SuperAgent includes default serializers for common content types.

```javascript
{
  'application/json': JSON.stringify,
  'application/x-www-form-urlencoded': qs.stringify
}
```

### Buffer Configuration (Node.js)

Control response buffering for specific content types.

```javascript { .api }
/**
 * Enable/disable response buffering for this request
 * @param enable - Whether to buffer the response
 * @returns Request instance for chaining
 */
buffer(enable: boolean): Request;

// Global buffer configuration (Node.js only)
request.buffer: {
  [contentType: string]: boolean
};
```

**Usage Examples (Node.js):**

```javascript
// Disable buffering for large response
request
  .get('/api/large-file')
  .buffer(false)
  .end((err, res) => {
    // res.body will be empty
    // Use res.on('data') to handle chunks
  });

// Handle streaming without buffering
request
  .get('/api/stream')
  .buffer(false)
  .on('data', chunk => {
    console.log('Received chunk:', chunk.length);
    // Process chunk immediately
  })
  .on('end', () => {
    console.log('Stream complete');
  });

// Global buffer configuration
request.buffer['application/octet-stream'] = false;
request.buffer['video/mp4'] = false;

// Now these content types won't be buffered by default
request
  .get('/api/video.mp4')
  .on('data', chunk => {
    // Process video chunk
  });
```

### Response Type (Browser)

Specify how binary responses should be handled in the browser.

```javascript { .api }
/**
 * Set binary response type (Browser only)
 * @param type - Response type
 * @returns Request instance for chaining
 */
responseType(type: 'blob' | 'arraybuffer'): Request;
```

**Usage Examples (Browser):**

```javascript
// Get image as Blob
request
  .get('/api/image.png')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    document.getElementById('img').src = url;
  });

// Get binary data as ArrayBuffer
request
  .get('/api/data.bin')
  .responseType('arraybuffer')
  .end((err, res) => {
    const buffer = res.body; // ArrayBuffer
    const view = new Uint8Array(buffer);
    // Process binary data
  });

// Download file as Blob
request
  .get('/api/download/report.pdf')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'report.pdf';
    a.click();
  });
```

## Example: Complete Custom Format Support

Add support for a custom format with both parser and serializer.

**Usage Example:**

```javascript
const msgpack = require('msgpack');

// Register MessagePack serializer
request.serialize['application/x-msgpack'] = function(obj) {
  return msgpack.pack(obj);
};

// Register MessagePack parser
request.parse['application/x-msgpack'] = function(res, fn) {
  const chunks = [];
  res.on('data', chunk => chunks.push(chunk));
  res.on('end', () => {
    try {
      const buffer = Buffer.concat(chunks);
      const data = msgpack.unpack(buffer);
      fn(null, data);
    } catch (err) {
      fn(err);
    }
  });
};

// Now use MessagePack for requests and responses
request
  .post('/api/data')
  .type('application/x-msgpack')
  .accept('application/x-msgpack')
  .send({ user: 'John', data: [1, 2, 3] })
  .end((err, res) => {
    console.log(res.body); // Automatically parsed from msgpack
  });
```
