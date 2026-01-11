# Request Configuration

Configure request headers, content types, authentication, timeouts, and other request options.

## Capabilities

### Headers

Set and manage HTTP request headers.

```javascript { .api }
/**
 * Set request header(s)
 * @param {string|object} field - Header name or object of headers
 * @param {string} [value] - Header value (if field is string)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.set = function(field, value);

/**
 * Get request header value
 * @param {string} field - Header name
 * @returns {string} Header value
 */
Request.prototype.get = function(field);

/**
 * Get request header value (deprecated, use .get())
 * @param {string} field - Header name
 * @returns {string} Header value
 */
Request.prototype.getHeader = function(field);

/**
 * Remove request header
 * @param {string} field - Header name
 * @returns {Request} Request instance for chaining
 */
Request.prototype.unset = function(field);
```

**Usage Examples:**

```javascript
// Set single header
request
  .get('/api/users')
  .set('API-Key', 'secret123')
  .end(callback);

// Set multiple headers with object
request
  .get('/api/users')
  .set({
    'API-Key': 'secret123',
    'User-Agent': 'MyApp/1.0',
    'Accept-Language': 'en-US'
  })
  .end(callback);

// Chain multiple set calls
request
  .get('/api/users')
  .set('API-Key', 'secret123')
  .set('Accept', 'application/json')
  .end(callback);

// Get header value
const req = request.get('/api/users').set('API-Key', 'secret');
console.log(req.get('API-Key')); // 'secret'

// Remove header
request
  .get('/api/users')
  .set('API-Key', 'secret')
  .unset('API-Key')
  .end(callback);
```

### Content Type

Set the Content-Type header with shortcuts or full MIME types.

```javascript { .api }
/**
 * Set Content-Type header
 * @param {string} type - MIME type or shortcut (json, xml, form, html, etc.)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.type = function(type);
```

**Usage Examples:**

```javascript
// Using shortcuts
request
  .post('/api/users')
  .type('json') // Sets Content-Type: application/json
  .send({ name: 'John' })
  .end(callback);

request
  .post('/api/form')
  .type('form') // Sets Content-Type: application/x-www-form-urlencoded
  .send({ name: 'John' })
  .end(callback);

// Using full MIME types
request
  .post('/api/data')
  .type('application/json')
  .send({ data: 'value' })
  .end(callback);

request
  .post('/api/xml')
  .type('application/xml')
  .send('<user><name>John</name></user>')
  .end(callback);

// Common type shortcuts:
// - 'json' → 'application/json'
// - 'form' → 'application/x-www-form-urlencoded'
// - 'html' → 'text/html'
// - 'xml' → 'application/xml'
// - 'text' → 'text/plain'
```

### Accept Header

Set the Accept header to specify expected response content type.

```javascript { .api }
/**
 * Set Accept header
 * @param {string} type - MIME type or shortcut (json, xml, etc.)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.accept = function(type);
```

**Usage Examples:**

```javascript
// Using shortcuts
request
  .get('/api/users')
  .accept('json') // Sets Accept: application/json
  .end(callback);

// Using full MIME type
request
  .get('/api/users')
  .accept('application/json')
  .end(callback);

// Accept XML response
request
  .get('/api/data')
  .accept('xml')
  .end(callback);
```

### Authentication

Configure authentication for the request.

```javascript { .api }
/**
 * Set authentication
 * @param {string} user - Username or token
 * @param {string} [pass] - Password (optional for bearer tokens)
 * @param {object} [options] - Authentication options
 * @param {string} [options.type] - Authentication type: 'basic', 'bearer', or 'auto' (default: 'basic')
 * @returns {Request} Request instance for chaining
 */
Request.prototype.auth = function(user, pass, options);
```

**Usage Examples:**

```javascript
// Basic authentication
request
  .get('/api/protected')
  .auth('username', 'password')
  .end(callback);

// Basic auth with colon notation
request
  .get('/api/protected')
  .auth('username:password')
  .end(callback);

// Basic auth with user only
request
  .get('/api/protected')
  .auth('username')
  .end(callback);

// Bearer token authentication
request
  .get('/api/protected')
  .auth('my-access-token', { type: 'bearer' })
  .end(callback);

// Auto authentication (uses .username and .password properties)
const req = request.get('/api/protected');
req.username = 'user';
req.password = 'pass';
req.auth('', '', { type: 'auto' }).end(callback);

// Authentication can also be in URL
request.get('http://user:pass@example.com/api/protected');
```

### CORS Credentials

Enable sending cookies and authentication headers in cross-origin requests.

```javascript { .api }
/**
 * Enable CORS credentials
 * @param {boolean} [on] - Enable flag (default: true)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.withCredentials = function(on);
```

**Usage Examples:**

```javascript
// Enable credentials (sends cookies)
request
  .get('https://api.example.com/users')
  .withCredentials()
  .end(callback);

// Explicitly enable
request
  .get('https://api.example.com/users')
  .withCredentials(true)
  .end(callback);

// Disable credentials
request
  .get('https://api.example.com/users')
  .withCredentials(false)
  .end(callback);
```

### Timeouts

Configure request and response timeouts.

```javascript { .api }
/**
 * Set timeout
 * @param {number|object} options - Timeout in ms or options object
 * @param {number} [options.response] - Time to first byte (response timeout)
 * @param {number} [options.deadline] - Total time for complete response (deadline)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.timeout = function(options);

/**
 * Clear timeout
 * @returns {Request} Request instance for chaining
 */
Request.prototype.clearTimeout = function();
```

