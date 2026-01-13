# Timeouts and Retries

SuperAgent provides comprehensive timeout and retry capabilities to handle network reliability concerns, slow responses, and transient failures. The timeout system supports both response-time limits and overall deadline constraints, while the retry mechanism can automatically retry failed requests based on configurable conditions.

## Capabilities

### Setting Timeouts

Configure request timeouts to prevent hanging requests and ensure timely failure handling.

```javascript { .api }
/**
 * Set timeout for request
 *
 * When called with a number, sets the deadline timeout (total time from request
 * start to response completion). When called with an object, can set both
 * response timeout and deadline timeout separately.
 *
 * Response timeout: Time between sending request and receiving the first byte
 * of the response. Includes DNS resolution and connection time.
 *
 * Deadline timeout: Total time from request start to receiving the complete
 * response body. If the deadline is too short, large files may not load on
 * slow connections.
 *
 * A value of 0 or false disables the timeout.
 *
 * @param ms - Timeout in milliseconds (sets deadline)
 * @returns Request instance for chaining
 */
timeout(ms: number): Request;

/**
 * Set separate response and deadline timeouts
 *
 * @param options - Timeout configuration object
 * @param options.response - Response timeout in milliseconds (time to first byte)
 * @param options.deadline - Deadline timeout in milliseconds (total time)
 * @returns Request instance for chaining
 */
timeout(options: { response?: number; deadline?: number }): Request;
```

**Simple timeout example:**

```javascript
// Set 5 second deadline timeout
request
  .get('/api/data')
  .timeout(5000)
  .then(res => console.log(res.body))
  .catch(err => {
    if (err.timeout) {
      console.error('Request timed out');
    }
  });
```

**Granular timeout configuration:**

```javascript
// Set both response and deadline timeouts
request
  .get('/api/large-file')
  .timeout({
    response: 5000,  // Wait 5 seconds for server to start sending
    deadline: 60000  // Allow 60 seconds for entire download
  })
  .then(res => console.log('Downloaded successfully'));
```

**Understanding timeout types:**

```javascript
// Response timeout: Fails if server doesn't respond within 3 seconds
// Useful for detecting unresponsive servers
request
  .get('/api/status')
  .timeout({ response: 3000 })
  .end(callback);

// Deadline timeout: Fails if entire request takes longer than 30 seconds
// Useful for preventing indefinitely long downloads
request
  .get('/api/large-data')
  .timeout({ deadline: 30000 })
  .end(callback);

// Combined: Quick response required, but allow time for data transfer
request
  .post('/api/upload')
  .attach('file', largeFile)
  .timeout({
    response: 5000,   // Server must acknowledge within 5 seconds
    deadline: 120000  // But allow 2 minutes for complete upload
  })
  .end(callback);
```

**Disabling timeouts:**

```javascript
// Disable timeout by setting to 0 or false
request
  .get('/api/long-running-task')
  .timeout(0)  // No timeout
  .end(callback);

// Disable specific timeout
request
  .get('/api/data')
  .timeout({
    response: 5000,
    deadline: 0  // No deadline limit
  })
  .end(callback);
```

### Clearing Timeouts

Clear any previously set timeout configuration.

```javascript { .api }
/**
 * Clear previous timeout settings
 *
 * Removes both response timeout and deadline timeout timers that were
 * previously set via timeout(). This is useful when you want to remove
 * timeout constraints that were set earlier in the request chain or
 * inherited from an agent.
 *
 * @returns Request instance for chaining
 */
clearTimeout(): Request;
```

**Example:**

```javascript
// Clear timeout inherited from agent
const agent = request.agent().timeout(5000);

agent
  .get('/api/long-operation')
  .clearTimeout()  // Remove the 5 second timeout for this request
  .end(callback);
```

```javascript
// Clear timeout set earlier in chain
request
  .get('/api/data')
  .timeout(3000)
  .clearTimeout()  // Actually, don't use a timeout
  .end(callback);
```

### Automatic Retries

Enable automatic retry logic for failed requests with configurable retry count and custom retry conditions.

