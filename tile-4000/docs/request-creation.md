# Request Creation

SuperAgent provides a factory function and HTTP method helpers for creating requests.

## Capabilities

### Request Factory

Creates a new Request instance for any HTTP method.

```javascript { .api }
/**
 * Create a new HTTP request
 * @param method - HTTP method (GET, POST, etc.)
 * @param url - Request URL
 * @returns Request instance for chaining
 */
function request(method: string, url: string): Request;

/**
 * Create a GET request
 * @param url - Request URL
 * @returns Request instance for chaining
 */
function request(url: string): Request;

/**
 * Create a GET request with callback
 * @param url - Request URL
 * @param callback - Callback function (err, res) => {}
 * @returns Request instance
 */
function request(url: string, callback: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Custom method
const req = request('PATCH', 'https://api.example.com/users/123');

// GET shorthand
const req = request('https://api.example.com/users');

// GET with callback
request('https://api.example.com/users', (err, res) => {
  if (err) return console.error(err);
  console.log(res.body);
});
```

### GET Requests

Creates a GET request.

```javascript { .api }
/**
 * Create a GET request
 * @param url - Request URL
 * @param data - Optional query parameters as object
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.get(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// Basic GET
request.get('https://api.example.com/users')
  .then(res => console.log(res.body));

// GET with query params
request.get('https://api.example.com/users', { page: 1, limit: 10 })
  .then(res => console.log(res.body));

// GET with callback
request.get('https://api.example.com/users', (err, res) => {
  if (err) return console.error(err);
  console.log(res.body);
});
```

### POST Requests

Creates a POST request.

```javascript { .api }
/**
 * Create a POST request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.post(url: string, data?: object | string, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// POST with JSON body
request.post('https://api.example.com/users')
  .send({ name: 'John', email: 'john@example.com' })
  .then(res => console.log(res.body));

// POST with data parameter
request.post('https://api.example.com/users', { name: 'John' })
  .then(res => console.log(res.body));

// POST with callback
request.post('https://api.example.com/users', { name: 'John' }, (err, res) => {
  if (err) return console.error(err);
  console.log(res.body);
});
```

### PUT Requests

Creates a PUT request.

```javascript { .api }
/**
 * Create a PUT request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.put(url: string, data?: object | string, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// PUT to update resource
request.put('https://api.example.com/users/123')
  .send({ name: 'Jane Doe', email: 'jane@example.com' })
  .then(res => console.log(res.body));
```

### PATCH Requests

Creates a PATCH request.

```javascript { .api }
/**
 * Create a PATCH request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.patch(url: string, data?: object | string, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// PATCH to partially update resource
request.patch('https://api.example.com/users/123')
  .send({ email: 'newemail@example.com' })
  .then(res => console.log(res.body));
```

### DELETE Requests

Creates a DELETE request.

```javascript { .api }
/**
 * Create a DELETE request
 * @param url - Request URL
 * @param data - Optional request body data
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.delete(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;

/**
 * Alias for request.delete()
 */
function request.del(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// DELETE request
request.delete('https://api.example.com/users/123')
  .then(res => console.log('Deleted'));

// Using del() alias
request.del('https://api.example.com/users/123')
  .then(res => console.log('Deleted'));
```

### HEAD Requests

Creates a HEAD request (retrieves headers only, no body).

```javascript { .api }
/**
 * Create a HEAD request
 * @param url - Request URL
 * @param data - Optional query parameters
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.head(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// HEAD request to check if resource exists
request.head('https://api.example.com/users/123')
  .then(res => {
    console.log('Status:', res.status);
    console.log('Content-Type:', res.type);
  });
```

### OPTIONS Requests

Creates an OPTIONS request.

```javascript { .api }
/**
 * Create an OPTIONS request
 * @param url - Request URL
 * @param data - Optional request data
 * @param callback - Optional callback function
 * @returns Request instance for chaining
 */
function request.options(url: string, data?: object, callback?: (err: Error, res: Response) => void): Request;
```

