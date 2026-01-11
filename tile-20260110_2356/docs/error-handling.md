# Error Handling

Comprehensive error handling with automatic retry logic for network failures and server errors.

## Capabilities

### Error Object Properties

SuperAgent error objects provide detailed information about failures.

```javascript { .api }
/**
 * SuperAgent Error object
 */
interface SuperAgentError extends Error {
  message: string;          // Error message
  status?: number;          // HTTP status code (if response received)
  response?: Response;      // Response object (if response received)
  method?: string;          // HTTP method
  url?: string;             // Request URL
  timeout?: number;         // Timeout value (if timeout error)
  code?: string;            // Error code for network errors
  retries?: number;         // Number of retries attempted
  crossDomain?: boolean;    // True for CORS errors (Browser only)
  parse?: boolean;          // True for parse errors
  original?: Error;         // Original error for wrapped errors
  rawResponse?: string;     // Raw response text (Browser only, for parse errors)
}
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Basic error handling
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Error message:', err.message);
      console.error('Status code:', err.status);
      console.error('Method:', err.method);
      console.error('URL:', err.url);
    }
  });

// Check error type
request
  .get('/api/data')
  .end((err, res) => {
    if (err) {
      if (err.timeout) {
        console.error('Request timed out');
      } else if (err.code) {
        console.error('Network error:', err.code);
      } else if (err.status) {
        console.error('HTTP error:', err.status);
      }
    }
  });

// Access response on error
request
  .get('/api/users')
  .end((err, res) => {
    if (err && err.response) {
      console.error('Status:', err.response.status);
      console.error('Headers:', err.response.headers);
      console.error('Body:', err.response.body);
    }
  });
```

### HTTP Status Errors

Handle different HTTP status code errors.

**Usage Examples:**

```javascript
// Handle specific status codes
request
  .get('/api/users/123')
  .end((err, res) => {
    if (err) {
      switch (err.status) {
        case 400:
          console.error('Bad request:', err.response.body);
          break;
        case 401:
          console.error('Unauthorized - authentication required');
          break;
        case 403:
          console.error('Forbidden - access denied');
          break;
        case 404:
          console.error('User not found');
          break;
        case 422:
          console.error('Validation failed:', err.response.body.errors);
          break;
        case 429:
          console.error('Rate limit exceeded');
          break;
        case 500:
          console.error('Internal server error');
          break;
        case 503:
          console.error('Service unavailable');
          break;
        default:
          console.error('Request failed:', err.message);
      }
    }
  });

// Use response status flags
request
  .get('/api/data')
  .end((err, res) => {
    if (err) {
      if (err.response) {
        if (err.response.clientError) {
          console.error('Client error (4xx)');
        } else if (err.response.serverError) {
          console.error('Server error (5xx)');
        }

        // Specific status checks
        if (err.response.unauthorized) {
          console.error('Authentication required');
        }
        if (err.response.notFound) {
          console.error('Resource not found');
        }
      }
    }
  });

// With promises
try {
  const res = await request.get('/api/users/123');
  console.log('User:', res.body);
} catch (err) {
  if (err.status === 404) {
    console.error('User not found');
  } else if (err.status >= 500) {
    console.error('Server error');
  }
}
```

### Network Errors

Handle network and connection errors.

```javascript { .api }
/**
 * Common network error codes
 */
const ERROR_CODES = [
  'ECONNRESET',      // Connection reset by peer
  'ETIMEDOUT',       // Connection timeout
  'EADDRINFO',       // DNS lookup failed
  'ESOCKETTIMEDOUT', // Socket timeout
  'ECONNABORTED',    // Connection aborted
  'ECONNREFUSED',    // Connection refused
  'ENOTFOUND',       // Domain not found
  'ENETUNREACH'      // Network unreachable
];
```

**Usage Examples:**

```javascript
// Handle network errors
request
  .get('https://api.example.com/data')
  .end((err, res) => {
    if (err && err.code) {
      switch (err.code) {
        case 'ECONNRESET':
          console.error('Connection was reset');
          break;
        case 'ETIMEDOUT':
        case 'ESOCKETTIMEDOUT':
          console.error('Connection timed out');
          break;
        case 'EADDRINFO':
        case 'ENOTFOUND':
          console.error('Could not resolve hostname');
          break;
        case 'ECONNREFUSED':
          console.error('Connection refused by server');
          break;
        case 'ECONNABORTED':
          console.error('Connection aborted');
          break;
        case 'ENETUNREACH':
          console.error('Network unreachable');
          break;
        default:
          console.error('Network error:', err.code);
      }
    }
  });

// Check if error is network-related
function isNetworkError(err) {
  return err.code && [
    'ECONNRESET',
    'ETIMEDOUT',
    'ESOCKETTIMEDOUT',
    'EADDRINFO',
    'ENOTFOUND',
    'ECONNREFUSED',
    'ECONNABORTED'
  ].includes(err.code);
}

request
  .get('/api/data')
  .end((err, res) => {
    if (err && isNetworkError(err)) {
      console.error('Network connectivity issue');
    }
  });
```

