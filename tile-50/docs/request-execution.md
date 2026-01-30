# Request Execution

Methods for executing requests, handling responses with promises or callbacks, aborting requests, configuring timeouts and retries.

## Capabilities

### Execute Request

Send the HTTP request and handle the response.

```javascript { .api }
/**
 * Execute the request and invoke callback
 * Required unless using promises (.then/.catch)
 * @param callback - Optional callback(err, res)
 * @returns Request for chaining
 */
request.get(url).end(callback?: (err: Error | null, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Callback style
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Request failed:', err);
      return;
    }
    console.log('Users:', res.body);
  });

// No callback (fires and forgets)
request
  .post('/api/log')
  .send({ message: 'Event occurred' })
  .end();

// Error handling in callback
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Status:', err.status);       // HTTP status code
      console.error('Message:', err.message);     // Error message
      console.error('Response:', err.response);   // Response object
      return;
    }
    // Success handling
  });
```

### Promise Support

Use promises for async/await or promise chains.

```javascript { .api }
/**
 * Promise support for request execution
 * Automatically calls .end() when promise is created
 * @param onFulfilled - Success handler receiving Response
 * @param onRejected - Optional error handler receiving Error
 * @returns Promise resolving to Response
 */
request.get(url).then(
  onFulfilled: (res: Response) => any,
  onRejected?: (err: Error) => any
): Promise<any>;

/**
 * Promise error handling
 * @param onRejected - Error handler receiving Error
 * @returns Promise
 */
request.get(url).catch(onRejected: (err: Error) => any): Promise<any>;
```

**Usage Examples:**

```javascript
// Promise chain
request
  .get('/api/users')
  .then(res => {
    console.log('Users:', res.body);
    return res.body;
  })
  .catch(err => {
    console.error('Error:', err.message);
  });

// Async/await
async function getUsers() {
  try {
    const res = await request.get('/api/users');
    return res.body;
  } catch (err) {
    console.error('Error:', err.message);
    throw err;
  }
}

// Multiple requests
async function getUserProfile(userId) {
  const user = await request.get(`/api/users/${userId}`);
  const posts = await request.get(`/api/posts?user=${userId}`);
  return { user: user.body, posts: posts.body };
}

// Promise.all for parallel requests
const [users, posts, comments] = await Promise.all([
  request.get('/api/users'),
  request.get('/api/posts'),
  request.get('/api/comments')
]);

// Warning: Do not mix .end() and .then()
// This is incorrect:
request
  .get('/api/users')
  .end()
  .then(res => console.log(res)); // ERROR: sends twice
```

### Abort Request

Cancel an in-progress request.

```javascript { .api }
/**
 * Abort the request
 * Clears timeouts and emits 'abort' event
 * Safe to call multiple times
 * @returns Request for chaining
 */
request.get(url).abort(): Request;
```

**Usage Examples:**

```javascript
// Abort after delay
const req = request.get('/api/large-data');

setTimeout(() => {
  req.abort();
  console.log('Request aborted');
}, 1000);

req.then(res => {
  console.log('Completed:', res.body);
}).catch(err => {
  if (err.message === 'Aborted') {
    console.log('Request was cancelled');
  }
});

// Abort on user action
const req = request.get('/api/search').query({ q: 'term' });

document.getElementById('cancelBtn').addEventListener('click', () => {
  req.abort();
});

req.then(res => {
  displayResults(res.body);
});

// Listen to abort event
const req = request.get('/api/data');
req.on('abort', () => {
  console.log('Request aborted');
});
req.abort();
```

### Timeout Configuration

Set timeout limits for requests.

```javascript { .api }
/**
 * Set timeout configuration
 * Value of 0 or false means no timeout
 * @param options - Milliseconds (deadline) or {response, deadline} object
 *   - response: Time to first byte (includes DNS and connection time)
 *   - deadline: Total time from start to completion
 * @returns Request for chaining
 */
request.get(url).timeout(options: number | {response?: number, deadline?: number}): Request;

/**
 * Clear timeout timers
 * @returns Request for chaining
 */
request.get(url).clearTimeout(): Request;
```

**Usage Examples:**

