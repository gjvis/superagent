# Making HTTP Requests

SuperAgent provides a fluent, chainable API for making HTTP requests. The library exports a main `request()` function and convenience methods for each HTTP verb. Requests can be created in multiple ways depending on your needs, with support for both modern promise-based and traditional callback-based patterns.

## Core Request Function

The `request()` function is the primary way to create HTTP requests. It supports multiple calling patterns for flexibility.

```javascript { .api }
/**
 * Create an HTTP request with the specified method and URL.
 *
 * @param {String} method - HTTP method (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS)
 * @param {String} url - Request URL
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Standard usage
 * request('GET', '/api/users')
 *
 * // Single argument (assumes GET method)
 * request('/api/users')
 *
 * // Callback as second argument (assumes GET method with first arg as URL)
 * request('/api/users', (err, res) => {
 *   console.log(res.body);
 * })
 */
function request(method, url)
```

### Usage Patterns

**Standard request creation:**
```javascript
const req = request('POST', '/api/users');
```

**Single argument pattern (assumes GET):**
```javascript
// When called with one argument, treats it as URL with GET method
const req = request('/api/users');
// Equivalent to: request('GET', '/api/users')
```

**Callback shorthand (assumes GET):**
```javascript
// When second argument is a function, treats first as URL
request('/api/users', (err, res) => {
  console.log(res.body);
});
// Equivalent to: request('GET', '/api/users').end(callback)
```

## HTTP Verb Convenience Methods

SuperAgent provides convenience methods for all standard HTTP verbs. These methods simplify request creation and support flexible parameter passing.

```javascript { .api }
/**
 * Make a GET request. Data is sent as query parameters.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Query parameters (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Basic GET
 * request.get('/api/users')
 *
 * // With query parameters
 * request.get('/api/users', { role: 'admin' })
 *
 * // With callback
 * request.get('/api/users', (err, res) => {})
 *
 * // With query parameters and callback
 * request.get('/api/users', { role: 'admin' }, (err, res) => {})
 */
function get(url, data, callback)

/**
 * Make a POST request. Data is sent as request body.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Basic POST
 * request.post('/api/users')
 *
 * // With body data
 * request.post('/api/users', { name: 'Alice', email: 'alice@example.com' })
 *
 * // With callback
 * request.post('/api/users', (err, res) => {})
 *
 * // With body data and callback
 * request.post('/api/users', { name: 'Alice' }, (err, res) => {})
 */
function post(url, data, callback)

/**
 * Make a PUT request. Data is sent as request body.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Update a user
 * request.put('/api/users/123', { name: 'Alice Updated' })
 *
 * // With callback
 * request.put('/api/users/123', { name: 'Alice' }, (err, res) => {})
 */
function put(url, data, callback)

/**
 * Make a PATCH request. Data is sent as request body.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Partially update a user
 * request.patch('/api/users/123', { email: 'newemail@example.com' })
 */
function patch(url, data, callback)

/**
 * Make a DELETE request. Data is sent as request body.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Delete a user
 * request.delete('/api/users/123')
 *
 * // With callback
 * request.delete('/api/users/123', (err, res) => {})
 */
function delete(url, data, callback)

/**
 * Alias for delete() to avoid JavaScript reserved word.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Using del() instead of delete()
 * request.del('/api/users/123')
 */
function del(url, data, callback)

/**
 * Make a HEAD request. Data is sent as query parameters.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Query parameters (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Check if resource exists
 * request.head('/api/users/123', (err, res) => {
 *   console.log('Status:', res.status);
 * })
 */
function head(url, data, callback)

/**
 * Make an OPTIONS request. Data is sent as request body.
 *
 * @param {String} url - Request URL
 * @param {Object|Function} [data] - Request body data (optional) or callback function
 * @param {Function} [callback] - Completion callback (err, res) (optional)
 * @return {Request} Request instance for chaining
 *
 * @example
 * // Check available methods for a resource
 * request.options('/api/users')
 */
function options(url, data, callback)
```

## Understanding Data Parameter Behavior

The HTTP verb methods handle the `data` parameter differently based on the HTTP method:

**GET and HEAD requests:**
- Data is sent as query parameters using `.query()`
- Appended to the URL as a query string

