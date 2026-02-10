# Streaming

This document covers streaming capabilities in Node.js, including piping responses and writing request bodies as streams for memory-efficient handling of large payloads.

**Note:** Streaming features (`.pipe()` and `.write()`) are only available in Node.js, not in browsers.

## Capabilities

### Piping Responses

Pipe the response body to a writable stream for memory-efficient processing of large responses.

```javascript { .api }
/**
 * Pipe response to a writable stream (Node.js only)
 * @param destination - Writable stream to pipe to
 * @param options - Pipe options (passed to stream.pipe())
 * @returns Destination stream
 */
pipe(destination: Stream, options?: object): Stream;
```

**Behavior:**
- Automatically disables response buffering (`.buffer(false)`)
- Automatically calls `.end()` to execute the request
- Handles automatic decompression (gzip, deflate)
- Follows redirects before piping final response

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Pipe response to file
request
  .get('https://example.com/large-file.zip')
  .pipe(fs.createWriteStream('/path/to/output.zip'))
  .on('finish', () => {
    console.log('Download complete');
  });

// Pipe to multiple destinations using PassThrough
const { PassThrough } = require('stream');
const passThrough = new PassThrough();

request
  .get('https://example.com/data.json')
  .pipe(passThrough);

passThrough.pipe(fs.createWriteStream('copy1.json'));
passThrough.pipe(fs.createWriteStream('copy2.json'));

// Pipe with transformation
const { Transform } = require('stream');

const uppercase = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

request
  .get('https://example.com/text.txt')
  .pipe(uppercase)
  .pipe(fs.createWriteStream('output.txt'));

// Pipe compressed response (automatic decompression)
request
  .get('https://example.com/data.json.gz')
  .pipe(fs.createWriteStream('data.json'));
// SuperAgent automatically decompresses gzip
```

### Pipe with Events

Monitor progress and handle errors when piping.

**Usage Example:**

```javascript
const fs = require('fs');
const request = require('superagent');

const req = request.get('https://example.com/large-file.zip');
const outputStream = fs.createWriteStream('output.zip');

req
  .on('error', err => {
    console.error('Request error:', err);
  })
  .on('response', res => {
    console.log('Status:', res.status);
    console.log('Content-Length:', res.header['content-length']);
  })
  .pipe(outputStream)
  .on('error', err => {
    console.error('Write error:', err);
  })
  .on('finish', () => {
    console.log('Download complete');
  });
```

### Pipe with Authentication and Headers

Combine piping with request configuration.

**Usage Example:**

```javascript
const fs = require('fs');

request
  .get('https://api.example.com/reports/annual.pdf')
  .set('Authorization', 'Bearer token123')
  .query({ format: 'pdf' })
  .on('response', res => {
    if (res.status !== 200) {
      console.error('Download failed:', res.status);
    }
  })
  .pipe(fs.createWriteStream('annual-report.pdf'));
```

### Writing Request Body Streams

Write data to the request body as a stream for uploading large payloads without loading them entirely into memory.

```javascript { .api }
/**
 * Write data to request body stream (Node.js only)
 * @param data - Data chunk to write (string or Buffer)
 * @param encoding - Optional encoding for string data
 * @returns true if data was flushed, false if buffered
 */
write(data: string | Buffer, encoding?: string): boolean;
```

**Behavior:**
- Marks the request as a streaming request
- Data is written to the underlying HTTP request stream
- Must eventually call `.end()` to complete the request

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Write chunks manually
const req = request.post('/api/upload');

req.write('First chunk\n');
req.write('Second chunk\n');
req.write('Third chunk\n');

req.end((err, res) => {
  if (err) {
    console.error('Upload error:', err);
  } else {
    console.log('Upload complete:', res.body);
  }
});

// Stream file upload using write
const stream = fs.createReadStream('/path/to/large-file.txt');

const req = request.post('/api/upload');

stream.on('data', chunk => {
  req.write(chunk);
});

stream.on('end', () => {
  req.end((err, res) => {
    if (err) {
      console.error('Upload error:', err);
    } else {
      console.log('Upload complete:', res.body);
    }
  });
});

stream.on('error', err => {
  console.error('Read error:', err);
  req.abort();
});
```

### Pipe Request from Stream

Pipe a readable stream directly to a request (simpler than using `.write()`).

**Usage Example:**

```javascript
const fs = require('fs');

// Pipe file to upload endpoint
const stream = fs.createReadStream('/path/to/file.txt');
const req = request.post('/api/upload');

stream.pipe(req)
  .on('response', res => {
    console.log('Upload complete:', res.status);
  })
  .on('error', err => {
    console.error('Upload error:', err);
  });
```

**Note:** When piping to a request, you're piping to the underlying HTTP request stream, not the SuperAgent Request object. The response events are still available on the SuperAgent Request.

### Buffer Control

Control whether responses are buffered in memory or streamed.

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param enable - true to buffer (default for most types), false to stream
 * @returns Request instance for chaining
 */
buffer(enable?: boolean): Request;
```

**Default buffering behavior:**
- **Enabled** by default for: JSON, text, form data, images
- **Disabled** when using `.pipe()`
- Can be explicitly controlled with `.buffer(true/false)`

**Usage Examples:**

```javascript
// Disable buffering for large response
request
  .get('/api/large-data')
  .buffer(false)
  .end((err, res) => {
    // Response not buffered in memory
    // res.body will be undefined
    // Use .pipe() instead for large responses
  });

