# Timeout and Retry

Configure request timeouts and automatic retry logic for failed requests.

## Capabilities

### Request Timeout

Set timeout limits for requests to prevent hanging connections.

```javascript { .api }
/**
 * Set request timeout
 * @param ms - Timeout in milliseconds (applies to both response and deadline)
 * @returns Request instance for chaining
 */
timeout(ms: number): Request;

/**
 * Set separate response and deadline timeouts
 * @param options - Timeout configuration
 * @param options.response - Maximum time to wait for first byte of response
 * @param options.deadline - Maximum total time for entire request (including redirects and retries)
 * @returns Request instance for chaining
 */
timeout(options: {response?: number, deadline?: number}): Request;

/**
 * Clear configured timeouts
 * @returns Request instance for chaining
 */
clearTimeout(): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Simple timeout (applies to both response and deadline)
request
  .get('/api/users')
  .timeout(5000) // 5 seconds
  .end((err, res) => {
    if (err && err.timeout) {
      console.log('Request timed out');
    }
  });

// Separate response and deadline timeouts
request
  .get('/api/large-file')
  .timeout({
    response: 5000,  // Wait max 5s for server to start responding
    deadline: 60000  // Allow max 60s total for entire request
  })
  .end((err, res) => {
    if (err && err.timeout) {
      console.log('Timeout occurred');
      console.log('Timeout type:', err.code); // 'ECONNABORTED'
    }
  });

// Clear timeout
const req = request
  .get('/api/users')
  .timeout(5000)
  .clearTimeout(); // removes timeout
```

**Timeout Details:**

- **Response timeout**: Time to wait for the first byte of the response
- **Deadline timeout**: Maximum time for the entire request including all retries and redirects
- If only a number is provided, it applies to both response and deadline
- Timeout errors will have `err.timeout = true` and `err.code = 'ECONNABORTED'`

### Automatic Retry

Configure automatic retry for failed requests.

```javascript { .api }
/**
 * Set automatic retry behavior
 * @param count - Number of retries (default: 1 if callback provided)
 * @param callback - Optional function to decide whether to retry
 * @returns Request instance for chaining
 */
retry(count?: number, callback?: (err: Error, res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Retry up to 2 times on failure
request
  .get('/api/users')
  .retry(2)
  .end((err, res) => {
    if (err) {
      console.log('Failed after 2 retries');
    }
  });

// Retry with custom logic
request
  .get('/api/users')
  .retry(3, (err, res) => {
    // Only retry on network errors, not 4xx/5xx
    if (err && !err.response) {
      return true; // retry
    }
    return false; // don't retry
  })
  .end((err, res) => {
    console.log(res.body);
  });

// Conditional retry based on error type
request
  .get('/api/users')
  .retry(2, (err, res) => {
    // Retry on 503 Service Unavailable
    if (err && err.status === 503) {
      return true;
    }
    // Retry on network errors
    if (err && !err.status) {
      return true;
    }
    return false;
  });
```

**Default Retry Behavior:**

By default, SuperAgent retries on the following error codes (Node.js):
- `ECONNRESET` - Connection reset
- `ETIMEDOUT` - Connection timeout
- `EADDRINFO` - DNS lookup failed
- `ESOCKETTIMEDOUT` - Socket timeout

HTTP status codes are NOT retried by default (including 5xx errors). Use a custom retry callback to retry on specific status codes.

### Combined Timeout and Retry

Use timeout and retry together for robust error handling.

**Usage Examples:**

```javascript
// Timeout with retry
request
  .get('/api/unreliable-service')
  .timeout({
    response: 5000,
    deadline: 20000
  })
  .retry(3)
  .end((err, res) => {
    if (err) {
      if (err.timeout) {
        console.log('Request timed out after 3 retries');
      } else {
        console.log('Request failed after 3 retries');
      }
    } else {
      console.log('Success:', res.body);
    }
  });

// Custom retry with timeout
request
  .get('/api/data')
  .timeout(10000)
  .retry(2, (err, res) => {
    // Don't retry on timeout (already took too long)
    if (err && err.timeout) {
      return false;
    }
    // Retry on network errors
    if (err && !err.response) {
      return true;
    }
    return false;
  })
  .end((err, res) => {
    console.log(res ? res.body : err);
  });

// Exponential backoff with retry
let retryCount = 0;
request
  .get('/api/data')
  .timeout(5000)
  .retry(3, (err, res) => {
    if (err && !err.timeout) {
      retryCount++;
      const delay = Math.pow(2, retryCount) * 1000; // exponential backoff
      console.log(`Retry ${retryCount} after ${delay}ms`);
      // Note: SuperAgent doesn't have built-in delay between retries
      // This is just for illustration
      return true;
    }
    return false;
  });
```

### Abort Request

Cancel a request that's in progress.

```javascript { .api }
/**
 * Abort the request
 * @returns Request instance
 */
abort(): Request;
```

**Usage Examples:**

```javascript
// Abort after timeout
const req = request.get('/api/large-file');

setTimeout(() => {
  req.abort();
  console.log('Request aborted');
}, 5000);

req.end((err, res) => {
  if (err && err.code === 'ABORTED') {
    console.log('Request was aborted');
  }
});

// Abort on user action
const req = request
  .get('/api/data')
  .end((err, res) => {
    if (err && err.code === 'ABORTED') {
      console.log('User cancelled');
    } else {
      console.log(res.body);
    }
  });

// User clicks cancel button
document.getElementById('cancel').addEventListener('click', () => {
  req.abort();
});
```

## Error Codes

Common error codes you may encounter:

- `ECONNABORTED` - Request timeout
- `ABORTED` - Request manually aborted
- `ECONNRESET` - Connection reset by server
- `ETIMEDOUT` - Connection timeout
- `EADDRINFO` - DNS lookup failed
- `ESOCKETTIMEDOUT` - Socket timeout