**POST, PUT, PATCH, DELETE, OPTIONS requests:**
- Data is sent as request body using `.send()`
- Serialized according to Content-Type header

```javascript
// GET: data becomes query parameters
request.get('/api/users', { role: 'admin', active: true })
// Results in: GET /api/users?role=admin&active=true

// POST: data becomes request body
request.post('/api/users', { name: 'Alice', email: 'alice@example.com' })
// Results in: POST /api/users with JSON body
```

## Callback Parameter Patterns

All HTTP verb methods support flexible callback positioning:

**Callback as second parameter:**
```javascript
// Omit data, provide callback directly
request.get('/api/users', (err, res) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log('Users:', res.body);
});
```

**Callback as third parameter:**
```javascript
// Provide both data and callback
request.post('/api/users', { name: 'Alice' }, (err, res) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log('Created user:', res.body);
});
```

**No callback (promise-based):**
```javascript
// Omit callback to use promises
const res = await request.get('/api/users');

// Or with .then()
request.get('/api/users')
  .then(res => {
    console.log('Users:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```

## Request Creation Patterns

### Pattern 1: Method Chaining (Most Common)

Build requests by chaining configuration methods:

```javascript
// POST request with headers and body
const res = await request
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('Content-Type', 'application/json')
  .send({ name: 'Alice', email: 'alice@example.com' });

// GET request with query parameters and headers
const res = await request
  .get('/api/search')
  .query({ q: 'nodejs', limit: 10 })
  .set('Accept', 'application/json');
```

### Pattern 2: Inline Data and Callback

Provide data and callback directly in the verb method:

```javascript
// POST with inline data and callback
request.post('/api/users', { name: 'Alice' }, (err, res) => {
  if (err) throw err;
  console.log(res.body);
});

// GET with inline query and callback
request.get('/api/search', { q: 'nodejs' }, (err, res) => {
  if (err) throw err;
  console.log(res.body);
});
```

### Pattern 3: Async/Await with Configuration

Combine async/await with method chaining:

```javascript
async function createUser(userData) {
  try {
    const res = await request
      .post('/api/users')
      .send(userData)
      .set('Authorization', 'Bearer token123')
      .timeout(5000)
      .retry(2);

    return res.body;
  } catch (err) {
    console.error('Failed to create user:', err.message);
    throw err;
  }
}
```

### Pattern 4: Reusable Request Objects

Create and configure request objects before sending:

```javascript
// Create request object
const req = request.post('/api/users');

// Configure it
req.set('Authorization', 'Bearer token123');
req.type('json');
req.timeout(5000);

// Conditionally add data
if (shouldIncludeEmail) {
  req.send({ name: 'Alice', email: 'alice@example.com' });
} else {
  req.send({ name: 'Alice' });
}

// Execute when ready
const res = await req;
```

### Pattern 5: One-Line Shorthand

Minimal syntax for simple requests:

```javascript
// Simple GET with callback
request.get('/api/users', (err, res) => { /* ... */ });

// Simple POST with data and callback
request.post('/api/users', { name: 'Alice' }, (err, res) => { /* ... */ });

// Simple GET with await
const res = await request.get('/api/users');

// Simple POST with data and await
const res = await request.post('/api/users', { name: 'Alice' });
```

## Sending Request Data

### Using .send()

The `.send()` method sends data in the request body. It can be called multiple times, and SuperAgent will intelligently merge the data.

```javascript
// Single send call
request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' });

// Multiple send calls (data is merged)
request
  .post('/api/users')
  .send({ name: 'Alice' })
  .send({ email: 'alice@example.com' })
  .send({ role: 'admin' });
// Results in: { name: 'Alice', email: 'alice@example.com', role: 'admin' }

// Sending strings
request
  .post('/api/data')
  .type('text/plain')
  .send('raw text data');

// Sending buffers (Node.js)
request
  .post('/api/upload')
  .send(Buffer.from('binary data'));
```

### Automatic Content-Type Detection

SuperAgent automatically sets the Content-Type based on the data:

```javascript
// Objects are sent as JSON (Content-Type: application/json)
request.post('/api/users')
  .send({ name: 'Alice' });

// URL-encoded form data (when Content-Type is set explicitly)
request.post('/api/login')
  .type('form')
  .send({ username: 'alice', password: 'secret' });

// String data preserves existing Content-Type or uses text/plain
request.post('/api/data')
  .send('plain text');
```

