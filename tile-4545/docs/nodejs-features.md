# Node.js Specific Features

Node.js-only features including streams, SSL/TLS, HTTP/2, and custom agents.

## Capabilities

### Stream Support

Pipe responses to streams or pipe streams to requests.

```javascript { .api }
interface Request {
  /**
   * Pipe response to a writable stream
   * @param stream - Destination stream
   * @param options - Pipe options
   * @returns Destination stream
   */
  pipe(stream: WritableStream, options?: object): WritableStream;

  /**
   * Write data to request stream
   * @param data - Data to write
   * @param encoding - Character encoding
   * @returns true if data was handled
   */
  write(data: string | Buffer, encoding?: string): boolean;

  /**
   * Handle data events from response stream
   */
  on(event: 'data', handler: (chunk: Buffer) => void): Request;

  /**
   * Handle drain events
   */
  on(event: 'drain', handler: () => void): Request;
}

interface Response {
  /**
   * Pause the response stream
   */
  pause(): void;

  /**
   * Resume the response stream
   */
  resume(): void;

  /**
   * Destroy the response stream
   * @param err - Optional error
   */
  destroy(err?: Error): void;

  /**
   * Set the character encoding for the response stream
   * @param encoding - Character encoding
   */
  setEncoding(encoding: string): void;

  /**
   * Handle response data events
   */
  on(event: 'data', handler: (chunk: Buffer) => void): Response;

  /**
   * Handle response end events
   */
  on(event: 'end', handler: () => void): Response;

  /**
   * Handle response close events
   */
  on(event: 'close', handler: () => void): Response;

  /**
   * Handle response error events
   */
  on(event: 'error', handler: (err: Error) => void): Response;
}
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Pipe response to file
request
  .get('/api/file.pdf')
  .pipe(fs.createWriteStream('output.pdf'))
  .on('finish', () => {
    console.log('File downloaded');
  });

// Pipe response to multiple streams
const req = request.get('/api/large-file');
req.pipe(fs.createWriteStream('copy1.dat'));
req.pipe(fs.createWriteStream('copy2.dat'));

// Process stream data
request
  .get('/api/data')
  .buffer(false) // Don't buffer response
  .on('data', chunk => {
    console.log('Received chunk:', chunk.length, 'bytes');
    // Process chunk immediately
  })
  .on('end', () => {
    console.log('Stream complete');
  });

// Pipe file to upload
const stream = fs.createReadStream('/path/to/file.dat');
stream.pipe(request.post('/api/upload'));

// Write to request stream
const req = request.post('/api/upload');
req.write('chunk1');
req.write('chunk2');
req.write('chunk3');
req.end(); // Signal end of stream

// Response stream control
request
  .get('/api/stream-data')
  .buffer(false)
  .end((err, res) => {
    // Pause response stream
    res.pause();

    // Process existing data
    setTimeout(() => {
      // Resume after delay
      res.resume();
    }, 1000);

    // Listen for data
    res.on('data', chunk => {
      console.log('Chunk:', chunk);
    });

    res.on('end', () => {
      console.log('Stream ended');
    });

    res.on('close', () => {
      console.log('Stream closed');
    });

    res.on('error', err => {
      console.error('Stream error:', err);
    });

    // Set encoding for text streams
    res.setEncoding('utf8');
  });
```

### SSL/TLS Configuration

Configure SSL/TLS certificates for HTTPS requests.

