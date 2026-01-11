# HTTP Requests

Core functionality for making HTTP requests using SuperAgent's fluent API.

## Capabilities

### Main Request Function

Creates a new HTTP request with the specified method and URL.

```javascript { .api }
/**
 * Create a new HTTP request
 * @param {string} method - HTTP method (GET, POST, PUT, DELETE, etc.)
 * @param {string} url - Request URL
 * @returns {Request} Request instance for chaining
 */
function request(method, url);

/**
 * Create a GET request (shorthand)
 * @param {string} url - Request URL
 * @returns {Request} Request instance for chaining
 */
function request(url);
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Full form
const req = request('GET', '/api/users');

// Shorthand for GET
const req = request('/api/users');

// With callback
request('GET', '/api/users').end((err, res) => {
  console.log(res.body);
});
```

### HTTP Method Helpers

Convenience functions for common HTTP methods. All accept optional data and callback parameters.

```javascript { .api }
/**
 * Perform GET request
 * @param {string} url - Request URL
 * @param {object|Function} [data] - Query parameters or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.get(url, data, callback);

/**
 * Perform POST request
 * @param {string} url - Request URL
 * @param {any|Function} [data] - Request body or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.post(url, data, callback);

/**
 * Perform PUT request
 * @param {string} url - Request URL
 * @param {any|Function} [data] - Request body or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.put(url, data, callback);

/**
 * Perform PATCH request
 * @param {string} url - Request URL
 * @param {any|Function} [data] - Request body or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.patch(url, data, callback);

/**
 * Perform DELETE request
 * @param {string} url - Request URL
 * @param {any|Function} [data] - Request body or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.delete(url, data, callback);

/**
 * Perform DELETE request (alias)
 */
function request.del(url, data, callback);

/**
 * Perform HEAD request
 * @param {string} url - Request URL
 * @param {object|Function} [data] - Query parameters or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.head(url, data, callback);

/**
 * Perform OPTIONS request
 * @param {string} url - Request URL
 * @param {any|Function} [data] - Request body or callback
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
function request.options(url, data, callback);
```

### Additional HTTP Method Helpers

SuperAgent provides helper functions for all standard and WebDAV HTTP methods.

```javascript { .api }
/**
 * Additional HTTP method helpers
 * All follow the same signature pattern as above
 */
function request.checkout(url, data, callback);
function request.connect(url, data, callback);
function request.copy(url, data, callback);
function request.lock(url, data, callback);
function request.merge(url, data, callback);
function request.mkactivity(url, data, callback);
function request.mkcol(url, data, callback);
function request.move(url, data, callback);
function request.notify(url, data, callback);
function request.propfind(url, data, callback);
function request.proppatch(url, data, callback);
function request.purge(url, data, callback);
function request.report(url, data, callback);
function request.search(url, data, callback);
function request.subscribe(url, data, callback);
function request.trace(url, data, callback);
function request.unlock(url, data, callback);
function request.unsubscribe(url, data, callback);
```

**Usage Examples:**

```javascript
// GET request with query parameters
request
  .get('/api/users')
  .query({ page: 1, limit: 10 })
  .end((err, res) => {
    console.log(res.body);
  });

// GET request with inline query and callback
request.get('/api/users', { page: 1 }, (err, res) => {
  console.log(res.body);
});

// POST request with body
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' })
  .end((err, res) => {
    console.log(res.body);
  });

// POST request with inline body and callback
request.post('/api/users', { name: 'John' }, (err, res) => {
  console.log(res.body);
});

// PUT request
request
  .put('/api/users/123')
  .send({ name: 'Jane' })
  .end((err, res) => {
    console.log(res.body);
  });

// PATCH request
request
  .patch('/api/users/123')
  .send({ email: 'newemail@example.com' })
  .end((err, res) => {
    console.log(res.body);
  });

// DELETE request
request
  .delete('/api/users/123')
  .end((err, res) => {
    console.log(res.status); // 204 No Content
  });

// HEAD request
request
  .head('/api/users/123')
  .end((err, res) => {
    console.log(res.header); // Headers only, no body
  });

// OPTIONS request
request
  .options('/api/users')
  .end((err, res) => {
    console.log(res.header['allow']); // Allowed methods
  });

// WebDAV methods examples
request
  .propfind('/dav/folder')
  .end((err, res) => {
    console.log(res.body); // WebDAV properties
  });

request
  .copy('/dav/file.txt')
  .set('Destination', '/dav/file-copy.txt')
  .end(callback);

request
  .move('/dav/old-name.txt')
  .set('Destination', '/dav/new-name.txt')
  .end(callback);
```

### Request Execution

Methods to execute the request and handle responses.

```javascript { .api }
/**
 * Send the request and invoke callback with response
 * @param {Function} [callback] - Callback function (err, res)
 * @returns {Request} Request instance
 */
Request.prototype.end = function(callback);

/**
 * Abort the request
 * @returns {Request} Request instance
 */
Request.prototype.abort = function();
```

**Usage Examples:**

