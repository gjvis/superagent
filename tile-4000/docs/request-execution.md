# Request Execution

Execute requests using callbacks, promises, or async/await patterns.

## Capabilities

### Callback Execution

Execute the request using a callback function.

```javascript { .api }
/**
 * Execute the request with a callback
 * @param callback - Callback function receiving (err, res)
 * @returns Request instance
 */
Request.prototype.end(callback?: (err: Error | null, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Basic callback
request.get('https://api.example.com/users')
  .end((err, res) => {
    if (err) {
      console.error('Error:', err.message);
      return;
    }
    console.log('Success:', res.body);
  });

// Error handling in callback
request.post('https://api.example.com/users')
  .send({ name: 'John' })
  .end((err, res) => {
    if (err) {
      // Network error or HTTP error (4xx, 5xx)
      console.error('Status:', err.status);
      console.error('Message:', err.message);
      if (err.response) {
        console.error('Response body:', err.response.body);
      }
      return;
    }
    console.log('User created:', res.body);
  });

// Calling .end() without callback
request.get('https://api.example.com/users')
  .end(); // Fires the request but doesn't handle response
```

### Promise Execution

Execute the request using promises with `.then()`.

```javascript { .api }
/**
 * Execute the request as a promise
 * @param resolve - Success handler receiving response
 * @param reject - Error handler receiving error
 * @returns Promise resolving to Response
 */
Request.prototype.then(resolve: (res: Response) => any, reject?: (err: Error) => any): Promise;
```

**Usage Examples:**

```javascript
// Basic promise
request.get('https://api.example.com/users')
  .then(res => {
    console.log('Success:', res.body);
  });

// Promise with error handler
request.get('https://api.example.com/users')
  .then(
    res => {
      console.log('Success:', res.body);
    },
    err => {
      console.error('Error:', err.message);
    }
  );

// Chaining promises
request.post('https://api.example.com/users')
  .send({ name: 'John' })
  .then(res => {
    console.log('User created:', res.body);
    return request.get(`https://api.example.com/users/${res.body.id}`);
  })
  .then(res => {
    console.log('User details:', res.body);
  });

// Promise chaining with data transformation
request.get('https://api.example.com/users')
  .then(res => res.body)
  .then(users => users.filter(u => u.active))
  .then(activeUsers => {
    console.log('Active users:', activeUsers);
  });
```

### Promise Error Handling

Handle errors using `.catch()`.

```javascript { .api }
/**
 * Catch promise rejections
 * @param reject - Error handler receiving error
 * @returns Promise
 */
Request.prototype.catch(reject: (err: Error) => any): Promise;
```

**Usage Examples:**

```javascript
// Basic error handling
request.get('https://api.example.com/users')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });

// Detailed error handling
request.post('https://api.example.com/users')
  .send({ name: 'John' })
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    if (err.status === 400) {
      console.error('Validation error:', err.response.body);
    } else if (err.status === 401) {
      console.error('Authentication required');
    } else if (err.timeout) {
      console.error('Request timeout');
    } else {
      console.error('Network error:', err.message);
    }
  });

// Rethrowing errors
request.get('https://api.example.com/users')
  .catch(err => {
    console.error('Request failed, logging...');
    logError(err);
    throw err; // Rethrow for upstream handling
  });
```

### Async/Await Execution

Execute requests using modern async/await syntax.

**Usage Examples:**

```javascript
// Basic async/await
async function getUsers() {
  const res = await request.get('https://api.example.com/users');
  return res.body;
}

// With try/catch error handling
async function createUser(userData) {
  try {
    const res = await request.post('https://api.example.com/users')
      .send(userData);
    console.log('User created:', res.body);
    return res.body;
  } catch (err) {
    console.error('Failed to create user:', err.message);
    throw err;
  }
}

// Sequential requests
async function getUserWithPosts(userId) {
  const userRes = await request.get(`https://api.example.com/users/${userId}`);
  const postsRes = await request.get(`https://api.example.com/users/${userId}/posts`);

  return {
    user: userRes.body,
    posts: postsRes.body
  };
}

// Parallel requests
async function getDashboardData() {
  const [usersRes, postsRes, commentsRes] = await Promise.all([
    request.get('https://api.example.com/users'),
    request.get('https://api.example.com/posts'),
    request.get('https://api.example.com/comments')
  ]);

  return {
    users: usersRes.body,
    posts: postsRes.body,
    comments: commentsRes.body
  };
}

// Error handling with status checks
async function fetchData(url) {
  try {
    const res = await request.get(url);
    return res.body;
  } catch (err) {
    if (err.status === 404) {
      return null; // Resource not found
    }
    if (err.status >= 500) {
      // Retry on server errors
      return fetchData(url);
    }
    throw err;
  }
}
```

## Execution Patterns

### Fire and Forget

Execute request without handling response.

```javascript
// Just trigger the request
request.post('https://api.example.com/analytics')
  .send({ event: 'page_view' })
  .end();

