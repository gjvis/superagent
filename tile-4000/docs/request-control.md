# Request Control

Control request behavior including timeouts, retries, redirects, and aborting.

## Capabilities

### Timeouts

Set timeout limits for requests with support for both deadline and response timeouts.

```javascript { .api }
/**
 * Set overall request deadline timeout
 * @param ms - Timeout in milliseconds (0 or false to disable)
 * @returns Request instance for chaining
 */
Request.prototype.timeout(ms: number | false): Request;

/**
 * Set response and deadline timeouts separately
 * @param options - Timeout options object
 * @returns Request instance for chaining
 */
Request.prototype.timeout(options: { response?: number, deadline?: number }): Request;
```

**Timeout Types:**
- **response**: Time to receive the first byte (includes DNS and connection time)
- **deadline**: Total time from request start to full response body received

**Usage Examples:**

```javascript
// Overall deadline timeout (60 seconds)
request.get('https://api.example.com/users')
  .timeout(60000)
  .catch(err => {
    if (err.timeout) {
      console.error('Request timed out');
    }
  });

// Separate response and deadline timeouts
request.get('https://api.example.com/large-file')
  .timeout({
    response: 5000,  // Wait max 5s for server to start sending
    deadline: 60000  // Allow up to 60s for full download
  })
  .catch(err => {
    if (err.timeout) {
      console.error('Timeout:', err.message);
    }
  });

// Disable timeout
request.get('https://api.example.com/long-running')
  .timeout(0); // No timeout

// Different timeouts for different requests
request.post('https://api.example.com/quick')
  .timeout(5000); // 5 second timeout

request.post('https://api.example.com/slow-process')
  .timeout(300000); // 5 minute timeout
```

### Retries

Automatically retry failed requests with customizable retry logic.

```javascript { .api }
/**
 * Enable automatic retries on failure
 * @param count - Number of retry attempts (default: 1)
 * @param callback - Optional custom retry logic function
 * @returns Request instance for chaining
 */
Request.prototype.retry(count?: number, callback?: (err: Error, res: Response) => boolean): Request;
```

**Default Retry Conditions:**
- Network errors: ECONNRESET, ETIMEDOUT, EADDRINFO, ESOCKETTIMEDOUT
- HTTP 5xx errors (except 501)
- Timeout errors
- CORS errors (browser)

**Usage Examples:**

```javascript
// Retry up to 3 times on failure
request.get('https://api.example.com/users')
  .retry(3)
  .then(res => console.log(res.body))
  .catch(err => console.error('Failed after 3 retries'));

// Default retry (1 attempt)
request.get('https://api.example.com/users')
  .retry()
  .then(res => console.log(res.body));

// Disable retries
request.get('https://api.example.com/users')
  .retry(0);

// Custom retry logic
request.get('https://api.example.com/users')
  .retry(3, (err, res) => {
    // Return true to retry, false to stop
    if (err) {
      // Retry on specific error codes
      return err.code === 'ECONNRESET';
    }
    if (res) {
      // Retry on specific status codes
      return res.status === 503; // Service Unavailable
    }
    return false;
  })
  .then(res => console.log(res.body));

// Retry with timeout
request.get('https://api.example.com/users')
  .timeout(5000)
  .retry(3)
  .then(res => console.log(res.body));
```

### Aborting Requests

Abort in-flight requests.

```javascript { .api }
/**
 * Abort the request
 * @returns Request instance
 */
Request.prototype.abort(): Request;
```

**Usage Examples:**

```javascript
// Abort after timeout
const req = request.get('https://api.example.com/large-data')
  .then(res => console.log(res.body))
  .catch(err => {
    if (err.code === 'ABORTED') {
      console.log('Request was aborted');
    }
  });

setTimeout(() => {
  req.abort();
}, 5000); // Abort after 5 seconds

// Abort on user action
const req = request.get('https://api.example.com/users');

document.querySelector('#cancel-button').addEventListener('click', () => {
  req.abort();
  console.log('Request cancelled');
});

// Abort promise
const controller = {
  req: null,
  cancel() {
    if (this.req) {
      this.req.abort();
    }
  }
};

controller.req = request.get('https://api.example.com/users')
  .then(res => console.log(res.body));

// Later: controller.cancel();
```

### Redirects (Node.js)

Control automatic redirect following behavior.

```javascript { .api }
/**
 * Set maximum number of redirects to follow
 * @param count - Maximum redirects (0 to disable, default: 5)
 * @returns Request instance for chaining
 */
Request.prototype.redirects(count: number): Request;
```

**Usage Examples:**

```javascript
// Default: follow up to 5 redirects
request.get('https://api.example.com/redirect');

// Follow up to 10 redirects
request.get('https://api.example.com/redirect')
  .redirects(10);

// Disable redirect following
request.get('https://api.example.com/redirect')
  .redirects(0)
  .then(res => {
    // Will receive 301/302 response instead of following
    console.log('Status:', res.status);
    console.log('Location:', res.header.location);
  });

// Track redirect chain
request.get('https://api.example.com/redirect')
  .then(res => {
    console.log('Final URL:', res.req.url);
    console.log('Redirects:', res.redirects);
    // res.redirects is an array of intermediate URLs
  });
```