```javascript
// Simple deadline timeout (5 seconds total)
request
  .get('/api/users')
  .timeout(5000)
  .catch(err => {
    if (err.timeout) {
      console.error('Request timed out');
    }
  });

// Response timeout (time to first byte)
request
  .get('/api/users')
  .timeout({ response: 5000 })
  .catch(err => {
    if (err.timeout) {
      console.error('No response within 5 seconds');
    }
  });

// Both timeouts
request
  .get('/api/large-download')
  .timeout({
    response: 5000,  // Must start responding within 5 seconds
    deadline: 60000  // Must complete within 60 seconds
  })
  .catch(err => {
    if (err.timeout) {
      console.error('Timeout:', err.message);
    }
  });

// No timeout
request
  .get('/api/long-running')
  .timeout(0);

// Clear timeout manually
const req = request
  .get('/api/data')
  .timeout(5000);

// Later, remove timeout
req.clearTimeout();

// Timeout error properties
request
  .get('/api/data')
  .timeout(1000)
  .catch(err => {
    if (err.timeout) {
      console.error('Timeout:', err.timeout);     // Timeout value
      console.error('Code:', err.code);           // 'ECONNABORTED'
      console.error('Errno:', err.errno);         // 'ETIME' or 'ETIMEDOUT'
    }
  });
```

### Retry Configuration

Automatically retry failed requests.

```javascript { .api }
/**
 * Set retry configuration
 * Retries on 5xx errors (except 501), timeouts, and network errors
 * @param count - Number of retry attempts (0 = no retry)
 * @param callback - Optional custom retry logic (err, res) => boolean
 *   Return true to retry, false to skip, undefined for default behavior
 * @returns Request for chaining
 */
request.get(url).retry(count: number, callback?: (err: Error, res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Retry up to 3 times
request
  .get('/api/unstable')
  .retry(3)
  .then(res => console.log('Success:', res.body))
  .catch(err => console.error('Failed after 3 retries'));

// Default retry behavior:
// - Retries on 5xx status (except 501 Not Implemented)
// - Retries on timeout errors
// - Retries on connection errors (ECONNRESET, ETIMEDOUT, etc.)

// Custom retry logic
request
  .get('/api/rate-limited')
  .retry(5, (err, res) => {
    // Retry on 429 Too Many Requests
    if (res && res.status === 429) {
      console.log('Rate limited, retrying...');
      return true;
    }
    // Retry on 503 Service Unavailable
    if (res && res.status === 503) {
      console.log('Service unavailable, retrying...');
      return true;
    }
    // Use default retry logic for other cases
    return undefined;
  })
  .then(res => console.log('Success:', res.body));

// Don't retry on client errors
request
  .get('/api/data')
  .retry(3, (err, res) => {
    // Don't retry 4xx errors
    if (res && res.status >= 400 && res.status < 500) {
      return false;
    }
  });

// Retry with exponential backoff (using plugin)
function retryWithBackoff(req) {
  let retryCount = 0;
  req.retry(3, (err, res) => {
    if (res && res.status >= 500) {
      retryCount++;
      const delay = Math.pow(2, retryCount) * 1000;
      console.log(`Retrying in ${delay}ms...`);
      setTimeout(() => {}, delay);
      return true;
    }
  });
}

request
  .get('/api/data')
  .use(retryWithBackoff)
  .then(res => console.log(res.body));

// No retry
request
  .get('/api/data')
  .retry(0);

// Error codes that trigger automatic retry:
// - ECONNRESET: Connection reset
// - ETIMEDOUT: Request timeout
// - EADDRINFO: DNS resolution error
// - ESOCKETTIMEDOUT: Socket timeout
// - ECONNABORTED: Connection aborted (timeout)
// - crossDomain: Cross-domain error
```

### Retry State

Internal retry state is tracked on the request object:

```javascript
const req = request
  .get('/api/data')
  .retry(3);

// Internal properties (read-only, for debugging):
// req._retries - Current retry count
// req._maxRetries - Max retry count
// req._retryCallback - Custom retry callback
```

## Error Handling

All execution methods provide detailed error information:

```javascript
request
  .get('/api/users')
  .then(res => {
    // Success
  })
  .catch(err => {
    // Error properties
    console.error('Message:', err.message);      // Error description
    console.error('Status:', err.status);        // HTTP status code (if available)
    console.error('Response:', err.response);    // Response object (if available)
    console.error('Timeout:', err.timeout);      // Timeout value (if timeout error)
    console.error('Code:', err.code);            // Error code (e.g., 'ECONNABORTED')
    console.error('Errno:', err.errno);          // System error number

    // Check error type
    if (err.timeout) {
      console.error('Request timed out');
    } else if (err.status >= 500) {
      console.error('Server error');
    } else if (err.status >= 400) {
      console.error('Client error');
    } else {
      console.error('Network or parse error');
    }
  });
```

