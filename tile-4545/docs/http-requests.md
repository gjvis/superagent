# HTTP Requests

Core functionality for creating and sending HTTP requests using various HTTP methods (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS).

## Capabilities

### Main Request Function

Creates a new HTTP request with the specified method and URL.

```javascript { .api }
/**
 * Create a new HTTP request
 * @param method - HTTP method (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS)
 * @param url - Request URL (absolute or relative)
 * @returns Request instance for chaining
 */
function request(method: string, url: string): Request;

/**
 * Create a GET request (shorthand)
 * @param url - Request URL
 * @returns Request instance for chaining
 */
function request(url: string): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Full syntax
const req1 = request('GET', '/api/users');

// Shorthand for GET
const req2 = request('/api/users');

// Must call .end() or use promises to send
request('GET', '/api/users').end((err, res) => {
  console.log(res.body);
});
```

### GET Request

Perform HTTP GET request.

```javascript { .api }
/**
 * Perform a GET request
 * @param url - Request URL
 * @param data - Optional query parameters (converted to query string)
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.get(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Simple GET
request
  .get('/api/users')
  .end((err, res) => {
    console.log(res.body);
  });

// GET with query parameters
request
  .get('/api/users', { role: 'admin', active: true })
  .end((err, res) => {
    console.log(res.body);
  });

// GET with callback shorthand
request.get('/api/users', (err, res) => {
  if (err) console.error(err);
  else console.log(res.body);
});

// GET with promises
request
  .get('/api/users')
  .then(res => console.log(res.body))
  .catch(err => console.error(err));

// GET with async/await
async function getUsers() {
  const res = await request.get('/api/users');
  return res.body;
}
```

### POST Request

Perform HTTP POST request.

```javascript { .api }
/**
 * Perform a POST request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.post(url: string, data?: any, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// POST with JSON body
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' })
  .end((err, res) => {
    console.log(res.body);
  });

// POST with data parameter
request
  .post('/api/users', { name: 'John' })
  .end((err, res) => {
    console.log(res.body);
  });

// POST with form data
request
  .post('/api/login')
  .type('form')
  .send({ username: 'admin', password: 'secret' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### PUT Request

Perform HTTP PUT request for updating resources.

```javascript { .api }
/**
 * Perform a PUT request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.put(url: string, data?: any, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Update user with PUT
request
  .put('/api/users/123')
  .send({ name: 'John Updated', email: 'john.new@example.com' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### PATCH Request

Perform HTTP PATCH request for partial updates.

```javascript { .api }
/**
 * Perform a PATCH request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.patch(url: string, data?: any, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Partial update with PATCH
request
  .patch('/api/users/123')
  .send({ email: 'newemail@example.com' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### DELETE Request

Perform HTTP DELETE request.

```javascript { .api }
/**
 * Perform a DELETE request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.delete(url: string, data?: any, callback?: (err: Error, res: Response) => void): Request;

/**
 * Alias for delete() method
 */
request.del(url: string, data?: any, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Delete resource
request
  .delete('/api/users/123')
  .end((err, res) => {
    console.log('User deleted');
  });

// Using del alias
request
  .del('/api/users/123')
  .end((err, res) => {
    console.log('User deleted');
  });
```

### HEAD Request

Perform HTTP HEAD request (retrieves headers only, no body).

```javascript { .api }
/**
 * Perform a HEAD request
 * @param url - Request URL
 * @param data - Optional query parameters
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.head(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Check if resource exists
request
  .head('/api/users/123')
  .end((err, res) => {
    if (res.ok) {
      console.log('User exists');
      console.log('Last-Modified:', res.header['last-modified']);
    }
  });
```

### OPTIONS Request

Perform HTTP OPTIONS request (retrieves allowed methods and CORS headers).

```javascript { .api }
/**
 * Perform an OPTIONS request
 * @param url - Request URL
 * @param data - Optional query parameters
 * @param callback - Optional callback function(err, res)
 * @returns Request instance for chaining
 */
request.options(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Check allowed methods
request
  .options('/api/users')
  .end((err, res) => {
    console.log('Allowed methods:', res.header['allow']);
    console.log('CORS headers:', res.header['access-control-allow-methods']);
  });
```

## Request Lifecycle

All request methods return a `Request` instance that can be configured before sending:

```javascript
// Build request step by step
const req = request
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('Content-Type', 'application/json')
  .send({ name: 'John' })
  .timeout(5000)
  .retry(2);

// Send with callback
req.end((err, res) => {
  console.log(res.body);
});

// Or send with promises
req.then(res => {
  console.log(res.body);
}).catch(err => {
  console.error(err);
});
```

The request is not sent until `.end()`, `.then()`, `.catch()`, or `await` is called.
