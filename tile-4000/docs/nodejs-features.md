# Node.js Specific Features

Node.js-only capabilities including HTTP/2 support, streams, buffer control, and HTTP agent configuration.

## Capabilities

### HTTP/2 Support

Enable HTTP/2 protocol for requests (requires Node.js 9+ with http2 module).

```javascript { .api }
/**
 * Enable or disable HTTP/2
 * @param enable - True to enable, false to disable (default: true if no argument)
 * @returns Request instance for chaining
 */
Request.prototype.http2(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable HTTP/2
request.get('https://api.example.com/users')
  .http2()
  .then(res => console.log(res.body));

// Explicitly enable
request.get('https://api.example.com/users')
  .http2(true)
  .then(res => console.log(res.body));

// Disable HTTP/2 (use HTTP/1.1)
request.get('https://api.example.com/users')
  .http2(false)
  .then(res => console.log(res.body));

// Error handling if HTTP/2 not supported
try {
  const res = await request.get('https://api.example.com/users')
    .http2();
} catch (err) {
  if (err.message.includes('http2')) {
    console.error('HTTP/2 not supported in this Node.js version');
  }
}
```

### HTTP Agent Configuration

Set custom HTTP/HTTPS agent for connection pooling and advanced options.

```javascript { .api }
/**
 * Set HTTP agent for request
 * @param agent - Node.js http.Agent or https.Agent instance
 * @returns Request instance for chaining
 */
Request.prototype.agent(agent: http.Agent | https.Agent): Request;
```

**Usage Examples:**

```javascript
const http = require('http');
const https = require('https');
const request = require('superagent');

// Custom HTTP agent with connection pooling
const httpAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 60000
});

request.get('http://api.example.com/users')
  .agent(httpAgent)
  .then(res => console.log(res.body));

// Custom HTTPS agent with SSL options
const httpsAgent = new https.Agent({
  keepAlive: true,
  maxSockets: 50,
  rejectUnauthorized: false // Disable cert validation (not recommended in production)
});

request.get('https://api.example.com/users')
  .agent(httpsAgent)
  .then(res => console.log(res.body));

// Reuse agent across multiple requests
const agent = new https.Agent({ keepAlive: true });

await request.get('https://api.example.com/users').agent(agent);
await request.get('https://api.example.com/posts').agent(agent);
await request.get('https://api.example.com/comments').agent(agent);

// Disable agent (new connection each time)
request.get('https://api.example.com/users')
  .agent(false);
```

### Response Streaming

Pipe response data to a writable stream.

```javascript { .api }
/**
 * Pipe response to a writable stream
 * @param stream - Writable stream
 * @param options - Optional pipe options
 * @returns Stream instance
 */
Request.prototype.pipe(stream: WritableStream, options?: object): Stream;
```

**Usage Examples:**

```javascript
const fs = require('fs');
const request = require('superagent');

// Download file and save to disk
request.get('https://api.example.com/file.pdf')
  .pipe(fs.createWriteStream('output.pdf'))
  .on('finish', () => {
    console.log('File downloaded');
  });

// Pipe to multiple streams
const file1 = fs.createWriteStream('copy1.pdf');
const file2 = fs.createWriteStream('copy2.pdf');

const req = request.get('https://api.example.com/file.pdf');
req.pipe(file1);
req.pipe(file2);

// Pipe with error handling
request.get('https://api.example.com/large-file.zip')
  .pipe(fs.createWriteStream('output.zip'))
  .on('error', err => {
    console.error('Download failed:', err.message);
  })
  .on('finish', () => {
    console.log('Download complete');
  });

// Pipe to HTTP response (proxy)
const http = require('http');

http.createServer((req, res) => {
  request.get('https://api.example.com/data')
    .pipe(res);
}).listen(3000);
```

### Request Streaming

Stream request body data for large uploads.

```javascript { .api }
/**
 * Write data to request stream
 * @param data - Data chunk to write (string or Buffer)
 * @param encoding - Optional encoding (default: 'utf8')
 * @returns Boolean indicating if buffer is full
 */
Request.prototype.write(data: string | Buffer, encoding?: string): boolean;
```

**Usage Examples:**

