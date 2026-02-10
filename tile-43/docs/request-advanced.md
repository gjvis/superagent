# Advanced Request Options

This document covers advanced request configuration including authentication, SSL/TLS certificates, timeouts, retries, redirects, and other specialized options.

## Capabilities

### Authentication

Set authentication credentials using Basic, Bearer, or automatic authentication.

```javascript { .api }
/**
 * Set authentication header
 * @param user - Username or token
 * @param pass - Password (optional, empty string if omitted)
 * @param options - Authentication options
 * @returns Request instance for chaining
 */
auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Request;
```

**Authentication types:**
- `'basic'` (default): Basic HTTP authentication with base64-encoded `user:pass`
- `'bearer'`: Bearer token authentication (pass token as `user` parameter)
- `'auto'`: Stores credentials and sends with redirect requests

**Usage Examples:**

```javascript
// Basic authentication
request
  .get('/api/users')
  .auth('username', 'password');
// Sets: Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

// Basic auth with explicit type
request
  .get('/api/users')
  .auth('username', 'password', { type: 'basic' });

// Bearer token authentication
request
  .get('/api/users')
  .auth('my-token-123', { type: 'bearer' });
// Sets: Authorization: Bearer my-token-123

// Auto authentication (preserves credentials on redirects)
request
  .get('/api/users')
  .auth('username', 'password', { type: 'auto' });

// Auth with single argument (password defaults to empty string)
request
  .get('/api/users')
  .auth('username');

// Parse user:pass from string
request
  .get('/api/users')
  .auth('username:password');
```

### Timeouts

Set request and response timeouts to prevent hanging requests.

```javascript { .api }
/**
 * Set deadline timeout (total time for entire request)
 * @param ms - Timeout in milliseconds (0 or false disables)
 * @returns Request instance for chaining
 */
timeout(ms: number): Request;

/**
 * Set specific timeouts for different phases
 * @param options - Timeout configuration object
 * @param options.response - Time to wait for first byte of response (includes DNS and connection time)
 * @param options.deadline - Total time from request start to complete response
 * @returns Request instance for chaining
 */
timeout(options: { response?: number, deadline?: number }): Request;
```

**Usage Examples:**

```javascript
// Set deadline timeout (entire request must complete within 5 seconds)
request
  .get('/api/users')
  .timeout(5000);

// Set response timeout (first byte must arrive within 3 seconds)
request
  .get('/api/users')
  .timeout({ response: 3000 });

// Set both timeouts
request
  .get('/api/large-file')
  .timeout({
    response: 5000,   // 5 seconds to get first byte
    deadline: 60000   // 60 seconds total
  });

// Disable timeout
request
  .get('/api/users')
  .timeout(0);
```

**Timeout errors:**
- Response timeout throws error with `.timeout` property and `.code` = `'ETIMEDOUT'`
- Deadline timeout throws error with `.timeout` property and `.code` = `'ETIME'`

### Clear Timeout

Manually clear any set timeouts.

```javascript { .api }
/**
 * Clear request timeouts
 * @returns Request instance for chaining
 */
clearTimeout(): Request;
```

**Usage Example:**

```javascript
const req = request
  .get('/api/users')
  .timeout(5000);

// Later, clear the timeout
req.clearTimeout();
```

### Retries

Automatically retry failed requests with configurable retry logic.

```javascript { .api }
/**
 * Set number of retry attempts on error
 * @param count - Number of retries (default 1 if true, 0 if omitted)
 * @param callback - Optional function to determine if retry should occur
 * @returns Request instance for chaining
 */
retry(count?: number, callback?: (err: Error, res: Response) => boolean): Request;
```

**Default retry conditions:**
- 5xx status codes (except 501)
- Network errors: `ECONNRESET`, `ETIMEDOUT`, `EADDRINFO`, `ESOCKETTIMEDOUT`
- SuperAgent timeout errors with code `ECONNABORTED`
- Cross-domain errors

**Usage Examples:**

```javascript
// Retry up to 2 times on failure
request
  .get('/api/users')
  .retry(2);

// Retry with default count (1)
request
  .get('/api/users')
  .retry();

// Custom retry logic
request
  .get('/api/users')
  .retry(3, (err, res) => {
    // Only retry on 503 Service Unavailable
    if (res && res.status === 503) {
      return true;  // Retry
    }
    return false;  // Don't retry
  });

// Disable retries
request
  .get('/api/users')
  .retry(0);
```

