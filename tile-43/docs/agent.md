# Agent

The Agent class provides persistent configuration and automatic cookie management across multiple requests. It's useful for maintaining sessions and applying common settings to groups of requests.

## Creating an Agent

```javascript { .api }
/**
 * Create a new Agent instance
 * @param options - Optional SSL/TLS configuration
 * @param options.ca - Certificate authority
 * @param options.key - Client private key
 * @param options.pfx - PFX/PKCS12 certificate
 * @param options.cert - Client certificate
 * @returns Agent instance
 */
function agent(options?: {
  ca?: Buffer | string,
  key?: Buffer | string,
  pfx?: Buffer | string,
  cert?: Buffer | string
}): Agent;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Create agent with no options
const agent = request.agent();

// Create agent with SSL certificates (Node.js)
const fs = require('fs');
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem')
});
```

## Agent HTTP Verb Methods

The Agent provides the same HTTP verb methods as the main request object, but with automatic cookie handling and default configuration.

```javascript { .api }
interface Agent {
  /**
   * Create GET request with agent defaults and cookies
   */
  get(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create POST request with agent defaults and cookies
   */
  post(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create PUT request with agent defaults and cookies
   */
  put(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create PATCH request with agent defaults and cookies
   */
  patch(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create DELETE request with agent defaults and cookies
   */
  delete(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create DELETE request (alias)
   */
  del(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create HEAD request with agent defaults and cookies
   */
  head(url: string, callback?: (err: Error, res: Response) => void): Request;

  /**
   * Create OPTIONS request with agent defaults and cookies
   */
  options(url: string, callback?: (err: Error, res: Response) => void): Request;
}
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Make requests using agent
agent
  .get('/api/users')
  .then(res => console.log('Users:', res.body));

agent
  .post('/api/users')
  .send({ name: 'Alice' })
  .then(res => console.log('Created:', res.body));

// With callback
agent.get('/api/users', (err, res) => {
  if (err) {
    console.error(err);
  } else {
    console.log('Users:', res.body);
  }
});
```

## Default Configuration Methods

Set default configuration that applies to all requests made by this agent. All Request configuration methods are available on Agent.

```javascript { .api }
interface Agent {
  // Headers
  set(field: string, value: string): Agent;
  set(headers: object): Agent;

  // Content types
  type(contentType: string): Agent;
  accept(acceptType: string): Agent;

  // Query parameters
  query(params: object | string): Agent;

  // Authentication
  auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Agent;

  // Request behavior
  timeout(ms: number): Agent;
  timeout(options: { response?: number, deadline?: number }): Agent;
  retry(count?: number, callback?: (err: Error, res: Response) => boolean): Agent;
  redirects(count: number): Agent;
  buffer(enable?: boolean): Agent;
  sortQuery(compareFn?: (a: string, b: string) => number): Agent;

  // Response validation
  ok(validator: (res: Response) => boolean): Agent;

  // Parsing and serialization
  parse(parser: (res: any, callback: Function) => void): Agent;
  serialize(serializer: (data: any) => string): Agent;

  // SSL/TLS (Node.js only)
  ca(cert: Buffer | string): Agent;
  key(cert: Buffer | string): Agent;
  pfx(cert: Buffer | string | { pfx: Buffer | string, passphrase: string }): Agent;
  cert(cert: Buffer | string): Agent;

  // CORS (Browser)
  withCredentials(enable?: boolean): Agent;

  // Plugins
  use(plugin: (req: Request) => void): Agent;

  // Events
  on(event: string, handler: Function): Agent;
  once(event: string, handler: Function): Agent;
}
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Set default headers for all requests
agent
  .set('Authorization', 'Bearer token123')
  .set('X-API-Version', 'v2');

// Set default query parameters
agent.query({ api_key: 'secret123' });

// Set default timeout and retries
agent
  .timeout(5000)
  .retry(2);

// Set default accept header
agent.accept('json');

// Now all requests use these defaults
agent.get('/api/users').then(res => {
  // Request includes:
  // - Authorization: Bearer token123
  // - X-API-Version: v2
  // - Query param: api_key=secret123
  // - Accept: application/json
  // - 5 second timeout
  // - 2 retries on failure
  console.log('Users:', res.body);
});

// Individual requests can still override defaults
agent
  .get('/api/admin')
  .set('Authorization', 'Bearer admin-token')  // Override default
  .timeout(10000);  // Override default timeout
```