```javascript { .api }
/**
 * Enable automatic retry on failure
 *
 * Failed requests will be retried up to 'count' times if the failure is
 * retryable. By default, requests are retried for:
 * - Network errors (ECONNRESET, ETIMEDOUT, EADDRINFO, ESOCKETTIMEDOUT)
 * - Timeout errors (ECONNABORTED with timeout flag)
 * - 5xx server errors (except 501 Not Implemented)
 * - Cross-domain errors
 *
 * The retry callback allows custom retry logic. Return true to force retry,
 * false to prevent retry, or undefined to use default retry logic.
 *
 * @param count - Number of retry attempts (default 1 if omitted or true)
 * @param callback - Optional function to determine if retry should occur
 * @returns Request instance for chaining
 */
retry(count?: number, callback?: (err: Error, res: Response) => boolean | undefined): Request;
```

**Basic retry:**

```javascript
// Retry once on failure
request
  .get('/api/unreliable-endpoint')
  .retry()  // Defaults to 1 retry
  .end(callback);

// Retry up to 3 times
request
  .get('/api/data')
  .retry(3)
  .end(callback);
```

**Custom retry logic:**

```javascript
// Retry with custom condition
request
  .get('/api/data')
  .retry(2, (err, res) => {
    // Only retry on 503 Service Unavailable
    if (res && res.status === 503) {
      return true;
    }
    // Don't retry on 4xx client errors
    if (res && res.status >= 400 && res.status < 500) {
      return false;
    }
    // Use default retry logic for other cases
    return undefined;
  })
  .end(callback);
```

**Retry with exponential backoff (using plugin):**

```javascript
// Custom retry plugin with backoff
function retryWithBackoff(req) {
  let retries = 0;
  const maxRetries = 3;

  req.retry(maxRetries, (err, res) => {
    if (err || res.status >= 500) {
      retries++;
      // Wait longer between each retry
      const delay = Math.pow(2, retries) * 1000;
      return new Promise(resolve => {
        setTimeout(() => resolve(true), delay);
      });
    }
  });
}

request
  .get('/api/data')
  .use(retryWithBackoff)
  .end(callback);
```

**Conditional retry based on error type:**

```javascript
request
  .post('/api/process')
  .send(data)
  .retry(3, (err, res) => {
    // Retry on timeout
    if (err && err.timeout) {
      console.log('Timeout, retrying...');
      return true;
    }

    // Retry on connection errors
    if (err && err.code === 'ECONNRESET') {
      console.log('Connection reset, retrying...');
      return true;
    }

    // Don't retry on 4xx errors (client errors)
    if (res && res.status >= 400 && res.status < 500) {
      return false;
    }

    // Use default behavior for other errors
  })
  .end(callback);
```

### Aborting Requests

Manually abort an in-progress request and clear any associated timeouts.

```javascript { .api }
/**
 * Abort the request
 *
 * Cancels an in-flight request, clears any timeout timers, and emits an
 * 'abort' event. The request callback will not be invoked after abort.
 * Safe to call multiple times (subsequent calls are no-ops).
 *
 * Works in both Node.js (aborts the underlying http.ClientRequest) and
 * browser (aborts the XMLHttpRequest).
 *
 * @returns Request instance for chaining
 */
abort(): Request;
```

**Manual abort:**

```javascript
const req = request.get('/api/large-file');

// Abort after 2 seconds
setTimeout(() => {
  req.abort();
}, 2000);

req.end((err, res) => {
  if (err) {
    console.log('Request aborted or failed');
  }
});
```

**Abort on user action:**

```javascript
let currentRequest = null;

function searchUsers(query) {
  // Abort previous request if still running
  if (currentRequest) {
    currentRequest.abort();
  }

  currentRequest = request
    .get('/api/users/search')
    .query({ q: query })
    .end((err, res) => {
      if (err) return;
      displayResults(res.body);
      currentRequest = null;
    });
}

// User types quickly - previous requests get aborted
searchUsers('joh');
searchUsers('john');
searchUsers('john doe');
```

**Abort with cleanup:**

```javascript
const req = request.get('/api/data');

// Listen for abort event
req.on('abort', () => {
  console.log('Request was aborted');
  // Perform cleanup
  cleanup();
});

// Abort the request
req.abort();

req.end((err, res) => {
  // This callback won't be called after abort
});
```

**Race condition handling:**

