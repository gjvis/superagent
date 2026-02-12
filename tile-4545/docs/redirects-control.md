# Redirects and Request Control

Configure redirect behavior and control request lifecycle.

## Capabilities

### Redirect Configuration

Control how many redirects to follow automatically.

```javascript { .api }
/**
 * Set maximum number of redirects to follow
 * @param count - Maximum redirects (default: 5)
 * @returns Request instance for chaining
 */
redirects(count: number): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Follow up to 10 redirects (default is 5)
request
  .get('/api/redirect-chain')
  .redirects(10)
  .end((err, res) => {
    console.log('Final URL:', res.request.url);
  });

// Disable redirects entirely
request
  .get('/api/no-follow')
  .redirects(0)
  .end((err, res) => {
    // Will return 301/302 response without following
    console.log('Status:', res.status);
    console.log('Location:', res.header.location);
  });

// Default behavior (5 redirects)
request
  .get('/api/data')
  .end((err, res) => {
    // Automatically follows up to 5 redirects
  });
```

**Redirect Details:**

- Automatic redirects for status codes: 301, 302, 303, 305, 307, 308
- Default maximum: 5 redirects
- Exceeding max redirects will result in an error
- On 303 (See Other), method is changed to GET
- On 301/302, POST requests are changed to GET (following browser behavior)
- On 307/308, method is preserved

### Redirect Events

Listen for redirect events to track redirect chains.

**Usage Examples:**

```javascript
request
  .get('/api/redirect-chain')
  .on('redirect', (res) => {
    console.log('Redirected to:', res.headers.location);
    console.log('From status:', res.status);
  })
  .end((err, res) => {
    console.log('Final destination:', res.request.url);
  });
```

### Aborting Requests

Cancel a request in progress.

```javascript { .api }
/**
 * Abort the request
 * @returns Request instance
 */
abort(): Request;
```

**Usage Examples:**

```javascript
// Abort based on timer
const req = request.get('/api/large-file');

setTimeout(() => {
  req.abort();
}, 5000);

req.end((err, res) => {
  if (err && err.code === 'ABORTED') {
    console.log('Request was aborted');
  } else {
    console.log(res.body);
  }
});

// Abort on user action (browser)
const downloadReq = request.get('/api/download');

document.getElementById('cancelBtn').onclick = () => {
  downloadReq.abort();
};

downloadReq
  .on('progress', (e) => {
    console.log('Progress:', e.percent);
  })
  .end((err, res) => {
    if (err) {
      console.log('Download cancelled or failed');
    }
  });

// Conditional abort
const req = request
  .get('/api/data')
  .on('response', (res) => {
    // Abort if response is too large
    const contentLength = parseInt(res.headers['content-length']);
    if (contentLength > 10 * 1024 * 1024) { // 10 MB
      req.abort();
      console.log('Response too large, aborting');
    }
  })
  .end((err, res) => {
    if (err && err.code === 'ABORTED') {
      console.log('Aborted due to size limit');
    }
  });
```

### Maximum Response Size

Limit the maximum size of response body to prevent memory issues.

```javascript { .api }
/**
 * Set maximum response size in bytes
 * @param bytes - Maximum size in bytes
 * @returns Request instance for chaining
 */
maxResponseSize(bytes: number): Request;
```

**Usage Examples:**

```javascript
// Limit response to 5 MB
request
  .get('/api/data')
  .maxResponseSize(5 * 1024 * 1024) // 5 MB
  .end((err, res) => {
    if (err && err.message.includes('maximum size')) {
      console.log('Response exceeded size limit');
    } else {
      console.log(res.body);
    }
  });

// Prevent large uploads
request
  .post('/api/upload')
  .attach('file', '/path/to/file')
  .maxResponseSize(1024 * 1024) // 1 MB response limit
  .end((err, res) => {
    console.log('Upload complete');
  });
```

### Request Lifecycle Events

Listen to events during the request lifecycle.

**Usage Examples:**

```javascript
const req = request.get('/api/data');

// Before request is sent
req.on('request', (httpReq) => {
  console.log('Sending request to:', httpReq.url);
});

// When response starts
req.on('response', (res) => {
  console.log('Response started:', res.status);
});

// On redirect
req.on('redirect', (res) => {
  console.log('Redirecting to:', res.headers.location);
});

// On error
req.on('error', (err) => {
  console.error('Error:', err.message);
});

// When complete
req.on('end', () => {
  console.log('Request complete');
});

// On abort
req.on('abort', () => {
  console.log('Request aborted');
});

// Progress events (upload and download)
req.on('progress', (e) => {
  console.log('Direction:', e.direction); // 'upload' or 'download'
  console.log('Percent:', e.percent);
  console.log('Total:', e.total);
  console.log('Loaded:', e.loaded);
});

// Node.js: data chunks
req.on('data', (chunk) => {
  console.log('Received chunk:', chunk.length, 'bytes');
});

// Node.js: socket drain
req.on('drain', () => {
  console.log('Write buffer drained');
});

req.end((err, res) => {
  console.log('Done');
});
```

### Progress Tracking

Track upload and download progress.

**Usage Examples:**