## Cookie Persistence (Node.js only)

In Node.js, Agent automatically manages cookies across requests using a built-in cookie jar.

```javascript { .api }
/**
 * Cookie jar instance for automatic cookie storage and sending (Node.js only)
 */
jar: CookieJar;
```

**Automatic cookie handling:**
1. **Set-Cookie headers** from responses are automatically stored in the jar
2. **Cookies are automatically sent** with subsequent requests to the same domain
3. **Redirects preserve cookies** across redirect chains

**Usage Example:**

```javascript
const agent = request.agent();

// First request - server sets cookies
agent
  .post('/api/login')
  .send({ username: 'alice', password: 'secret' })
  .then(res => {
    // Server responded with Set-Cookie header
    // Cookies automatically stored in agent.jar
    console.log('Logged in');
  });

// Subsequent requests - cookies automatically sent
agent
  .get('/api/profile')
  .then(res => {
    // Request includes cookies from login
    // No need to manually set Cookie header
    console.log('Profile:', res.body);
  });

agent
  .get('/api/dashboard')
  .then(res => {
    // Cookies still included
    console.log('Dashboard:', res.body);
  });

// Access cookie jar directly (advanced usage)
console.log('Cookies:', agent.jar);
```

## Complete Agent Example

```javascript
const request = require('superagent');

// Create agent with default configuration
const api = request.agent()
  .set('Accept', 'application/json')
  .set('X-API-Version', 'v2')
  .timeout({ response: 5000, deadline: 10000 })
  .retry(2)
  .ok(res => res.status < 500);  // Accept 4xx errors

// Login - sets session cookie
api
  .post('/api/login')
  .send({ username: 'alice', password: 'secret123' })
  .then(res => {
    console.log('Login successful');
    // Cookie automatically stored in agent
  })
  .catch(err => {
    console.error('Login failed:', err.message);
  });

// Fetch user profile - cookie automatically sent
api
  .get('/api/users/me')
  .then(res => {
    console.log('Profile:', res.body);
  });

// Update profile - cookie automatically sent
api
  .patch('/api/users/me')
  .send({ bio: 'Updated bio' })
  .then(res => {
    console.log('Profile updated:', res.body);
  });

// Fetch dashboard data - cookie automatically sent
api
  .get('/api/dashboard')
  .query({ period: '30d' })
  .then(res => {
    console.log('Dashboard data:', res.body);
  });

// Override agent defaults for specific request
api
  .get('/api/large-report')
  .timeout({ deadline: 60000 })  // Override timeout for this request
  .buffer(false)  // Disable buffering
  .then(res => {
    console.log('Report ready');
  });

// Logout - cookie automatically sent
api
  .post('/api/logout')
  .then(res => {
    console.log('Logged out');
  });
```

## Agent vs Regular Requests

**Use Agent when:**
- Making multiple requests to the same API
- Need to maintain session cookies (Node.js)
- Want to share common configuration (headers, timeout, etc.)
- Building API clients or wrappers

**Use regular requests when:**
- Making single, independent requests
- Each request needs completely different configuration
- No need for cookie persistence

**Comparison:**

```javascript
// Without agent (manual cookie handling)
let sessionCookie;

request
  .post('/api/login')
  .send({ username: 'alice', password: 'secret' })
  .then(res => {
    sessionCookie = res.header['set-cookie'];

    return request
      .get('/api/profile')
      .set('Cookie', sessionCookie);  // Manual cookie handling
  })
  .then(res => {
    console.log('Profile:', res.body);
  });

// With agent (automatic cookie handling)
const agent = request.agent();

agent
  .post('/api/login')
  .send({ username: 'alice', password: 'secret' })
  .then(res => {
    // Cookie automatically stored
    return agent.get('/api/profile');  // Cookie automatically sent
  })
  .then(res => {
    console.log('Profile:', res.body);
  });
```

## Browser Considerations

In browsers, the Agent class is available but with limited functionality:
- **Cookie persistence**: Handled automatically by the browser (not via `.jar`)
- **Default configuration**: Works the same as Node.js
- **SSL/TLS methods** (`.ca()`, `.key()`, etc.): Not applicable

```javascript
// Browser agent usage
const agent = superagent.agent()
  .set('X-API-Version', 'v2')
  .accept('json');

// Cookies handled by browser automatically
agent.get('/api/users').then(res => {
  console.log('Users:', res.body);
});
```