### Using Query Parameters

For GET and HEAD requests, use `.query()` to add query string parameters:

```javascript
// Single query call
request.get('/api/search')
  .query({ q: 'nodejs', limit: 10, offset: 0 });

// Multiple query calls (parameters are merged)
request.get('/api/search')
  .query({ q: 'nodejs' })
  .query({ limit: 10 })
  .query({ offset: 0 });
// Results in: /api/search?q=nodejs&limit=10&offset=0

// Query string format
request.get('/api/search')
  .query('q=nodejs&limit=10');

// Mixed usage
request.get('/api/search')
  .query('q=nodejs')
  .query({ limit: 10 });
// Results in: /api/search?q=nodejs&limit=10
```

## Executing Requests

### Promise-Based Execution

SuperAgent requests implement the Promise interface:

```javascript
// Using await
const res = await request.get('/api/users');
console.log(res.body);

// Using .then()
request.get('/api/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });

// Using .catch()
request.get('/api/users')
  .catch(err => {
    console.error('Request failed:', err.message);
  });

// Promise chaining
request.get('/api/users')
  .then(res => res.body)
  .then(users => users.filter(u => u.active))
  .then(activeUsers => {
    console.log('Active users:', activeUsers);
  })
  .catch(err => {
    console.error('Error:', err);
  });
```

### Callback-Based Execution

Use `.end()` for traditional callback-based execution:

```javascript { .api }
/**
 * Execute the request and invoke callback with (err, res).
 *
 * @param {Function} callback - Callback function (err, res) => void
 * @return {void}
 *
 * @example
 * request.get('/api/users')
 *   .end((err, res) => {
 *     if (err) {
 *       console.error(err);
 *       return;
 *     }
 *     console.log(res.body);
 *   });
 */
Request.prototype.end = function(callback)
```

**Usage examples:**

```javascript
// Basic callback
request.get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Error:', err.message);
      return;
    }
    console.log('Users:', res.body);
  });

// With configuration
request.post('/api/users')
  .send({ name: 'Alice' })
  .set('Authorization', 'Bearer token123')
  .end((err, res) => {
    if (err) throw err;
    console.log('Created:', res.body);
  });

// Error-first callback pattern
request.get('/api/users')
  .end(function(err, res) {
    if (err) {
      console.error('Status:', err.status);
      console.error('Message:', err.message);
      console.error('Response:', err.response);
      return;
    }

    console.log('Success:', res.body);
  });
```

### Implicit Execution with await

When using await, `.end()` is called automatically:

```javascript
// These are equivalent:
const res = await request.get('/api/users');

// Explicit .end() call (not necessary with await)
const res = await request.get('/api/users').end();
```

## Common Usage Examples

### Simple GET Request

```javascript
// With async/await
const res = await request.get('/api/users');
console.log(res.body);

// With callback
request.get('/api/users', (err, res) => {
  if (err) throw err;
  console.log(res.body);
});
```

### POST Request with JSON Body

```javascript
// With async/await
const res = await request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' })
  .set('Authorization', 'Bearer token123');

// With callback
request.post('/api/users', { name: 'Alice' }, (err, res) => {
  if (err) throw err;
  console.log('Created:', res.body);
});
```

### GET Request with Query Parameters

```javascript
// With async/await
const res = await request
  .get('/api/search')
  .query({ q: 'nodejs', limit: 10, offset: 0 });

// Inline data parameter
const res = await request.get('/api/search', { q: 'nodejs', limit: 10 });

// With callback
request.get('/api/search', { q: 'nodejs' }, (err, res) => {
  if (err) throw err;
  console.log(res.body);
});
```

### PUT Request for Updates

```javascript
// With async/await
const res = await request
  .put('/api/users/123')
  .send({ name: 'Alice Updated', email: 'alice.new@example.com' })
  .set('Authorization', 'Bearer token123');

// With callback
request.put('/api/users/123', { name: 'Updated' }, (err, res) => {
  if (err) throw err;
  console.log('Updated:', res.body);
});
```

### PATCH Request for Partial Updates

```javascript
// With async/await
const res = await request
  .patch('/api/users/123')
  .send({ email: 'newemail@example.com' });

console.log('Updated user:', res.body);
```

### DELETE Request