### Timeout Errors

Handle request timeout errors.

**Usage Examples:**

```javascript
// Simple timeout
request
  .get('/api/slow')
  .timeout(5000)
  .end((err, res) => {
    if (err && err.timeout) {
      console.error('Request timed out after', err.timeout, 'ms');
    }
  });

// Response vs deadline timeout
request
  .get('/api/data')
  .timeout({
    response: 5000,  // Time to first byte
    deadline: 10000  // Total time
  })
  .end((err, res) => {
    if (err && err.timeout) {
      if (err.code === 'ECONNABORTED') {
        console.error('Deadline exceeded');
      } else {
        console.error('Response timeout');
      }
    }
  });

// With async/await
try {
  const res = await request
    .get('/api/data')
    .timeout(5000);
} catch (err) {
  if (err.timeout) {
    console.error('Request timed out');
    // Retry with longer timeout
    const res = await request.get('/api/data').timeout(10000);
  }
}
```

### Parse Errors

Handle response body parsing errors.

**Usage Examples:**

```javascript
// JSON parse error
request
  .get('/api/invalid-json')
  .end((err, res) => {
    if (err && err.parse) {
      console.error('Failed to parse response');
      console.error('Raw response:', err.rawResponse);  // Browser only
      console.error('Status:', err.status);

      // Try parsing manually
      if (res && res.text) {
        try {
          const data = JSON.parse(res.text);
          console.log('Manually parsed:', data);
        } catch (e) {
          console.error('Cannot parse response:', res.text);
        }
      }
    }
  });

// Handle with custom parser
request
  .get('/api/data')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      try {
        const parsed = JSON.parse(data);
        callback(null, parsed);
      } catch (err) {
        callback(err);
      }
    });
  })
  .end((err, res) => {
    if (err && err.parse) {
      console.error('Custom parser failed:', err.message);
    }
  });
```

### Retry Logic

Configure automatic retry on failure.

```javascript { .api }
/**
 * Set retry count and callback
 * @param {number} [count] - Number of retry attempts (default: 1)
 * @param {Function} [callback] - Custom retry decision function (err, res) => boolean
 * @returns {Request} Request instance for chaining
 */
Request.prototype.retry = function(count, callback);
```

**Usage Examples:**

```javascript
// Basic retry
request
  .get('/api/unreliable')
  .retry(3)  // Retry up to 3 times
  .end((err, res) => {
    if (err) {
      console.error('Failed after', err.retries, 'retries');
    }
  });

// Default retry behavior:
// - Retries on network errors (ECONNRESET, ETIMEDOUT, etc.)
// - Retries on 5xx status codes (except 501)
// - Retries on timeout errors

// Custom retry logic
request
  .get('/api/data')
  .retry(3, (err, res) => {
    // Return true to retry, false to stop
    if (err && err.code === 'ECONNRESET') {
      return true;  // Retry connection resets
    }

    if (res && res.status === 503) {
      return true;  // Retry service unavailable
    }

    if (res && res.status === 429) {
      // Check rate limit headers
      const retryAfter = res.get('Retry-After');
      if (retryAfter) {
        console.log('Rate limited, retry after:', retryAfter);
        return true;
      }
    }

    return false;  // Don't retry other cases
  })
  .end((err, res) => {
    if (err) {
      console.log('Retries attempted:', err.retries);
    }
  });

// Retry with exponential backoff (custom implementation)
async function requestWithBackoff(url, maxRetries = 3) {
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const res = await request.get(url);
      return res.body;
    } catch (err) {
      lastError = err;

      if (i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError;
}

// Only retry on specific conditions
request
  .get('/api/data')
  .retry(5, (err, res) => {
    // Only retry on network errors and 503
    if (err && isNetworkError(err)) {
      return true;
    }
    if (res && res.status === 503) {
      return true;
    }
    return false;
  })
  .end(callback);
```

### Override OK Validation