```javascript
// Timeout vs manual abort
const req = request
  .get('/api/slow-endpoint')
  .timeout(5000)
  .end((err, res) => {
    if (err && err.timeout) {
      console.log('Timed out');
    } else if (err && err.code === 'ABORTED') {
      console.log('Manually aborted');
    }
  });

// Manually abort if user navigates away
window.addEventListener('beforeunload', () => {
  req.abort();
});
```

## Error Handling

### Timeout Errors

When a request times out, an error is passed to the callback or promise rejection with specific properties:

```javascript
try {
  const res = await request
    .get('/api/slow-endpoint')
    .timeout(3000);
} catch (err) {
  console.error(err.message);  // "Timeout of 3000ms exceeded"
  console.error(err.timeout);  // 3000
  console.error(err.code);     // "ECONNABORTED"
  console.error(err.errno);    // "ETIME" or "ETIMEDOUT"
}
```

**Error properties:**

- `err.timeout` - The timeout value in milliseconds
- `err.code` - Error code: `'ECONNABORTED'`
- `err.errno` - More specific error code: `'ETIME'` for deadline timeout, `'ETIMEDOUT'` for response timeout
- `err.message` - Human-readable error message

**Distinguishing timeout types:**

```javascript
request
  .get('/api/data')
  .timeout({
    response: 5000,
    deadline: 30000
  })
  .end((err, res) => {
    if (err) {
      if (err.errno === 'ETIMEDOUT') {
        console.error('Server took too long to respond (>5s)');
      } else if (err.errno === 'ETIME') {
        console.error('Download took too long (>30s)');
      }
    }
  });
```

### Retry Tracking

Error objects contain retry information when retry is enabled:

```javascript
request
  .get('/api/data')
  .retry(3)
  .end((err, res) => {
    if (err) {
      console.error(`Failed after ${err.retries} retries`);
      console.error('Original error:', err.message);
    }
  });
```

**Retry information:**

- `err.retries` - Number of retry attempts that were made
- `err.response` - Response object from the last failed attempt (if available)

### Combining Timeout and Retry

```javascript
// Retry on timeout
request
  .get('/api/flaky-endpoint')
  .timeout(5000)        // 5 second timeout
  .retry(3, (err, res) => {
    // Retry if timeout occurred
    if (err && err.timeout) {
      console.log(`Timeout on attempt ${err.retries + 1}, retrying...`);
      return true;
    }
  })
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    console.error('Failed after all retries:', err.message);
  });
```

**Increasing timeout on retry:**

```javascript
let attempt = 0;
const baseTimeout = 5000;

function makeRequest() {
  attempt++;
  const timeout = baseTimeout * attempt;  // Increase timeout each retry

  return request
    .get('/api/data')
    .timeout(timeout)
    .retry(3, (err, res) => {
      if (err && err.timeout) {
        console.log(`Attempt ${attempt} timed out after ${timeout}ms`);
        return true;
      }
    });
}

makeRequest()
  .then(res => console.log('Success'))
  .catch(err => console.error('All attempts failed'));
```

## Agent-Level Configuration

Agents can set default timeout and retry settings for all requests:

```javascript
// Create agent with default timeout and retry
const agent = request.agent()
  .timeout(10000)   // All requests timeout after 10 seconds
  .retry(2);        // All requests retry up to 2 times

// These settings apply to all requests from this agent
agent
  .get('/api/endpoint1')
  .end(callback);

agent
  .post('/api/endpoint2')
  .send(data)
  .end(callback);

// Override agent defaults for specific request
agent
  .get('/api/long-operation')
  .timeout(60000)   // Override with longer timeout
  .end(callback);
```

**Complex agent configuration:**

```javascript
const agent = request.agent()
  .timeout({
    response: 5000,
    deadline: 30000
  })
  .retry(3, (err, res) => {
    // Custom retry logic for all agent requests
    if (res && res.status === 429) {
      // Retry on rate limit
      return true;
    }
    if (res && res.status >= 400 && res.status < 500) {
      // Don't retry client errors
      return false;
    }
  });

// All requests inherit these settings
agent.get('/api/users').end(callback);
agent.post('/api/data').send(data).end(callback);
```

## Best Practices