```javascript { .api }
interface Request {
  /**
   * Set CA certificate(s)
   * @param cert - CA certificate(s) as string, Buffer, or array
   * @returns Request instance for chaining
   */
  ca(cert: string | Buffer | Array<string | Buffer>): Request;

  /**
   * Set client certificate
   * @param cert - Client certificate as string or Buffer
   * @returns Request instance for chaining
   */
  cert(cert: string | Buffer): Request;

  /**
   * Set client private key
   * @param key - Client private key as string or Buffer
   * @returns Request instance for chaining
   */
  key(key: string | Buffer): Request;

  /**
   * Set PFX or PKCS12 encoded private key and certificate chain
   * @param pfx - PFX certificate as string or Buffer
   * @returns Request instance for chaining
   */
  pfx(pfx: string | Buffer): Request;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Set CA certificate
request
  .get('https://secure-api.example.com/data')
  .ca(fs.readFileSync('/path/to/ca-cert.pem'))
  .end((err, res) => {
    console.log(res.body);
  });

// Multiple CA certificates
request
  .get('https://api.example.com/data')
  .ca([
    fs.readFileSync('/path/to/ca1.pem'),
    fs.readFileSync('/path/to/ca2.pem')
  ])
  .end((err, res) => {
    console.log(res.body);
  });

// Client certificate authentication
request
  .get('https://api.example.com/secure')
  .cert(fs.readFileSync('/path/to/client-cert.pem'))
  .key(fs.readFileSync('/path/to/client-key.pem'))
  .end((err, res) => {
    console.log(res.body);
  });

// PFX certificate
request
  .get('https://api.example.com/secure')
  .pfx(fs.readFileSync('/path/to/cert.pfx'))
  .end((err, res) => {
    console.log(res.body);
  });

// Combined SSL configuration
request
  .post('https://secure-api.example.com/data')
  .ca(fs.readFileSync('/path/to/ca.pem'))
  .cert(fs.readFileSync('/path/to/client-cert.pem'))
  .key(fs.readFileSync('/path/to/client-key.pem'))
  .send({ data: 'secure' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### Custom HTTP Agents

Use custom HTTP/HTTPS agents for connection pooling and proxy support.

```javascript { .api }
interface Request {
  /**
   * Set custom HTTP agent
   * @param agent - http.Agent or https.Agent instance
   * @returns Request instance for chaining
   */
  agent(agent: http.Agent | https.Agent): Request;

  /**
   * Get the current HTTP agent
   * @returns Current http.Agent or https.Agent instance
   */
  agent(): http.Agent | https.Agent;
}
```

**Usage Examples:**

```javascript
const http = require('http');
const https = require('https');

// Custom HTTP agent with connection pooling
const httpAgent = new http.Agent({
  keepAlive: true,
  keepAliveMsecs: 10000,
  maxSockets: 50,
  maxFreeSockets: 10
});

request
  .get('http://api.example.com/data')
  .agent(httpAgent)
  .end((err, res) => {
    console.log(res.body);
  });

// Custom HTTPS agent
const httpsAgent = new https.Agent({
  keepAlive: true,
  maxSockets: 20,
  rejectUnauthorized: false // Accept self-signed certificates (use with caution)
});

request
  .get('https://api.example.com/data')
  .agent(httpsAgent)
  .end((err, res) => {
    console.log(res.body);
  });

// Proxy agent (requires https-proxy-agent package)
const HttpsProxyAgent = require('https-proxy-agent');
const proxyAgent = new HttpsProxyAgent('http://proxy.example.com:8080');

request
  .get('https://api.example.com/data')
  .agent(proxyAgent)
  .end((err, res) => {
    console.log(res.body);
  });

// Use global agent
request
  .get('http://api.example.com/data')
  .agent(http.globalAgent)
  .end((err, res) => {
    console.log(res.body);
  });

// Get current agent
const req = request.get('http://api.example.com/data').agent(httpAgent);
console.log(req.agent()); // Returns httpAgent
```

### HTTP/2 Support

Enable HTTP/2 for requests (Node.js 8.4+).

```javascript { .api }
interface Request {
  /**
   * Enable HTTP/2 support
   * @param enable - Enable/disable HTTP/2 (default: true)
   * @returns Request instance for chaining
   */
  http2(enable?: boolean): Request;
}
```

**Usage Examples:**

```javascript
// Enable HTTP/2
request
  .get('https://http2-api.example.com/data')
  .http2()
  .end((err, res) => {
    console.log(res.body);
  });

// Explicitly enable
request
  .get('https://api.example.com/data')
  .http2(true)
  .end((err, res) => {
    console.log(res.body);
  });

// Disable HTTP/2 (use HTTP/1.1)
request
  .get('https://api.example.com/data')
  .http2(false)
  .end((err, res) => {
    console.log(res.body);
  });
