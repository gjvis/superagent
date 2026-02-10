# Events and Progress Tracking

SuperAgent emits events during the request lifecycle and provides progress tracking for uploads and downloads.

## Capabilities

### Request Events

Request instances emit events for various lifecycle stages.

```javascript { .api }
/**
 * Request lifecycle events
 */
Request.prototype.on(event: 'request', listener: (req: Request) => void): Request;
Request.prototype.on(event: 'response', listener: (res: Response) => void): Request;
Request.prototype.on(event: 'error', listener: (err: Error) => void): Request;
Request.prototype.on(event: 'abort', listener: () => void): Request;
Request.prototype.on(event: 'redirect', listener: (res: Response) => void): Request;
Request.prototype.on(event: 'drain', listener: () => void): Request;
Request.prototype.on(event: 'end', listener: () => void): Request;
Request.prototype.on(event: 'progress', listener: (event: ProgressEvent) => void): Request;
```

**Event Types:**

- **`request`**: Emitted when the request is sent to the server
- **`response`**: Emitted when a response is received from the server
- **`error`**: Emitted when an error occurs during the request
- **`abort`**: Emitted when the request is aborted
- **`redirect`**: Emitted when the request follows a redirect (Node.js only)
- **`drain`**: Emitted when the socket is drained (Node.js only)
- **`end`**: Emitted when the request/response cycle completes
- **`progress`**: Emitted during upload and download progress

**Usage Examples:**

```javascript
const req = request.get('https://api.example.com/users');

// Track request lifecycle
req.on('request', () => {
  console.log('Request sent');
});

req.on('response', res => {
  console.log('Response received:', res.status);
});

req.on('error', err => {
  console.error('Request error:', err.message);
});

req.on('end', () => {
  console.log('Request complete');
});

// Execute the request
req.then(res => console.log('Success:', res.body));
```

### Progress Tracking

Monitor upload and download progress using the `progress` event.

```javascript { .api }
/**
 * Progress event object
 */
interface ProgressEvent {
  /** Direction of data transfer: 'upload' or 'download' */
  direction: 'upload' | 'download';

  /** Completion percentage (0-100) */
  percent: number;

  /** Bytes transferred so far */
  loaded: number;

  /** Total bytes to transfer (if known) */
  total: number;

  /** Whether the total length is computable */
  lengthComputable: boolean;
}
```

**Usage Examples:**

```javascript
// Track download progress
request.get('https://api.example.com/large-file.zip')
  .on('progress', event => {
    if (event.direction === 'download') {
      console.log('Download progress:', event.percent.toFixed(2) + '%');
      console.log('Downloaded:', event.loaded, 'of', event.total, 'bytes');
    }
  })
  .then(res => {
    console.log('Download complete');
  });

// Track upload progress
const fs = require('fs');

request.post('https://api.example.com/upload')
  .attach('file', '/path/to/large-file.zip')
  .on('progress', event => {
    if (event.direction === 'upload') {
      console.log('Upload progress:', event.percent.toFixed(2) + '%');
      console.log('Uploaded:', event.loaded, 'of', event.total, 'bytes');

      if (!event.lengthComputable) {
        console.log('Total size unknown');
      }
    }
  })
  .then(res => {
    console.log('Upload complete:', res.body);
  });
```

### Upload Progress (Browser)

Browser file uploads with UI feedback.

**Usage Examples:**

```javascript
// Upload with progress bar
const fileInput = document.querySelector('input[type="file"]');
const progressBar = document.querySelector('.progress-bar');
const progressText = document.querySelector('.progress-text');

fileInput.addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (!file) return;

  request.post('https://api.example.com/upload')
    .attach('file', file)
    .on('progress', event => {
      if (event.direction === 'upload') {
        const percent = Math.round(event.percent);
        progressBar.style.width = percent + '%';
        progressText.textContent = percent + '%';
      }
    })
    .then(res => {
      console.log('Upload successful:', res.body);
      progressText.textContent = 'Complete!';
    })
    .catch(err => {
      console.error('Upload failed:', err.message);
      progressText.textContent = 'Failed';
    });
});
```

### Download Progress (Browser)

Track download progress for large files.

**Usage Examples:**

```javascript
// Download with progress indicator
const progressIndicator = document.querySelector('#download-progress');

request.get('https://api.example.com/large-file.pdf')
  .responseType('blob')
  .on('progress', event => {
    if (event.direction === 'download' && event.lengthComputable) {
      const percent = Math.round(event.percent);
      progressIndicator.textContent = `Downloading: ${percent}%`;
    }
  })
  .then(res => {
    progressIndicator.textContent = 'Download complete';

    // Save file
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'document.pdf';
    a.click();
    URL.revokeObjectURL(url);
  })
  .catch(err => {
    progressIndicator.textContent = 'Download failed';
  });
```

### Error Event Handling

Handle errors using the error event.

**Usage Examples:**