### Maximum Response Size (Node.js)

Limit the maximum size of response bodies to prevent memory issues.

```javascript { .api }
/**
 * Set maximum response body size in bytes
 * @param bytes - Maximum size (default: 200MB)
 * @returns Request instance for chaining
 */
Request.prototype.maxResponseSize(bytes: number): Request;
```

**Usage Examples:**

```javascript
// Limit response to 10MB
request.get('https://api.example.com/large-data')
  .maxResponseSize(10 * 1024 * 1024) // 10MB
  .catch(err => {
    if (err.message.includes('maximum')) {
      console.error('Response too large');
    }
  });

// Limit to 1MB for JSON API
request.get('https://api.example.com/users')
  .maxResponseSize(1024 * 1024) // 1MB
  .then(res => console.log(res.body));

// Default is 200MB
request.get('https://api.example.com/users'); // 200MB limit
```

### Clear Timeouts

Manually clear active timeouts.

```javascript { .api }
/**
 * Clear all active timeouts on the request
 * @returns Request instance for chaining
 */
Request.prototype.clearTimeout(): Request;
```

**Usage Examples:**

```javascript
// Start request with timeout
const req = request.get('https://api.example.com/users')
  .timeout(10000);

// Clear timeout if needed
if (someCondition) {
  req.clearTimeout(); // Remove timeout constraint
}

req.then(res => console.log(res.body));
```

### Request Serialization

Convert request to a plain object for logging or debugging.

```javascript { .api }
/**
 * Serialize request to plain object
 * @returns Object with method, URL, headers, and other request details
 */
Request.prototype.toJSON(): object;
```

**Usage Examples:**

```javascript
const req = request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer token123')
  .query({ page: 1, limit: 10 });

// Serialize for logging
const serialized = req.toJSON();
console.log(JSON.stringify(serialized, null, 2));
// {
//   method: 'GET',
//   url: 'https://api.example.com/users?page=1&limit=10',
//   header: { authorization: 'Bearer token123', ... },
//   ...
// }

// Log request before sending
function logRequest(req) {
  const json = req.toJSON();
  console.log(`[REQUEST] ${json.method} ${json.url}`);
  console.log('[HEADERS]', json.header);
  return req;
}

// Usage
logRequest(request.get('https://api.example.com/users'))
  .then(res => console.log(res.body));
```

## Control Flow Patterns

### Timeout with Retry

```javascript
// Retry on timeout
async function fetchWithRetry(url, retries = 3) {
  try {
    const res = await request.get(url)
      .timeout(5000)
      .retry(retries);
    return res.body;
  } catch (err) {
    console.error('Failed after retries:', err.message);
    throw err;
  }
}
```

### Cancellable Async Operation

```javascript
// Wrapper for cancellable requests
function cancellableRequest(url) {
  let aborted = false;
  const req = request.get(url);

  return {
    promise: req.then(res => {
      if (aborted) throw new Error('Aborted');
      return res;
    }),
    cancel: () => {
      aborted = true;
      req.abort();
    }
  };
}

// Usage
const { promise, cancel } = cancellableRequest('https://api.example.com/users');

promise
  .then(res => console.log(res.body))
  .catch(err => console.error(err));

// Cancel if needed
// cancel();
```

### Progressive Timeout

```javascript
// Increase timeout on retries
let attempt = 0;
request.get('https://api.example.com/users')
  .timeout(5000 * Math.pow(2, attempt)) // 5s, 10s, 20s
  .retry(3, (err) => {
    if (err && err.timeout) {
      attempt++;
      return true; // Retry with longer timeout
    }
    return false;
  })
  .then(res => console.log(res.body));
```

### Conditional Abort

```javascript
// Abort if condition changes
let userCancelled = false;
const req = request.get('https://api.example.com/large-file')
  .then(res => {
    if (userCancelled) {
      console.log('Ignoring response - user cancelled');
      return;
    }
    console.log('Processing:', res.body);
  });

// Check condition periodically
const interval = setInterval(() => {
  if (userCancelled) {
    req.abort();
    clearInterval(interval);
  }
}, 100);
```

### Redirect Handling

```javascript
// Manually handle redirects
request.get('https://api.example.com/redirect')
  .redirects(0)
  .then(res => {
    if (res.status >= 300 && res.status < 400) {
      const location = res.header.location;
      console.log('Redirecting to:', location);
      return request.get(location);
    }
    return res;
  })
  .then(res => console.log('Final response:', res.body));
```

## Important Notes

### Timeout Behavior

- Timeouts apply to the entire request lifecycle (DNS, connection, transfer)
- A timeout causes the request to abort and reject/error
- Default: No timeout (waits indefinitely)
- Setting timeout to `0` or `false` disables timeout

### Retry Behavior

- Retries are only attempted for retriable errors (network, 5xx)
- Each retry is a completely new request
- Original request configuration is preserved
- Retries do NOT reset on successful connection - only on error

### Abort Behavior

- Aborting emits an 'abort' event
- Pending promise is rejected with an error
- Callback receives an error
- Abort is immediate - no cleanup delay

### Browser Limitations

- `redirects()` has no effect in browser (browser handles redirects)
- `maxResponseSize()` has no effect in browser
- Timeout precision may vary based on browser implementation