**Error information:**
When a request fails after retries, the error object includes:
- `.retries` - Number of retries attempted

### Redirects

Configure maximum number of redirects to follow.

```javascript { .api }
/**
 * Set maximum number of redirects to follow
 * @param count - Max redirects (default is 5 for most methods, 0 for HEAD)
 * @returns Request instance for chaining
 */
redirects(count: number): Request;
```

**Redirect behavior:**
- 301, 302: Changes method to GET, clears request body
- 303: Forces GET method, clears request body
- 307, 308: Preserves original method and body
- Authorization headers are removed when redirecting to different host

**Usage Examples:**

```javascript
// Follow up to 10 redirects
request
  .get('/api/users')
  .redirects(10);

// Disable redirects
request
  .get('/api/users')
  .redirects(0);

// Default is 5 redirects for most methods
request.get('/api/users');  // Max 5 redirects

// HEAD requests default to 0 redirects
request.head('/api/users');  // No redirects by default
```

**Redirect events:**
The request emits a `'redirect'` event on each redirect with the response object:

```javascript
request
  .get('/api/users')
  .on('redirect', (res) => {
    console.log('Redirecting to:', res.headers.location);
  });
```

**Response redirect information:**
- `response.redirects` - Array of all redirect URLs (Node.js only)

### SSL/TLS Certificate Configuration (Node.js only)

Configure SSL/TLS certificates for HTTPS requests in Node.js.

```javascript { .api }
/**
 * Set Certificate Authority
 * @param cert - CA certificate (Buffer or String)
 * @returns Request instance for chaining
 */
ca(cert: Buffer | string): Request;

/**
 * Set client private key
 * @param cert - Private key (Buffer or String)
 * @returns Request instance for chaining
 */
key(cert: Buffer | string): Request;

/**
 * Set client certificate
 * @param cert - Client certificate (Buffer or String)
 * @returns Request instance for chaining
 */
cert(cert: Buffer | string): Request;

/**
 * Set PFX or PKCS12 encoded certificate and private key
 * @param cert - PFX certificate (Buffer or String) or object with pfx and passphrase
 * @returns Request instance for chaining
 */
pfx(cert: Buffer | string | { pfx: Buffer | string, passphrase: string }): Request;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Set CA certificate
request
  .get('https://example.com')
  .ca(fs.readFileSync('ca-cert.pem'));

// Set client key and certificate
request
  .get('https://example.com')
  .key(fs.readFileSync('client-key.pem'))
  .cert(fs.readFileSync('client-cert.pem'));

// Use PFX certificate
request
  .get('https://example.com')
  .pfx(fs.readFileSync('client.pfx'));

// PFX with passphrase
request
  .get('https://example.com')
  .pfx({
    pfx: fs.readFileSync('client.pfx'),
    passphrase: 'secret123'
  });
```

### HTTP Agent Configuration (Node.js only)

Set a custom HTTP agent for connection pooling and other low-level options.

```javascript { .api }
/**
 * Set HTTP agent for this request
 * @param agent - http.Agent or https.Agent instance, or false to disable pooling
 * @returns Request instance for chaining when called with argument
 * @returns Current agent when called without argument
 */
agent(agent: http.Agent | https.Agent | false): Request;
agent(): http.Agent | https.Agent | false;
```

**Default behavior:**
SuperAgent opts out of connection pooling by default (`agent: false`).

**Usage Examples:**

```javascript
const http = require('http');
const https = require('https');

// Use custom HTTP agent with connection pooling
const keepaliveAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 50
});

request
  .get('http://example.com')
  .agent(keepaliveAgent);

// Use custom HTTPS agent
const httpsAgent = new https.Agent({
  keepAlive: true,
  rejectUnauthorized: false  // Disable certificate validation (not recommended in production)
});

request
  .get('https://example.com')
  .agent(httpsAgent);

// Explicitly disable connection pooling
request
  .get('http://example.com')
  .agent(false);

// Get current agent
const req = request.get('http://example.com').agent(keepaliveAgent);
console.log(req.agent());  // Returns keepaliveAgent
```