```

**Note**: HTTP/2 support requires Node.js 8.4+ and is only available for HTTPS connections.

### Response Buffering

Control response buffering for specific content types.

```javascript { .api }
interface Request {
  /**
   * Enable/disable response buffering
   * @param enable - Whether to buffer the response
   * @returns Request instance for chaining
   */
  buffer(enable: boolean): Request;
}

// Global buffer configuration
request.buffer: {[contentType: string]: boolean};
```

**Usage Examples:**

```javascript
// Disable buffering for large response
request
  .get('/api/large-file')
  .buffer(false)
  .on('data', chunk => {
    console.log('Chunk:', chunk.length);
    // Process chunk immediately without buffering
  })
  .on('end', () => {
    console.log('Stream complete');
  });

// Enable buffering (default)
request
  .get('/api/data')
  .buffer(true)
  .end((err, res) => {
    // res.body contains full buffered response
    console.log(res.body);
  });

// Global configuration for content types
request.buffer['video/mp4'] = false;
request.buffer['application/octet-stream'] = false;

// Now these types won't be buffered by default
request
  .get('/api/video.mp4')
  .on('data', chunk => {
    // Video chunks not buffered
  });
```

### Compression Support

Automatic decompression of gzip and deflate responses.

**Usage Examples:**

```javascript
// Automatic decompression
request
  .get('/api/data')
  .end((err, res) => {
    // Response is automatically decompressed if:
    // - Content-Encoding: gzip
    // - Content-Encoding: deflate
    console.log(res.body); // Decompressed data
  });

// Accept compressed responses
request
  .get('/api/data')
  .set('Accept-Encoding', 'gzip, deflate')
  .end((err, res) => {
    // Server may compress response
    console.log(res.body); // Automatically decompressed
  });
```

### Multipart Response Parsing

Parse multipart form data responses (with formidable).

**Usage Examples:**

```javascript
// Multipart response is automatically parsed
request
  .get('/api/multipart-data')
  .end((err, res) => {
    // res.files contains uploaded files
    console.log('Files:', res.files);

    // res.body contains form fields
    console.log('Fields:', res.body);
  });
```

### Protocol Configuration

Access and configure protocol handlers.

```javascript { .api }
// Protocol map (Node.js only)
request.protocols: {
  'http:': http,
  'https:': https,
  'http2:': http2
};
```

**Usage Examples:**

```javascript
// Access protocol modules
console.log(request.protocols['http:']); // http module
console.log(request.protocols['https:']); // https module
console.log(request.protocols['http2:']); // http2 module (if available)

// Protocols are used internally based on URL scheme
request.get('http://api.example.com/data'); // Uses http
request.get('https://api.example.com/data'); // Uses https
request.get('https://api.example.com/data').http2(); // Uses http2
```

### Unix Domain Sockets

Connect to Unix domain sockets for local inter-process communication.

**Usage Examples:**

```javascript
// Connect to Unix domain socket
// URL format: http+unix://ENCODED_SOCKET_PATH/route
const socketPath = '/var/run/docker.sock';
const encodedPath = socketPath.replace(/\//g, '%2F');

request
  .get(`http+unix://${encodedPath}/info`)
  .end((err, res) => {
    console.log(res.body);
  });

// Another example with Docker API
request
  .get('http+unix://%2Fvar%2Frun%2Fdocker.sock/containers/json')
  .end((err, res) => {
    console.log('Containers:', res.body);
  });

// POST to Unix socket
request
  .post('http+unix://%2Ftmp%2Fmy-app.sock/api/data')
  .send({ message: 'Hello' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### File System Integration

Read files from the file system for uploads.

**Usage Examples:**

```javascript
const fs = require('fs');

// Attach file by path
request
  .post('/api/upload')
  .attach('document', '/path/to/file.pdf')
  .end((err, res) => {
    console.log('Upload complete');
  });

// Attach Buffer
const buffer = fs.readFileSync('/path/to/file.pdf');
request
  .post('/api/upload')
  .attach('document', buffer, 'file.pdf')
  .end((err, res) => {
    console.log('Upload complete');
  });

// Attach stream
const stream = fs.createReadStream('/path/to/large-file.dat');
request
  .post('/api/upload')
  .attach('file', stream)
  .end((err, res) => {
    console.log('Upload complete');
  });
```