// Enable buffering (default for most types)
request
  .get('/api/data')
  .buffer(true)
  .end((err, res) => {
    console.log('Buffered response:', res.body);
  });

// Pipe without buffering
request
  .get('/api/large-file')
  .buffer(false)  // Redundant when using .pipe(), but explicit
  .pipe(fs.createWriteStream('output.dat'));
```

### Progress Events

Monitor upload and download progress with streaming requests.

```javascript { .api }
/**
 * Progress event data
 */
interface ProgressEvent {
  direction: 'upload' | 'download';
  lengthComputable: boolean;
  loaded: number;
  total: number;
}
```

**Usage Example:**

```javascript
const fs = require('fs');

// Download progress
request
  .get('https://example.com/large-file.zip')
  .on('progress', event => {
    if (event.direction === 'download') {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      console.log(`Downloaded: ${percent}%`);
    }
  })
  .pipe(fs.createWriteStream('output.zip'));

// Upload progress
const stream = fs.createReadStream('/path/to/large-file.zip');

request
  .post('/api/upload')
  .on('progress', event => {
    if (event.direction === 'upload') {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      console.log(`Uploaded: ${percent}%`);
    }
  })
  .on('end', () => {
    console.log('Upload complete');
  });

stream.pipe(request.post('/api/upload'));
```

### Response Type for Binary Data

Set the response type for binary data when not piping.

```javascript { .api }
/**
 * Set response body format
 * @param type - Response type: 'blob', 'arraybuffer', etc.
 * @returns Request instance for chaining
 */
responseType(type: string): Request;
```

**Common types:**
- `'blob'` - Blob (browser only)
- `'arraybuffer'` - ArrayBuffer (browser only)
- In Node.js, binary responses are returned as Buffer by default

**Usage Example:**

```javascript
// Browser: Get binary data as ArrayBuffer
request
  .get('/api/image.png')
  .responseType('arraybuffer')
  .then(res => {
    const buffer = res.body;  // ArrayBuffer
    console.log('Size:', buffer.byteLength);
  });

// Node.js: Binary data is Buffer by default
request
  .get('/api/image.png')
  .then(res => {
    const buffer = res.body;  // Buffer
    console.log('Size:', buffer.length);
  });
```

### Complete Streaming Examples

#### Download Large File with Progress

```javascript
const fs = require('fs');
const request = require('superagent');

const url = 'https://example.com/large-file.zip';
const outputPath = 'download.zip';

request
  .get(url)
  .on('response', res => {
    const totalSize = parseInt(res.header['content-length'], 10);
    console.log(`Total size: ${(totalSize / 1024 / 1024).toFixed(2)} MB`);
  })
  .on('progress', event => {
    if (event.direction === 'download' && event.total) {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      const loaded = (event.loaded / 1024 / 1024).toFixed(2);
      const total = (event.total / 1024 / 1024).toFixed(2);
      console.log(`Progress: ${percent}% (${loaded}/${total} MB)`);
    }
  })
  .on('error', err => {
    console.error('Download failed:', err);
  })
  .pipe(fs.createWriteStream(outputPath))
  .on('finish', () => {
    console.log('Download complete');
  })
  .on('error', err => {
    console.error('Write failed:', err);
  });
```

#### Upload Large File with Progress

```javascript
const fs = require('fs');
const request = require('superagent');

const filePath = '/path/to/large-file.zip';
const uploadUrl = 'https://api.example.com/upload';

const stream = fs.createReadStream(filePath);
const stat = fs.statSync(filePath);
const fileSize = stat.size;

console.log(`Uploading file: ${(fileSize / 1024 / 1024).toFixed(2)} MB`);

const req = request.post(uploadUrl)
  .set('Authorization', 'Bearer token123')
  .on('progress', event => {
    if (event.direction === 'upload') {
      const percent = (event.loaded / event.total * 100).toFixed(2);
      const loaded = (event.loaded / 1024 / 1024).toFixed(2);
      const total = (event.total / 1024 / 1024).toFixed(2);
      console.log(`Upload: ${percent}% (${loaded}/${total} MB)`);
    }
  })
  .on('error', err => {
    console.error('Upload failed:', err);
  });

stream.on('error', err => {
  console.error('Read failed:', err);
  req.abort();
});

stream.pipe(req)
  .on('response', res => {
    console.log('Upload complete:', res.status);
    console.log('Response:', res.body);
  });
```

#### Stream Processing Pipeline

```javascript
const fs = require('fs');
const { Transform } = require('stream');
const request = require('superagent');

// Transform stream to process data
const processStream = new Transform({
  transform(chunk, encoding, callback) {
    // Process chunk (e.g., encryption, compression, transformation)
    const processed = chunk.toString().toUpperCase();
    this.push(processed);
    callback();
  }
});

// Download, transform, and save
request
  .get('https://example.com/data.txt')
  .pipe(processStream)
  .pipe(fs.createWriteStream('processed.txt'))
  .on('finish', () => {
    console.log('Processing complete');
  });
```