// Or with promises (but not awaiting)
request.post('https://api.example.com/analytics')
  .send({ event: 'page_view' })
  .then(() => {})
  .catch(() => {}); // Swallow errors
```

### Conditional Execution

Execute requests based on conditions.

```javascript
async function fetchUser(userId, includeDetails = false) {
  const userRes = await request.get(`https://api.example.com/users/${userId}`);

  if (includeDetails) {
    const detailsRes = await request.get(`https://api.example.com/users/${userId}/details`);
    return { ...userRes.body, details: detailsRes.body };
  }

  return userRes.body;
}
```

### Retry with Exponential Backoff

```javascript
async function fetchWithBackoff(url, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const res = await request.get(url);
      return res.body;
    } catch (err) {
      if (i === maxRetries - 1) throw err;

      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Request Cancellation

```javascript
// Store request reference to abort later
const req = request.get('https://api.example.com/large-data')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    if (err.message === 'Aborted') {
      console.log('Request was cancelled');
    }
  });

// Cancel the request
setTimeout(() => {
  req.abort();
}, 5000); // Cancel after 5 seconds
```

## Important Notes

### Don't Mix Execution Methods

```javascript
// Bad: calling both .end() and .then()
request.get('https://api.example.com/users')
  .end((err, res) => {
    console.log('Callback:', res.body);
  })
  .then(res => {
    console.log('Promise:', res.body); // Will execute twice!
  });

// Good: use one method
request.get('https://api.example.com/users')
  .then(res => {
    console.log('Promise:', res.body);
  });

// Or
request.get('https://api.example.com/users')
  .end((err, res) => {
    if (err) return console.error(err);
    console.log('Callback:', res.body);
  });
```

### Request is Only Sent on Execution

```javascript
// This does NOT send the request (just builds it)
const req = request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer token');

// Request is sent when you call .end(), .then(), or .catch()
req.end(); // Now the request is sent

// Or
req.then(res => console.log(res.body)); // Request sent here
```

### Error Types and Properties

Errors can occur from:
- **Network failures**: Connection refused, DNS lookup failed, timeout
- **HTTP errors**: 4xx and 5xx status codes (unless custom `.ok()` validator)
- **Parse errors**: Invalid JSON or malformed response body
- **Abort**: Request was manually aborted

```javascript { .api }
/**
 * Error object properties
 */
interface SuperAgentError extends Error {
  /** Error message describing what went wrong */
  message: string;

  /** HTTP status code (if available, e.g., 404, 500) */
  status?: number;

  /** Response object for HTTP errors */
  response?: Response;

  /** Timeout value for timeout errors (in milliseconds) */
  timeout?: number;

  /** Error code for network/system errors (e.g., 'ECONNREFUSED', 'ETIMEDOUT', 'ABORTED') */
  code?: string;

  /** System errno for low-level errors */
  errno?: number;
}
```

**Usage Examples:**

```javascript
request.get('https://api.example.com/users')
  .catch(err => {
    console.log('Error properties:');
    console.log('- message:', err.message);        // 'Not Found' or 'connect ECONNREFUSED'
    console.log('- status:', err.status);          // 404 (HTTP errors only)
    console.log('- response:', err.response);      // Response object (HTTP errors only)
    console.log('- timeout:', err.timeout);        // 5000 (timeout errors only)
    console.log('- code:', err.code);              // 'ECONNREFUSED', 'ETIMEDOUT', 'ABORTED'
    console.log('- errno:', err.errno);            // System error number
  });

// Detailed error handling
request.post('https://api.example.com/users')
  .send({ name: 'John' })
  .catch(err => {
    // Network errors
    if (err.code === 'ECONNREFUSED') {
      console.error('Connection refused - server not running?');
    } else if (err.code === 'ETIMEDOUT' || err.timeout) {
      console.error('Request timed out after', err.timeout, 'ms');
    } else if (err.code === 'ENOTFOUND') {
      console.error('DNS lookup failed - invalid hostname');
    } else if (err.code === 'ABORTED') {
      console.error('Request was aborted');
    }

    // HTTP errors
    else if (err.status) {
      console.error('HTTP error:', err.status, err.message);
      if (err.response) {
        console.error('Response body:', err.response.body);
        console.error('Response headers:', err.response.headers);
      }

      // Specific status codes
      if (err.status === 400) {
        console.error('Bad request - validation errors:', err.response.body);
      } else if (err.status === 401) {
        console.error('Unauthorized - authentication required');
      } else if (err.status === 403) {
        console.error('Forbidden - insufficient permissions');
      } else if (err.status === 404) {
        console.error('Not found - resource does not exist');
      } else if (err.status === 422) {
        console.error('Unprocessable entity:', err.response.body);
      } else if (err.status >= 500) {
        console.error('Server error - try again later');
      }
    }

    // Parse errors
    else if (err.message.includes('JSON')) {
      console.error('Failed to parse response as JSON');
    }

    // Unknown errors
    else {
      console.error('Unknown error:', err.message);
    }
  });
```