Customize which status codes are considered successful.

```javascript { .api }
/**
 * Override response OK validation
 * @param {Function} callback - Validation function (res) => boolean
 * @returns {Request} Request instance for chaining
 */
Request.prototype.ok = function(callback);
```

**Usage Examples:**

```javascript
// Accept 404 as OK
request
  .get('/api/users/123')
  .ok(res => res.status === 200 || res.status === 404)
  .end((err, res) => {
    // err is null for both 200 and 404
    if (res.status === 404) {
      console.log('User not found');
    } else {
      console.log('User:', res.body);
    }
  });

// Accept all status codes
request
  .get('/api/data')
  .ok(res => true)
  .end((err, res) => {
    // err only set for network errors
    console.log('Status:', res.status);
    console.log('Body:', res.body);
  });

// Custom validation logic
request
  .get('/api/data')
  .ok(res => {
    // Accept 2xx or 304 Not Modified
    return (res.status >= 200 && res.status < 300) || res.status === 304;
  })
  .end(callback);

// Accept 4xx errors but not 5xx
request
  .get('/api/data')
  .ok(res => res.status < 500)
  .end((err, res) => {
    if (err) {
      // Only 5xx errors or network errors
      console.error('Server error or network error');
    }
  });
```

### CORS Errors (Browser)

Handle cross-origin request errors in browsers.

**Usage Examples:**

```javascript
// Browser: Handle CORS errors
request
  .get('https://api.example.com/data')
  .withCredentials()
  .end((err, res) => {
    if (err && err.crossDomain) {
      console.error('CORS error: Cross-origin request blocked');
      console.error('Make sure server sends appropriate CORS headers');
    }
  });

// Check for CORS-related issues
function isCORSError(err) {
  return err.crossDomain || (err.status === 0 && !err.code);
}

request
  .get('https://api.example.com/data')
  .end((err, res) => {
    if (err && isCORSError(err)) {
      console.error('CORS error detected');
    }
  });
```

### Error Recovery Patterns

Common patterns for handling and recovering from errors.

**Usage Examples:**

```javascript
// 1. Fallback to default value
async function getUsersWithFallback() {
  try {
    const res = await request.get('/api/users');
    return res.body;
  } catch (err) {
    console.error('Failed to fetch users, using empty array');
    return [];
  }
}

// 2. Fallback to cached data
const cache = new Map();

async function getUsersWithCache(userId) {
  try {
    const res = await request.get(`/api/users/${userId}`);
    const user = res.body;

    cache.set(userId, user);
    return user;

  } catch (err) {
    const cached = cache.get(userId);
    if (cached) {
      console.log('Using cached data');
      return cached;
    }
    throw err;
  }
}

// 3. Graceful degradation
async function getUserData(userId) {
  const result = {
    user: null,
    posts: [],
    comments: []
  };

  // Try to get user
  try {
    const res = await request.get(`/api/users/${userId}`);
    result.user = res.body;
  } catch (err) {
    console.error('Failed to load user');
  }

  // Try to get posts (independent of user)
  try {
    const res = await request.get(`/api/posts?userId=${userId}`);
    result.posts = res.body;
  } catch (err) {
    console.error('Failed to load posts');
  }

  // Try to get comments (independent of user and posts)
  try {
    const res = await request.get(`/api/comments?userId=${userId}`);
    result.comments = res.body;
  } catch (err) {
    console.error('Failed to load comments');
  }

  return result;
}

// 4. Circuit breaker pattern
class CircuitBreaker {
  constructor(threshold = 5, resetTimeout = 60000) {
    this.failures = 0;
    this.threshold = threshold;
    this.resetTimeout = resetTimeout;
    this.isOpen = false;
    this.resetTimer = null;
  }

  async execute(requestFn) {
    if (this.isOpen) {
      throw new Error('Circuit breaker is open');
    }

    try {
      const result = await requestFn();
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  onSuccess() {
    this.failures = 0;
  }

  onFailure() {
    this.failures++;

    if (this.failures >= this.threshold) {
      this.open();
    }
  }

  open() {
    this.isOpen = true;
    console.error('Circuit breaker opened');

    this.resetTimer = setTimeout(() => {
      this.close();
    }, this.resetTimeout);
  }

  close() {
    this.isOpen = false;
    this.failures = 0;
    console.log('Circuit breaker closed');
  }
}

const breaker = new CircuitBreaker(3, 30000);

async function fetchWithBreaker() {
  return breaker.execute(() => request.get('/api/data'));
}

// 5. Retry with different endpoints
async function fetchFromMultipleEndpoints(endpoints) {
  let lastError;

  for (const endpoint of endpoints) {
    try {
      const res = await request.get(endpoint);
      return res.body;
    } catch (err) {
      console.error(`Failed to fetch from ${endpoint}:`, err.message);
      lastError = err;
    }
  }

  throw new Error('All endpoints failed: ' + lastError.message);
}

const endpoints = [
  'https://api1.example.com/data',
  'https://api2.example.com/data',
  'https://api3.example.com/data'
];

const data = await fetchFromMultipleEndpoints(endpoints);
```