**Usage Examples:**

```javascript
// OPTIONS request to check supported methods
request.options('https://api.example.com/users')
  .then(res => {
    console.log('Allowed methods:', res.header['allow']);
  });
```

### Additional HTTP Methods

SuperAgent supports all standard and extended HTTP methods through the `methods` npm package.

```javascript { .api }
/**
 * Additional HTTP methods available
 */
function request.connect(url: string, data?: object, callback?: Function): Request;
function request.trace(url: string, data?: object, callback?: Function): Request;
function request.search(url: string, data?: object, callback?: Function): Request;
function request.purge(url: string, data?: object, callback?: Function): Request;

// WebDAV methods
function request.copy(url: string, data?: object, callback?: Function): Request;
function request.lock(url: string, data?: object, callback?: Function): Request;
function request.unlock(url: string, data?: object, callback?: Function): Request;
function request.mkcol(url: string, data?: object, callback?: Function): Request;
function request.move(url: string, data?: object, callback?: Function): Request;
function request.propfind(url: string, data?: object, callback?: Function): Request;
function request.proppatch(url: string, data?: object, callback?: Function): Request;
function request.report(url: string, data?: object, callback?: Function): Request;
function request.mkactivity(url: string, data?: object, callback?: Function): Request;
function request.checkout(url: string, data?: object, callback?: Function): Request;
function request.merge(url: string, data?: object, callback?: Function): Request;

// Advanced methods
function request.subscribe(url: string, data?: object, callback?: Function): Request;
function request.unsubscribe(url: string, data?: object, callback?: Function): Request;
function request.notify(url: string, data?: object, callback?: Function): Request;
function request.msearch(url: string, data?: object, callback?: Function): Request;
function request.link(url: string, data?: object, callback?: Function): Request;
function request.unlink(url: string, data?: object, callback?: Function): Request;
function request.bind(url: string, data?: object, callback?: Function): Request;
function request.unbind(url: string, data?: object, callback?: Function): Request;
function request.rebind(url: string, data?: object, callback?: Function): Request;
function request.acl(url: string, data?: object, callback?: Function): Request;
function request.mkcalendar(url: string, data?: object, callback?: Function): Request;
function request.source(url: string, data?: object, callback?: Function): Request;
```

**Usage Examples:**

```javascript
// TRACE method for debugging
request.trace('https://api.example.com/debug')
  .then(res => console.log('Trace:', res.text));

// PURGE method for cache invalidation
request.purge('https://cdn.example.com/cached-resource')
  .then(res => console.log('Cache purged'));

// WebDAV PROPFIND
request.propfind('https://webdav.example.com/folder')
  .then(res => console.log('Properties:', res.body));

// WebDAV MKCOL (create collection/folder)
request.mkcol('https://webdav.example.com/newfolder')
  .then(res => console.log('Folder created'));
```

## Request Class

All HTTP method helpers and the factory function return instances of the Request class.

```javascript { .api }
/**
 * Request class constructor
 * Note: Typically not called directly - use request() factory or HTTP method helpers
 * @param method - HTTP method
 * @param url - Request URL
 */
class Request {
  constructor(method: string, url: string);
}
```

The Request class is exported for advanced use cases:

```javascript
const request = require('superagent');
const Request = request.Request;

const req = new Request('GET', 'https://api.example.com/users');
```

### Request Properties

Request instances have the following properties:

```javascript { .api }
interface Request {
  /** HTTP method (e.g., 'GET', 'POST') */
  method: string;

  /** Request URL */
  url: string;

  /** Request headers object */
  header: object;

  /** Cookies string (Node.js only) */
  cookies: string;
}
```

**Usage Examples:**

```javascript
const req = request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer token123');

console.log('Method:', req.method);   // 'GET'
console.log('URL:', req.url);         // 'https://api.example.com/users'
console.log('Headers:', req.header);  // { authorization: 'Bearer token123', ... }
```