```javascript
// With async/await
await request
  .delete('/api/users/123')
  .set('Authorization', 'Bearer token123');

// Using del() alias
await request.del('/api/users/123');

// With callback
request.delete('/api/users/123', (err, res) => {
  if (err) throw err;
  console.log('Deleted successfully');
});
```

### HEAD Request to Check Resource

```javascript
// Check if resource exists
const res = await request.head('/api/users/123');
console.log('Status:', res.status);
console.log('Exists:', res.ok);

// With callback
request.head('/api/users/123', (err, res) => {
  if (err) throw err;
  console.log('Resource exists:', res.ok);
});
```

### OPTIONS Request

```javascript
// Check available methods
const res = await request.options('/api/users');
console.log('Allowed methods:', res.headers['allow']);
```

## Error Handling

Errors occur when requests fail due to network issues, timeouts, or unsuccessful HTTP status codes (4xx, 5xx):

```javascript
// Promise-based error handling
try {
  const res = await request.get('/api/users');
  console.log(res.body);
} catch (err) {
  console.error('Status:', err.status); // HTTP status code (e.g., 404, 500)
  console.error('Message:', err.message); // Error message
  console.error('Response:', err.response); // Response object if available
}

// Callback-based error handling
request.get('/api/users')
  .end((err, res) => {
    if (err) {
      // err.status contains HTTP status code
      // err.response contains the response object
      // err.message contains error message
      console.error(err);
      return;
    }
    console.log(res.body);
  });

// Catching specific errors
try {
  const res = await request.get('/api/users');
} catch (err) {
  if (err.status === 404) {
    console.log('User not found');
  } else if (err.status >= 500) {
    console.log('Server error');
  } else if (err.timeout) {
    console.log('Request timed out');
  } else {
    console.log('Request failed:', err.message);
  }
}
```

## Best Practices

### Use Async/Await for Modern Code

```javascript
// Preferred: Clean async/await syntax
async function fetchUsers() {
  const res = await request.get('/api/users');
  return res.body;
}

// Less preferred: Nested callbacks
function fetchUsers(callback) {
  request.get('/api/users')
    .end((err, res) => {
      if (err) return callback(err);
      callback(null, res.body);
    });
}
```

### Chain Configuration Methods

```javascript
// Good: Method chaining
const res = await request
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .type('json')
  .send({ name: 'Alice' })
  .timeout(5000)
  .retry(2);

// Less readable: Separate lines
const req = request.post('/api/users');
req.set('Authorization', 'Bearer token123');
req.type('json');
req.send({ name: 'Alice' });
req.timeout(5000);
req.retry(2);
const res = await req;
```

### Handle Errors Explicitly

```javascript
// Always handle errors
try {
  const res = await request.get('/api/users');
  console.log(res.body);
} catch (err) {
  // Handle error appropriately
  console.error('Failed to fetch users:', err.message);
  // Maybe retry, show user message, etc.
}
```

### Use Appropriate HTTP Methods

```javascript
// Good: Use semantic HTTP methods
await request.get('/api/users'); // Retrieve
await request.post('/api/users', { name: 'Alice' }); // Create
await request.put('/api/users/123', { name: 'Updated' }); // Full update
await request.patch('/api/users/123', { email: 'new@example.com' }); // Partial update
await request.delete('/api/users/123'); // Delete

// Bad: Using POST for everything
await request.post('/api/users/get'); // Should be GET
await request.post('/api/users/delete'); // Should be DELETE
```

## Request Object

All request creation methods return a `Request` object that can be configured and executed:

```javascript { .api }
/**
 * Request class represents an HTTP request.
 * Inherits from Stream (Node.js) or Emitter (Browser).
 *
 * @class Request
 */
class Request {
  /**
   * Constructor creates a new Request instance.
   * Generally not called directly; use request() or verb methods instead.
   *
   * @param {String} method - HTTP method
   * @param {String} url - Request URL
   */
  constructor(method, url)

  // Properties
  method: string;      // HTTP method (GET, POST, etc.)
  url: string;         // Request URL
  header: object;      // Request headers (preserves case)
  _header: object;     // Request headers (lowercase)

  // See other documentation pages for additional methods
}
```

The Request object is chainable and can be configured before execution. See [Request Configuration](./request-configuration.md) for details on configuration methods.