### Complete Error Handling Example

Comprehensive example showing all error handling features.

**Usage Examples:**

```javascript
const request = require('superagent');

class APIClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }

  async request(method, path, data) {
    try {
      const req = request[method.toLowerCase()](this.baseUrl + path);

      if (data) {
        req.send(data);
      }

      req
        .timeout({ response: 5000, deadline: 10000 })
        .retry(3, (err, res) => {
          // Retry on network errors and 5xx
          if (err && isNetworkError(err)) {
            return true;
          }
          if (res && res.status >= 500) {
            return true;
          }
          return false;
        });

      const res = await req;
      return res.body;

    } catch (err) {
      return this.handleError(err);
    }
  }

  handleError(err) {
    // Log error details
    console.error('Request failed:', {
      message: err.message,
      status: err.status,
      method: err.method,
      url: err.url,
      code: err.code,
      retries: err.retries
    });

    // Network errors
    if (err.code) {
      throw new Error(`Network error: ${err.code}`);
    }

    // Timeout errors
    if (err.timeout) {
      throw new Error('Request timed out');
    }

    // Parse errors
    if (err.parse) {
      throw new Error('Failed to parse response');
    }

    // HTTP errors
    if (err.status) {
      switch (err.status) {
        case 400:
          throw new Error('Bad request: ' + JSON.stringify(err.response.body));
        case 401:
          throw new Error('Unauthorized: Authentication required');
        case 403:
          throw new Error('Forbidden: Access denied');
        case 404:
          throw new Error('Not found');
        case 422:
          throw new Error('Validation error: ' + JSON.stringify(err.response.body));
        case 429:
          const retryAfter = err.response.get('Retry-After');
          throw new Error(`Rate limit exceeded. Retry after: ${retryAfter}`);
        case 500:
          throw new Error('Internal server error');
        case 502:
          throw new Error('Bad gateway');
        case 503:
          throw new Error('Service unavailable');
        default:
          throw new Error(`HTTP error ${err.status}: ${err.message}`);
      }
    }

    // Unknown error
    throw err;
  }

  async get(path) {
    return this.request('GET', path);
  }

  async post(path, data) {
    return this.request('POST', path, data);
  }

  async put(path, data) {
    return this.request('PUT', path, data);
  }

  async delete(path) {
    return this.request('DELETE', path);
  }
}

function isNetworkError(err) {
  return err.code && [
    'ECONNRESET',
    'ETIMEDOUT',
    'ESOCKETTIMEDOUT',
    'EADDRINFO',
    'ENOTFOUND',
    'ECONNREFUSED',
    'ECONNABORTED'
  ].includes(err.code);
}

// Usage
const api = new APIClient('https://api.example.com');

async function main() {
  try {
    const users = await api.get('/users');
    console.log('Users:', users);

    const newUser = await api.post('/users', { name: 'John' });
    console.log('Created:', newUser);

  } catch (err) {
    console.error('API call failed:', err.message);
  }
}

main();
```

### Important Notes

- **Always Handle Errors**: Always include error handling in `.end()` callbacks or use `.catch()` with promises.
- **Error vs Response**: For 4xx/5xx status codes, both `err` and `err.response` are available. The response contains the server's error details.
- **Retry Strategy**: Default retry behavior handles network errors and 5xx status codes. Customize with retry callback for specific needs.
- **Network Errors**: Check `err.code` to identify network-level errors (DNS, connection, timeout).
- **Status Checking**: Use `err.status` for HTTP status codes or response status flags like `err.response.notFound`.
- **Parse Errors**: Check `err.parse` to identify JSON parsing errors. Raw response may be available in `err.rawResponse` (browser).
- **Timeout Types**: Response timeout (time to first byte) vs deadline timeout (total time). Both can trigger timeout errors.
- **CORS**: Browser-only `err.crossDomain` flag indicates cross-origin errors.