```javascript
request.get('https://api.example.com/users')
  .on('error', err => {
    // Handle network errors, timeouts, etc.
    if (err.timeout) {
      console.error('Request timed out');
    } else if (err.code === 'ECONNREFUSED') {
      console.error('Connection refused');
    } else if (err.status) {
      console.error('HTTP error:', err.status);
    } else {
      console.error('Network error:', err.message);
    }
  })
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    // This also catches errors (redundant with error event)
    console.error('Caught error:', err.message);
  });
```

### Redirect Events (Node.js)

Track redirects in Node.js.

**Usage Examples:**

```javascript
const redirectChain = [];

request.get('https://api.example.com/redirect')
  .on('redirect', res => {
    console.log('Redirected to:', res.headers.location);
    redirectChain.push(res.headers.location);
  })
  .on('response', res => {
    console.log('Final response:', res.status);
    console.log('Redirect chain:', redirectChain);
  })
  .then(res => {
    console.log('Final body:', res.body);
  });
```

### Abort Event

Handle request abortion.

**Usage Examples:**

```javascript
const req = request.get('https://api.example.com/large-data')
  .on('abort', () => {
    console.log('Request was aborted');
  })
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    if (err.code === 'ABORTED') {
      console.log('Request cancelled');
    }
  });

// Abort after 5 seconds
setTimeout(() => {
  req.abort();
}, 5000);
```

## Event Patterns

### Comprehensive Request Monitoring

```javascript
function monitorRequest(url) {
  const startTime = Date.now();

  const req = request.get(url)
    .on('request', () => {
      console.log('[REQUEST] Starting request to:', url);
    })
    .on('response', res => {
      const duration = Date.now() - startTime;
      console.log('[RESPONSE] Status:', res.status, `(${duration}ms)`);
      console.log('[RESPONSE] Content-Type:', res.type);
      console.log('[RESPONSE] Content-Length:', res.get('content-length'));
    })
    .on('progress', event => {
      if (event.direction === 'download' && event.lengthComputable) {
        console.log('[PROGRESS] Downloaded:', event.percent.toFixed(2) + '%');
      }
    })
    .on('error', err => {
      console.error('[ERROR]', err.message);
    })
    .on('end', () => {
      const duration = Date.now() - startTime;
      console.log('[END] Request completed in', duration, 'ms');
    });

  return req;
}

// Usage
monitorRequest('https://api.example.com/users')
  .then(res => console.log('Final result:', res.body));
```

### Progress Bar Implementation

```javascript
function createProgressBar() {
  return {
    start() {
      console.log('Progress: [' + '·'.repeat(50) + '] 0%');
    },
    update(percent) {
      const filled = Math.round(percent / 2);
      const bar = '█'.repeat(filled) + '·'.repeat(50 - filled);
      process.stdout.write(`\rProgress: [${bar}] ${percent.toFixed(1)}%`);
    },
    complete() {
      console.log('\nComplete!');
    }
  };
}

// Usage
const progressBar = createProgressBar();

request.get('https://api.example.com/large-file')
  .on('request', () => {
    progressBar.start();
  })
  .on('progress', event => {
    if (event.direction === 'download') {
      progressBar.update(event.percent);
    }
  })
  .then(res => {
    progressBar.complete();
    console.log('Downloaded:', res.body.length, 'bytes');
  });
```

### Rate Limiting with Events

```javascript
const requestQueue = [];
let activeRequests = 0;
const MAX_CONCURRENT = 5;

function makeRateLimitedRequest(url) {
  return new Promise((resolve, reject) => {
    const execute = () => {
      activeRequests++;

      request.get(url)
        .on('end', () => {
          activeRequests--;
          if (requestQueue.length > 0) {
            const next = requestQueue.shift();
            next();
          }
        })
        .then(resolve)
        .catch(reject);
    };

    if (activeRequests < MAX_CONCURRENT) {
      execute();
    } else {
      requestQueue.push(execute);
    }
  });
}

// Usage
const urls = Array.from({ length: 20 }, (_, i) =>
  `https://api.example.com/item/${i}`
);

Promise.all(urls.map(makeRateLimitedRequest))
  .then(responses => {
    console.log('All requests complete');
  });
```

## Important Notes

### Event Timing

- Events are emitted in the order: `request` → `progress` (multiple) → `response` → `end`
- The `error` event can be emitted at any time if an error occurs
- The `abort` event is emitted immediately when `.abort()` is called
- Redirect events occur between `request` and `response` (Node.js only)

### Progress Event Behavior

- Progress events are emitted periodically during transfer
- The frequency depends on the platform and data size
- `lengthComputable` is `false` if the total size is unknown (e.g., chunked encoding)
- Upload progress requires the total file size to be known
- Download progress requires the `Content-Length` header

### Browser Limitations

- Redirect events are not available in browsers (browser handles redirects automatically)
- Drain events are Node.js only
- Progress accuracy may vary between browsers
- Some browsers throttle progress events

### Error Handling

- Errors can be handled via both `.on('error')` and `.catch()`
- The error event is emitted before promise rejection
- Network errors don't have a `status` property
- HTTP errors (4xx, 5xx) have both `status` and `response` properties