**Usage Examples:**

```javascript
// Simple timeout (deadline)
request
  .get('/api/users')
  .timeout(5000) // 5 seconds total
  .end(callback);

// Response timeout (time to first byte)
request
  .get('/api/users')
  .timeout({ response: 5000 }) // 5 seconds to receive first byte
  .end(callback);

// Deadline timeout (total time)
request
  .get('/api/users')
  .timeout({ deadline: 10000 }) // 10 seconds for complete response
  .end(callback);

// Both response and deadline timeouts
request
  .get('/api/users')
  .timeout({
    response: 5000,  // 5 seconds to first byte
    deadline: 10000  // 10 seconds total
  })
  .end(callback);

// Disable timeout
request
  .get('/api/users')
  .timeout(0) // or false
  .end(callback);

// Clear timeout manually
const req = request
  .get('/api/users')
  .timeout(5000);

req.clearTimeout(); // Remove timeout
req.end(callback);

// Handle timeout errors
request
  .get('/api/slow')
  .timeout(1000)
  .end((err, res) => {
    if (err && err.timeout) {
      console.error('Request timed out');
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
// Retry once on failure
request
  .get('/api/unreliable')
  .retry()
  .end(callback);

// Retry specific number of times
request
  .get('/api/unreliable')
  .retry(3) // Retry up to 3 times
  .end(callback);

// Custom retry logic
request
  .get('/api/unreliable')
  .retry(3, (err, res) => {
    // Return true to retry, false to stop
    if (err && err.code === 'ECONNRESET') {
      return true; // Retry on connection reset
    }
    if (res && res.status === 503) {
      return true; // Retry on service unavailable
    }
    return false; // Don't retry other errors
  })
  .end(callback);

// Default retry conditions:
// - Network errors: ECONNRESET, ETIMEDOUT, EADDRINFO, ESOCKETTIMEDOUT
// - HTTP status: 5xx (except 501)
// - Timeout errors

// Access retry count in error
request
  .get('/api/unreliable')
  .retry(3)
  .end((err, res) => {
    if (err) {
      console.log('Retries attempted:', err.retries);
    }
  });
```

### Response Validation

Override the default "OK" response validation.

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
// Default behavior: 2xx status codes are OK
request
  .get('/api/users')
  .end((err, res) => {
    // err is null for 2xx, Error for other status codes
  });

// Accept 404 as OK
request
  .get('/api/users/123')
  .ok(res => res.status === 200 || res.status === 404)
  .end((err, res) => {
    // err is null for 200 and 404
    if (res.status === 404) {
      console.log('User not found');
    }
  });

// Accept all status codes as OK
request
  .get('/api/users')
  .ok(res => true)
  .end((err, res) => {
    // err is only set for network errors
    console.log('Status:', res.status);
  });

// Custom validation logic
request
  .get('/api/data')
  .ok(res => {
    // Accept 200-299 or 304 (Not Modified)
    return (res.status >= 200 && res.status < 300) || res.status === 304;
  })
  .end(callback);
```

### Response Type

Set the expected response type for binary responses.

```javascript { .api }
/**
 * Set response type
 * @param {string} type - Response type ('blob', 'arraybuffer' in browser; any value results in Buffer in Node.js)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.responseType = function(type);
```

**Usage Examples:**

```javascript
// Browser: Get response as Blob
request
  .get('/api/image.png')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
  });

// Browser: Get response as ArrayBuffer
request
  .get('/api/binary-data')
  .responseType('arraybuffer')
  .end((err, res) => {
    const arrayBuffer = res.body;
  });

// Node.js: Get response as Buffer (any responseType value)
request
  .get('/api/image.png')
  .responseType('blob') // Results in Buffer
  .end((err, res) => {
    const buffer = res.body; // Buffer instance
  });
```

### Maximum Response Size

Set maximum allowed response body size (Node.js only).

```javascript { .api }
/**
 * Set maximum response size in bytes
 * @param {number} bytes - Maximum size (default: 200000000 / ~200MB)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.maxResponseSize = function(bytes);
```

**Usage Examples:**

```javascript
// Set 10MB limit
request
  .get('/api/large-file')
  .maxResponseSize(10 * 1024 * 1024) // 10 MB
  .end((err, res) => {
    if (err && err.code === 'ETOOLARGE') {
      console.error('Response too large');
    }
  });

// Set 1MB limit
request
  .get('/api/data')
  .maxResponseSize(1048576) // 1 MB
  .end(callback);
```

### Buffer Control

Enable or disable response buffering (Node.js only).

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param {boolean} [enable] - Enable buffering (default: true)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.buffer = function(enable);
```

**Usage Examples:**

```javascript
// Enable buffering (default)
request
  .get('/api/data')
  .buffer(true)
  .end((err, res) => {
    console.log(res.body); // Parsed response
  });

// Disable buffering (for streaming)
request
  .get('/api/large-file')
  .buffer(false)
  .end((err, res) => {
    // res.body will be undefined
    // Use streaming instead
  });
```

**Browser Compatibility:**

The `.buffer()` method is a no-op in browsers and will log a warning. Buffering control is only meaningful in Node.js where streaming is supported. In browsers, responses are always buffered.