```javascript
// Execute with callback
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Error:', err.message);
      console.error('Status:', err.status);
      return;
    }
    console.log('Success:', res.body);
  });

// Execute without callback (use promises instead)
request
  .get('/api/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });

// Abort request
const req = request.get('/api/large-file');
req.end((err, res) => {
  console.log('Done');
});

// Abort after 1 second
setTimeout(() => {
  req.abort();
}, 1000);
```

### Request Constructor

Direct instantiation of Request class (typically not used directly).

```javascript { .api }
/**
 * Request class constructor
 * @param {string} method - HTTP method
 * @param {string} url - Request URL
 * @returns {Request} Request instance
 */
function Request(method, url);
```

## Query Parameters

Add query string parameters to the request URL.

```javascript { .api }
/**
 * Add query string parameters
 * @param {object|string} params - Query parameters as object or string
 * @returns {Request} Request instance for chaining
 */
Request.prototype.query = function(params);

/**
 * Sort query string parameters
 * @param {Function} [comparator] - Optional custom sort function
 * @returns {Request} Request instance for chaining
 */
Request.prototype.sortQuery = function(comparator);
```

**Usage Examples:**

```javascript
// Query with object
request
  .get('/api/users')
  .query({ page: 1, limit: 10 })
  .end(callback);

// Query with string
request
  .get('/api/users')
  .query('page=1&limit=10')
  .end(callback);

// Multiple query calls merge parameters
request
  .get('/api/users')
  .query({ page: 1 })
  .query({ limit: 10 })
  .query({ sort: 'name' })
  .end(callback);
// Results in: /api/users?page=1&limit=10&sort=name

// Sort query parameters alphabetically
request
  .get('/api/users')
  .query({ z: 1, a: 2, m: 3 })
  .sortQuery()
  .end(callback);
// Results in: /api/users?a=2&m=3&z=1

// Custom sort function
request
  .get('/api/users')
  .query({ z: 1, a: 2 })
  .sortQuery((a, b) => b.localeCompare(a)) // Reverse sort
  .end(callback);
```

## Request Body

Send data in the request body.

```javascript { .api }
/**
 * Send request body data
 * @param {any} data - Body data (object, string, Buffer, etc.)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.send = function(data);
```

**Usage Examples:**

```javascript
// Send JSON object
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' })
  .end(callback);

// Send string
request
  .post('/api/data')
  .type('text')
  .send('raw text data')
  .end(callback);

// Send form data
request
  .post('/api/form')
  .type('form')
  .send({ name: 'John' })
  .send({ email: 'john@example.com' }) // Multiple sends merge for objects
  .end(callback);

// Send Buffer (Node.js)
const buffer = Buffer.from('binary data');
request
  .post('/api/upload')
  .send(buffer)
  .end(callback);

// GET request with data goes to query string
request
  .get('/api/users')
  .send({ page: 1 }) // Equivalent to .query({ page: 1 })
  .end(callback);
```

## URL Handling

SuperAgent supports various URL formats including absolute, relative, and Unix socket URLs.

**Usage Examples:**

```javascript
// Absolute URL
request.get('https://api.example.com/users');

// Relative URL
request.get('/api/users');

// URL without protocol defaults to http://
request.get('example.com/api'); // Becomes http://example.com/api

// URL with authentication
request.get('http://user:pass@example.com/api');
// Equivalent to:
request.get('http://example.com/api').auth('user', 'pass');
```

### Unix Domain Sockets

Connect to Unix domain sockets for IPC (Inter-Process Communication) on Node.js.

**Usage Examples:**

```javascript
// Unix socket with HTTP (Node.js only)
request.get('http+unix:///var/run/docker.sock/containers/json');

// Unix socket with HTTPS (Node.js only)
request.get('https+unix:///var/run/secure.sock/api/data');

// URL-encode the socket path for special characters
const socketPath = encodeURIComponent('/var/run/my app.sock');
request.get(`http+unix://${socketPath}/api/endpoint`);

// Example: Docker API via Unix socket
request
  .get('http+unix:///var/run/docker.sock/containers/json')
  .query({ all: true })
  .end((err, res) => {
    console.log('Docker containers:', res.body);
  });

// Example: Custom application socket
request
  .post('http+unix:///tmp/app.sock/api/command')
  .send({ action: 'restart' })
  .end(callback);
```

## Redirects

Configure redirect following behavior.

```javascript { .api }
/**
 * Set maximum number of redirects to follow
 * @param {number} count - Maximum redirects (0 disables)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.redirects = function(count);
```

**Usage Examples:**

```javascript
// Default: 5 redirects for most methods, 0 for HEAD
request.get('/api/redirect');

// Disable redirects
request
  .get('/api/redirect')
  .redirects(0)
  .end(callback);

// Allow up to 10 redirects
request
  .get('/api/redirect')
  .redirects(10)
  .end(callback);

// Access redirect history
request
  .get('/api/redirect')
  .end((err, res) => {
    console.log(res.redirects); // Array of redirect URLs
  });
```

## Request Utilities

```javascript { .api }
/**
 * Convert request to JSON representation
 * @returns {object} JSON representation of request
 */
Request.prototype.toJSON = function();
```

**Usage Examples:**

```javascript
const req = request
  .get('/api/users')
  .query({ page: 1 });

const json = req.toJSON();
console.log(json);
// { method: 'GET', url: '/api/users?page=1', headers: {...} }
```