### HTTP/2 Support (Node.js only)

Enable or disable HTTP/2 protocol support.

```javascript { .api }
/**
 * Enable or disable HTTP/2
 * @param enable - true to enable, false to disable (default true if omitted)
 * @returns Request instance for chaining
 */
http2(enable?: boolean): Request;
```

**Requirements:**
- Node.js with HTTP/2 support
- Will throw error if HTTP/2 is not available

**Usage Examples:**

```javascript
// Enable HTTP/2
request
  .get('https://example.com')
  .http2();

// Explicitly enable HTTP/2
request
  .get('https://example.com')
  .http2(true);

// Disable HTTP/2
request
  .get('https://example.com')
  .http2(false);
```

### CORS Credentials (Browser)

Enable transmission of cookies and credentials with cross-origin requests.

```javascript { .api }
/**
 * Enable credentials for CORS requests
 * @param enable - true to enable (default), false to disable
 * @returns Request instance for chaining
 */
withCredentials(enable?: boolean): Request;
```

**Requirements for CORS:**
- Server must not use wildcard in `Access-Control-Allow-Origin`
- Server must set `Access-Control-Allow-Credentials: true`

**Usage Examples:**

```javascript
// Enable credentials (browser only)
request
  .get('https://api.example.com/users')
  .withCredentials();

// Explicitly enable
request
  .get('https://api.example.com/users')
  .withCredentials(true);

// Disable credentials
request
  .get('https://api.example.com/users')
  .withCredentials(false);
```

**Note:** In Node.js, this method is a no-op and has no effect.

### Maximum Response Size

Limit the maximum size of response body to prevent memory exhaustion.

```javascript { .api }
/**
 * Set maximum response body size
 * @param bytes - Maximum size in bytes (default 200MB)
 * @returns Request instance for chaining
 */
maxResponseSize(bytes: number): Request;
```

**Usage Examples:**

```javascript
// Limit response to 1MB
request
  .get('/api/large-data')
  .maxResponseSize(1024 * 1024);

// Limit response to 10MB
request
  .get('/api/file')
  .maxResponseSize(10 * 1024 * 1024);
```

**Error handling:**
If response exceeds the limit, request fails with error:
- `.message` = `"Maximum response size reached"`
- `.code` = `"ETOOLARGE"`

### Custom Response Validation

Define custom logic to determine if a response should be treated as successful.

```javascript { .api }
/**
 * Set custom response validator
 * @param callback - Function that returns true if response is OK
 * @returns Request instance for chaining
 */
ok(callback: (res: Response) => boolean): Request;
```

**Default behavior:**
By default, responses with status 200-299 are considered successful.

**Usage Examples:**

```javascript
// Accept 404 as success
request
  .get('/api/users/123')
  .ok(res => res.status === 200 || res.status === 404)
  .then(res => {
    if (res.status === 404) {
      console.log('User not found');
    } else {
      console.log('User found:', res.body);
    }
  });

// Only accept 200
request
  .get('/api/users')
  .ok(res => res.status === 200);

// Accept any status (never throw error based on status)
request
  .get('/api/users')
  .ok(res => true);
```

### Abort Request

Abort an in-progress request and clear timeouts.

```javascript { .api }
/**
 * Abort the request
 * @returns Request instance
 */
abort(): Request;
```

**Usage Example:**

```javascript
const req = request
  .get('/api/large-file')
  .timeout(10000);

// Abort after 2 seconds
setTimeout(() => {
  req.abort();
}, 2000);

// Listen for abort event
req.on('abort', () => {
  console.log('Request aborted');
});
```

### Complete Example

```javascript
const request = require('superagent');
const fs = require('fs');

// Advanced request configuration
request
  .get('https://api.example.com/users')
  .auth('my-token', { type: 'bearer' })
  .timeout({
    response: 5000,
    deadline: 10000
  })
  .retry(3, (err, res) => {
    // Only retry on specific status codes
    return res && (res.status === 503 || res.status === 429);
  })
  .redirects(5)
  .ca(fs.readFileSync('ca-cert.pem'))
  .maxResponseSize(5 * 1024 * 1024)  // 5MB limit
  .ok(res => res.status < 500)  // Accept 4xx errors
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
    console.error('Retries attempted:', err.retries);
  });
```