## Execution Patterns

### Sequential Requests

```javascript
async function sequentialRequests() {
  const user = await request.get('/api/user');
  const posts = await request.get(`/api/posts?user=${user.body.id}`);
  const comments = await request.get(`/api/comments?post=${posts.body[0].id}`);
  return { user: user.body, posts: posts.body, comments: comments.body };
}
```

### Parallel Requests

```javascript
async function parallelRequests() {
  const [users, posts, comments] = await Promise.all([
    request.get('/api/users'),
    request.get('/api/posts'),
    request.get('/api/comments')
  ]);
  return {
    users: users.body,
    posts: posts.body,
    comments: comments.body
  };
}
```

### Conditional Requests

```javascript
async function conditionalRequests(userId) {
  const user = await request.get(`/api/users/${userId}`);

  if (user.body.role === 'admin') {
    const adminData = await request.get('/api/admin/data');
    return { ...user.body, admin: adminData.body };
  }

  return user.body;
}
```

### Request with Fallback

```javascript
async function requestWithFallback() {
  try {
    return await request.get('/api/primary');
  } catch (err) {
    console.warn('Primary failed, trying fallback');
    return await request.get('/api/fallback');
  }
}
```

### Polling Pattern

```javascript
async function pollUntilComplete(jobId) {
  while (true) {
    const res = await request.get(`/api/jobs/${jobId}`);

    if (res.body.status === 'completed') {
      return res.body;
    }

    if (res.body.status === 'failed') {
      throw new Error('Job failed');
    }

    // Wait before next poll
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```

### Request Cancellation Pattern

```javascript
class CancellableRequest {
  constructor(url) {
    this.req = request.get(url);
  }

  execute() {
    return this.req;
  }

  cancel() {
    this.req.abort();
  }
}

const req = new CancellableRequest('/api/data');

// Start request
const promise = req.execute();

// Cancel after 5 seconds
setTimeout(() => {
  req.cancel();
}, 5000);

promise
  .then(res => console.log('Completed:', res.body))
  .catch(err => console.log('Cancelled or failed:', err.message));
```

## Execution Lifecycle

The request execution follows this lifecycle:

1. **Build Phase**: Chain configuration methods (`.set()`, `.send()`, `.query()`, etc.)
2. **Trigger Phase**: Call `.end()`, `.then()`, or `.catch()` to start execution
3. **Request Phase**: Send HTTP request with configured parameters
4. **Response Phase**: Receive and parse response
5. **Retry Phase** (if configured): Retry on failure based on retry logic
6. **Callback Phase**: Invoke success or error handler

```javascript
// Lifecycle example with logging
request
  .get('/api/users')
  .on('request', req => console.log('1. Request started'))
  .on('response', res => console.log('2. Response received'))
  .on('end', () => console.log('3. Request completed'))
  .then(res => {
    console.log('4. Success handler');
    return res.body;
  })
  .catch(err => {
    console.log('4. Error handler');
    throw err;
  });
```

## Best Practices

### Always Handle Errors

```javascript
// Good
request
  .get('/api/data')
  .then(res => console.log(res.body))
  .catch(err => console.error('Error:', err));

// Bad (unhandled rejection)
request
  .get('/api/data')
  .then(res => console.log(res.body));
```

### Use Timeouts for External APIs

```javascript
request
  .get('https://external-api.com/data')
  .timeout({ response: 5000, deadline: 10000 })
  .retry(2)
  .then(res => console.log(res.body))
  .catch(err => console.error('Failed:', err));
```

### Don't Mix .end() and Promises

```javascript
// Bad - sends request twice
request
  .get('/api/data')
  .end((err, res) => console.log(res))
  .then(res => console.log(res));

// Good - use one or the other
request
  .get('/api/data')
  .then(res => console.log(res));
```

### Abort Long-Running Requests

```javascript
const controller = new AbortController();
const req = request.get('/api/long-task');

// Abort on signal
controller.signal.addEventListener('abort', () => {
  req.abort();
});

// Abort after timeout
setTimeout(() => controller.abort(), 30000);
```