```javascript
const fs = require('fs');
const request = require('superagent');

// Stream file upload
const req = request.post('https://api.example.com/upload');
const stream = fs.createReadStream('large-file.zip');

stream.on('data', chunk => {
  req.write(chunk);
});

stream.on('end', () => {
  req.end((err, res) => {
    if (err) return console.error(err);
    console.log('Upload complete:', res.body);
  });
});

// Manual chunked upload
const req = request.post('https://api.example.com/upload')
  .set('Content-Type', 'application/octet-stream');

req.write(Buffer.from('chunk1'));
req.write(Buffer.from('chunk2'));
req.write(Buffer.from('chunk3'));

req.end((err, res) => {
  if (err) return console.error(err);
  console.log('Upload complete');
});
```

### Buffer Control

Control response body buffering.

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param enable - True to buffer (default), false to disable
 * @returns Request instance for chaining
 */
Request.prototype.buffer(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable buffering (default)
request.get('https://api.example.com/data')
  .buffer(true)
  .then(res => {
    console.log('Buffered body:', res.body);
  });

// Disable buffering for large files
request.get('https://api.example.com/large-file.zip')
  .buffer(false)
  .pipe(fs.createWriteStream('output.zip'));

// Conditional buffering based on content type
request.get(url)
  .buffer(res => {
    // Buffer text and JSON, stream binary
    return res.type === 'application/json' || res.type.startsWith('text/');
  });
```

### File System Operations

Read files from disk for uploads.

**Usage Examples:**

```javascript
const fs = require('fs');
const request = require('superagent');

// Upload file by path
request.post('https://api.example.com/upload')
  .attach('file', '/path/to/document.pdf')
  .then(res => console.log('Uploaded'));

// Upload file buffer
const buffer = fs.readFileSync('/path/to/file.pdf');
request.post('https://api.example.com/upload')
  .attach('file', buffer, 'document.pdf')
  .then(res => console.log('Uploaded'));

// Upload stream
const stream = fs.createReadStream('/path/to/large-file.zip');
request.post('https://api.example.com/upload')
  .attach('file', stream, 'archive.zip')
  .then(res => console.log('Uploaded'));
```

### Protocol Handlers

Access protocol handlers for HTTP, HTTPS, and HTTP/2.

```javascript { .api }
// Available as request.protocols
interface Protocols {
  'http:': typeof http;
  'https:': typeof https;
  'http2:': typeof http2 | undefined;
}
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Access protocol modules
console.log('HTTP module:', request.protocols['http:']);
console.log('HTTPS module:', request.protocols['https:']);
console.log('HTTP/2 module:', request.protocols['http2:']); // May be undefined

// Check HTTP/2 support
if (request.protocols['http2:']) {
  console.log('HTTP/2 is supported');
} else {
  console.log('HTTP/2 is not available');
}
```

## Node.js Patterns

### Efficient File Download

```javascript
const fs = require('fs');
const request = require('superagent');

async function downloadFile(url, destination) {
  return new Promise((resolve, reject) => {
    const stream = fs.createWriteStream(destination);

    request.get(url)
      .buffer(false)
      .pipe(stream)
      .on('finish', () => resolve(destination))
      .on('error', reject);
  });
}

// Usage
await downloadFile('https://api.example.com/file.pdf', 'output.pdf');
```

### Streaming Upload with Progress

```javascript
const fs = require('fs');
const request = require('superagent');

async function uploadFileWithProgress(filePath, url) {
  const stats = fs.statSync(filePath);
  const fileSize = stats.size;
  let uploaded = 0;

  const stream = fs.createReadStream(filePath);
  const req = request.post(url);

  stream.on('data', chunk => {
    uploaded += chunk.length;
    const progress = (uploaded / fileSize * 100).toFixed(2);
    console.log(`Upload progress: ${progress}%`);
    req.write(chunk);
  });

  stream.on('end', () => {
    req.end((err, res) => {
      if (err) return console.error(err);
      console.log('Upload complete');
    });
  });
}

// Usage
await uploadFileWithProgress('/path/to/file.zip', 'https://api.example.com/upload');
```

### HTTP Proxy Server

```javascript
const http = require('http');
const request = require('superagent');

http.createServer((req, res) => {
  const targetUrl = 'https://api.example.com' + req.url;

  request(req.method, targetUrl)
    .set(req.headers)
    .buffer(false)
    .pipe(res)
    .on('error', err => {
      res.statusCode = 500;
      res.end('Proxy error: ' + err.message);
    });
}).listen(8080);

console.log('Proxy server listening on port 8080');
```

### Connection Pool Management

```javascript
const https = require('https');
const request = require('superagent');

// Create agent with connection pooling
const agent = new https.Agent({
  keepAlive: true,
  keepAliveMsecs: 10000,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 60000
});

// Reuse agent for all requests
async function makeRequests() {
  const requests = [];

  for (let i = 0; i < 100; i++) {
    requests.push(
      request.get('https://api.example.com/users')
        .agent(agent)
    );
  }

  const responses = await Promise.all(requests);
  console.log(`Completed ${responses.length} requests`);
}

makeRequests();

// Cleanup agent when done
process.on('exit', () => {
  agent.destroy();
});
```

### Certificate Pinning

```javascript
const https = require('https');
const fs = require('fs');
const request = require('superagent');

// Load trusted certificate
const trustedCert = fs.readFileSync('/path/to/ca-cert.pem');

// Create agent with certificate pinning
const agent = new https.Agent({
  ca: trustedCert,
  checkServerIdentity: (hostname, cert) => {
    // Custom certificate validation
    console.log('Validating cert for:', hostname);
    // Return undefined for valid, Error for invalid
  }
});

// Use agent for requests
request.get('https://api.example.com/users')
  .agent(agent)
  .then(res => console.log(res.body));
```

### Unix Domain Sockets

Connect to servers via Unix domain sockets using special URL format.

**URL Pattern:** `http+unix://SOCKET_PATH/REQUEST_PATH`

- Socket path uses `%2F` to represent `/` characters
- Request path follows the socket path

**Usage Examples:**

```javascript
// Connect to Unix socket
// Socket: /var/run/docker.sock
// Request path: /containers/json
const socketPath = '/var/run/docker.sock'.replace(/\//g, '%2F');
const url = `http+unix://${socketPath}/containers/json`;

request.get(url)
  .then(res => {
    console.log('Docker containers:', res.body);
  });

// Another example
// Socket: /tmp/api.sock
// Request path: /api/users
const socket = '/tmp/api.sock'.replace(/\//g, '%2F');

request.get(`http+unix://${socket}/api/users`)
  .then(res => console.log('Users:', res.body));

// Helper function for Unix socket URLs
function unixUrl(socketPath, requestPath) {
  const encoded = socketPath.replace(/\//g, '%2F');
  return `http+unix://${encoded}${requestPath}`;
}

// Usage
request.get(unixUrl('/var/run/app.sock', '/status'))
  .then(res => console.log('Status:', res.body));
```

### Response Stream Methods

Response objects in Node.js have additional stream methods for advanced control.

```javascript { .api }
/**
 * Pipe response to a writable stream
 */
Response.prototype.pipe(stream: WritableStream): Stream;

/**
 * Pause the response stream
 */
Response.prototype.pause(): Response;

/**
 * Resume the response stream
 */
Response.prototype.resume(): Response;

/**
 * Destroy the response stream
 * @param err - Optional error
 */
Response.prototype.destroy(err?: Error): Response;

/**
 * Set encoding for the response stream
 * @param encoding - String encoding (e.g., 'utf8', 'base64')
 */
Response.prototype.setEncoding(encoding: string): Response;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Pause and resume response stream
request.get('https://api.example.com/stream')
  .end((err, res) => {
    res.pause();

    setTimeout(() => {
      console.log('Resuming stream...');
      res.resume();
    }, 1000);

    res.on('data', chunk => {
      console.log('Received chunk:', chunk.length, 'bytes');
    });

    res.on('end', () => {
      console.log('Stream complete');
    });
  });

// Destroy stream on error condition
request.get('https://api.example.com/data')
  .end((err, res) => {
    res.on('data', chunk => {
      if (/* some error condition */) {
        res.destroy(new Error('Invalid data'));
      }
    });
  });

// Set encoding
request.get('https://api.example.com/text')
  .end((err, res) => {
    res.setEncoding('utf8');
    res.on('data', chunk => {
      console.log('Text chunk:', chunk); // String, not Buffer
    });
  });
```

## Important Notes

### HTTP/2 Requirements

- Requires Node.js 9.0 or later
- HTTP/2 module must be available (some builds may not include it)
- Server must support HTTP/2 (ALPN negotiation)
- HTTPS is required (HTTP/2 over plain HTTP is not supported)

### Streaming Caveats

- Streaming disables automatic response parsing (`.body` may be undefined)
- Error handling must be done on the stream
- Memory-efficient for large files
- Cannot retry streamed requests

### Agent Behavior

- Agents maintain connection pools for better performance
- Setting `.agent(false)` disables connection pooling
- Default agent has connection limits (may need custom agent for high concurrency)
- Always destroy agents when done to release resources

### Buffer Control

- Buffering is enabled by default for parsing
- Disable buffering for large responses to save memory
- When buffering is disabled, must handle response stream manually
- Buffer control affects `.body` availability