```javascript
// Track download progress
request
  .get('/api/large-file')
  .on('progress', (event) => {
    if (event.direction === 'download') {
      console.log(`Downloaded: ${event.loaded} / ${event.total} bytes`);
      console.log(`Progress: ${event.percent}%`);

      // Update progress bar
      updateProgressBar(event.percent);
    }
  })
  .end((err, res) => {
    console.log('Download complete');
  });

// Track upload progress
request
  .post('/api/upload')
  .attach('file', largeFile)
  .on('progress', (event) => {
    if (event.direction === 'upload') {
      console.log(`Uploaded: ${event.loaded} / ${event.total} bytes`);
      console.log(`Progress: ${event.percent}%`);
    }
  })
  .end((err, res) => {
    console.log('Upload complete');
  });

// Track both upload and download
request
  .post('/api/process')
  .send(largeData)
  .on('progress', (event) => {
    if (event.direction === 'upload') {
      console.log('Uploading:', event.percent);
    } else {
      console.log('Downloading:', event.percent);
    }
  })
  .end((err, res) => {
    console.log('Complete');
  });
```

### Serialization

Convert request to JSON for debugging or storage.

```javascript { .api }
/**
 * Serialize request to plain object
 * @returns Plain object representation of request
 */
toJSON(): object;
```

**Usage Examples:**

```javascript
const req = request
  .post('/api/users')
  .set('Authorization', 'Bearer token')
  .send({ name: 'John' });

// Serialize to object
const json = req.toJSON();
console.log(json);
// {
//   method: 'POST',
//   url: '/api/users',
//   data: { name: 'John' },
//   headers: { Authorization: 'Bearer token', ... }
// }

// Useful for logging
console.log('Request details:', JSON.stringify(req.toJSON(), null, 2));
```

### Request Properties

The Request object exposes several properties for inspection and advanced use cases.

```javascript { .api }
interface Request {
  // HTTP method (e.g., 'GET', 'POST', etc.)
  method: string;

  // Request URL (settable during configuration)
  url: string;

  // Request headers object (preserves original case)
  header: object;

  // Internal headers object (lowercase keys)
  _header: object;

  // Query string parameters object
  qs: object;

  // Raw query strings array
  _query: string[];

  // Cookie string
  cookies: string;
}
```

**Usage Examples:**

```javascript
const req = request
  .get('/api/users')
  .query({ role: 'admin', active: true })
  .set('X-Custom-Header', 'value');

// Access request properties
console.log(req.method);  // 'GET'
console.log(req.url);     // '/api/users'
console.log(req.qs);      // { role: 'admin', active: true }
console.log(req.header);  // { 'X-Custom-Header': 'value', ... }

// Modify URL during configuration
req.url = '/api/admins';

// Access internal properties for debugging
console.log(req._query);  // ['role=admin&active=true']
```

### EventEmitter API

The Request object extends EventEmitter and provides full event handling capabilities.

```javascript { .api }
/**
 * Attach event listener
 * @param event - Event name
 * @param callback - Event handler function
 * @returns Request instance for chaining
 */
on(event: string, callback: Function): Request;

/**
 * Attach one-time event listener
 * @param event - Event name
 * @param callback - Event handler function (called once only)
 * @returns Request instance for chaining
 */
once(event: string, callback: Function): Request;

/**
 * Emit an event
 * @param event - Event name
 * @param args - Arguments to pass to listeners
 * @returns Boolean indicating if event had listeners
 */
emit(event: string, ...args: any[]): boolean;

/**
 * Get array of listeners for an event
 * @param event - Event name
 * @returns Array of listener functions
 */
listeners(event: string): Function[];

/**
 * Check if event has listeners
 * @param event - Event name
 * @returns Boolean indicating if event has listeners
 */
hasListeners(event: string): boolean;
```

**Available Events:**

- `'request'` - Fired before sending request
- `'response'` - Fired when response is received
- `'redirect'` - Fired on redirect
- `'error'` - Fired on error
- `'end'` - Fired when request completes
- `'abort'` - Fired when request is aborted
- `'progress'` - Fired with upload/download progress
- `'drain'` - Fired when writable stream drains (Node.js)
- `'data'` - Fired with response data chunks (Node.js)
- `'close'` - Fired when connection closes (Node.js)

**Usage Examples:**

```javascript
const req = request.get('/api/data');

// Attach multiple listeners to same event
req.on('progress', (e) => console.log('Listener 1:', e.percent));
req.on('progress', (e) => console.log('Listener 2:', e.percent));

// One-time listener
req.once('response', (res) => {
  console.log('This will only log once:', res.status);
});

// Check if event has listeners
if (req.hasListeners('progress')) {
  console.log('Progress listeners attached');
}

// Get all listeners for an event
const progressListeners = req.listeners('progress');
console.log('Number of progress listeners:', progressListeners.length);

req.end();
```

### Progress Event Object

The progress event emits an object with detailed transfer information.

```javascript { .api }
interface ProgressEvent {
  // Transfer direction
  direction: 'upload' | 'download';

  // Bytes transferred so far
  loaded: number;

  // Total bytes (if known, otherwise may be undefined)
  total: number | undefined;

  // Percentage complete (0-100, if total is known)
  percent: number | undefined;
}
```

**Usage Examples:**

```javascript
request
  .post('/api/upload')
  .attach('file', largeFile)
  .on('progress', (event) => {
    console.log('Direction:', event.direction);
    console.log('Loaded:', event.loaded, 'bytes');

    if (event.total) {
      console.log('Total:', event.total, 'bytes');
      console.log('Percent:', event.percent, '%');
    } else {
      console.log('Total size unknown');
    }
  });
```