**Use appropriate timeout values:**

```javascript
// Quick API calls
request
  .get('/api/status')
  .timeout(3000);  // 3 seconds is enough

// Large file downloads
request
  .get('/api/download/large-file')
  .timeout({
    response: 5000,   // Server should respond quickly
    deadline: 300000  // But allow 5 minutes for download
  });

// Real-time updates
request
  .get('/api/stream')
  .timeout({
    response: 2000,
    deadline: 0  // No deadline for streaming
  });
```

**Implement smart retry logic:**

```javascript
// Good: Retry transient errors, not permanent failures
request
  .post('/api/create-user')
  .send(userData)
  .retry(2, (err, res) => {
    // Don't retry validation errors
    if (res && res.status === 400) {
      return false;
    }
    // Don't retry conflicts
    if (res && res.status === 409) {
      return false;
    }
    // Retry server errors and timeouts
    if ((err && err.timeout) || (res && res.status >= 500)) {
      return true;
    }
  })
  .end(callback);
```

**Handle cleanup on abort:**

```javascript
class RequestManager {
  constructor() {
    this.activeRequests = new Set();
  }

  fetch(url) {
    const req = request.get(url);
    this.activeRequests.add(req);

    req.on('abort', () => {
      this.activeRequests.delete(req);
    });

    return req.end((err, res) => {
      this.activeRequests.delete(req);
      // Handle response
    });
  }

  abortAll() {
    this.activeRequests.forEach(req => req.abort());
    this.activeRequests.clear();
  }
}
```

**Provide user feedback during retries:**

```javascript
let retryCount = 0;

request
  .get('/api/data')
  .retry(3, (err, res) => {
    retryCount++;
    if (err || (res && res.status >= 500)) {
      updateUI(`Retry attempt ${retryCount}...`);
      return true;
    }
  })
  .then(res => {
    updateUI('Success!');
  })
  .catch(err => {
    updateUI(`Failed after ${retryCount} retries`);
  });
```

## Types

```javascript { .api }
interface Request {
  /**
   * Set timeout in milliseconds or configure response and deadline timeouts
   */
  timeout(ms: number): Request;
  timeout(options: { response?: number; deadline?: number }): Request;

  /**
   * Enable automatic retry on failure
   */
  retry(count?: number, callback?: (err: Error, res: Response) => boolean | undefined): Request;

  /**
   * Clear any set timeouts
   */
  clearTimeout(): Request;

  /**
   * Abort the current request
   */
  abort(): Request;
}

interface TimeoutError extends Error {
  /** Timeout value in milliseconds */
  timeout: number;

  /** Error code: 'ECONNABORTED' */
  code: 'ECONNABORTED';

  /** Specific timeout type: 'ETIME' (deadline) or 'ETIMEDOUT' (response) */
  errno: 'ETIME' | 'ETIMEDOUT';

  /** Error message */
  message: string;
}

interface RetryError extends Error {
  /** Number of retry attempts made */
  retries: number;

  /** Response from last failed attempt (if available) */
  response?: Response;
}
```

## Internal Implementation Details

**Timeout implementation:**

The timeout system uses two separate timers internally:
- `_timer` - Tracks the deadline timeout (overall request duration)
- `_responseTimeoutTimer` - Tracks the response timeout (time to first byte)

Both timers are cleared when the request completes or is aborted.

**Retry mechanism:**

The `_shouldRetry()` method determines if a request should be retried based on:
1. Remaining retry attempts (`_maxRetries` and `_retries`)
2. Custom retry callback result (`_retryCallback`)
3. Default retry conditions:
   - Network error codes: ECONNRESET, ETIMEDOUT, EADDRINFO, ESOCKETTIMEDOUT
   - Timeout errors (ECONNABORTED with timeout flag)
   - 5xx server errors (except 501)
   - Cross-domain errors

The `_retry()` method clears timeouts, resets the request object, and calls `_end()` to retry.

**Error codes for retry:**

```javascript
const ERROR_CODES = [
  'ECONNRESET',      // Connection reset by peer
  'ETIMEDOUT',       // Operation timed out
  'EADDRINFO',       // DNS lookup failed
  'ESOCKETTIMEDOUT'  // Socket timeout
];
```
