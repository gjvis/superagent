# Agent

Create agents that maintain state (cookies, default headers) across multiple requests.

## Capabilities

### Creating an Agent

Create an agent instance to persist state across requests.

```javascript { .api }
/**
 * Create a new agent
 * @param options - Optional agent configuration
 * @returns Agent instance
 */
request.agent(options?: object): Agent;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Create agent
const agent = request.agent();

// Use agent for multiple requests
agent
  .get('/api/login')
  .auth('user', 'pass')
  .end((err, res) => {
    // Cookies from this response are saved
    console.log('Logged in');
  });

// Subsequent requests use saved cookies
agent
  .get('/api/profile')
  .end((err, res) => {
    // Automatically includes cookies from login
    console.log(res.body);
  });
```

### Agent HTTP Methods

Agents support all standard HTTP methods.

```javascript { .api }
interface Agent {
  get(url: string, callback?: (err: Error, res: Response) => void): Request;
  post(url: string, callback?: (err: Error, res: Response) => void): Request;
  put(url: string, callback?: (err: Error, res: Response) => void): Request;
  patch(url: string, callback?: (err: Error, res: Response) => void): Request;
  delete(url: string, callback?: (err: Error, res: Response) => void): Request;
  del(url: string, callback?: (err: Error, res: Response) => void): Request;
  head(url: string, callback?: (err: Error, res: Response) => void): Request;
  options(url: string, callback?: (err: Error, res: Response) => void): Request;
}
```

**Usage Examples:**

```javascript
const agent = request.agent();

// GET request
agent.get('/api/users').end((err, res) => {
  console.log(res.body);
});

// POST request
agent.post('/api/users')
  .send({ name: 'John' })
  .end((err, res) => {
    console.log(res.body);
  });

// All HTTP methods work the same
agent.put('/api/users/1').send({ name: 'Jane' });
agent.delete('/api/users/1');
agent.patch('/api/users/1').send({ email: 'new@email.com' });
```

### Cookie Persistence (Node.js)

Agents automatically maintain cookies across requests.

```javascript { .api }
interface Agent {
  // Node.js only
  jar: CookieJar; // Cookie storage
}
```

**Usage Examples (Node.js):**

```javascript
const agent = request.agent();

// Login request sets cookies
agent
  .post('/api/login')
  .send({ username: 'admin', password: 'secret' })
  .end((err, res) => {
    // Server sets session cookie in response
    console.log('Logged in');
  });

// Subsequent requests automatically include cookies
agent
  .get('/api/protected')
  .end((err, res) => {
    // Session cookie is sent automatically
    console.log('Accessed protected resource');
  });

// Access cookie jar
console.log(agent.jar); // CookieJar instance

// Logout
agent
  .post('/api/logout')
  .end((err, res) => {
    // Session cookie is cleared by server
  });
```

### Agent Default Configuration

Set default configuration that applies to all requests from the agent.

```javascript { .api }
interface Agent {
  // Set default headers
  set(field: string, value: string): Agent;
  set(headers: object): Agent;

  // Set default authentication
  auth(user: string, pass: string, options?: object): Agent;

  // Set default content type
  type(type: string): Agent;

  // Set default accept header
  accept(type: string): Agent;

  // Set default timeout
  timeout(ms: number | {response?: number, deadline?: number}): Agent;

  // Set default retry
  retry(count: number, callback?: Function): Agent;

  // Set default redirects
  redirects(count: number): Agent;

  // Set default success check
  ok(callback: (res: Response) => boolean): Agent;

  // Apply plugin to all requests
  use(plugin: Function): Agent;

  // Set default serializer
  serialize(fn: (obj: any) => string): Agent;

  // Set default parser
  parse(fn: (res: Response, callback: Function) => void): Agent;

  // Set default query sorting
  sortQuery(fn?: (a: string, b: string) => number): Agent;

  // Set default credentials (Browser)
  withCredentials(enable?: boolean): Agent;

  // Node.js specific defaults
  buffer(enable: boolean): Agent;
  ca(cert: string | Buffer | Array): Agent;
  cert(cert: string | Buffer): Agent;
  key(key: string | Buffer): Agent;
  pfx(pfx: string | Buffer): Agent;

  // EventEmitter methods
  on(event: string, callback: Function): Agent;
  once(event: string, callback: Function): Agent;
}
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Set default headers for all requests
agent.set('Authorization', 'Bearer token123');
agent.set('User-Agent', 'MyApp/1.0');

// Now all requests from this agent include these headers
agent.get('/api/users').end((err, res) => {
  // Includes Authorization and User-Agent headers
});

agent.post('/api/data').send({...}).end((err, res) => {
  // Also includes the default headers
});

// Set multiple defaults
agent
  .set('X-API-Key', 'abc123')
  .timeout(5000)
  .retry(2)
  .accept('json');

// All requests from this agent use these defaults
agent.get('/api/users'); // 5s timeout, 2 retries, accepts JSON
agent.post('/api/data'); // same defaults

// Override defaults for specific request
agent
  .get('/api/slow')
  .timeout(30000); // Override the 5s default timeout
```

### Full Agent Example

Complete example showing agent usage with authentication.

**Usage Example:**

```javascript
const request = require('superagent');

// Create agent with defaults
const agent = request.agent()
  .set('User-Agent', 'MyApp/1.0')
  .timeout(10000)
  .retry(2);

// Login
async function login(username, password) {
  try {
    const res = await agent
      .post('/api/login')
      .send({ username, password });

    // Set auth token as default header
    agent.set('Authorization', `Bearer ${res.body.token}`);

    return res.body;
  } catch (err) {
    console.error('Login failed:', err.message);
    throw err;
  }
}

// Use authenticated agent
async function getProfile() {
  const res = await agent.get('/api/profile');
  return res.body;
}

async function updateProfile(data) {
  const res = await agent
    .put('/api/profile')
    .send(data);
  return res.body;
}

// Main flow
(async () => {
  await login('admin', 'password');

  const profile = await getProfile();
  console.log('Profile:', profile);

  await updateProfile({ email: 'new@email.com' });
  console.log('Profile updated');
})();
```

### Agent vs Regular Requests

**Agent**: Use when you need to maintain state across multiple requests
- Persistent cookies (Node.js)
- Shared default headers
- Shared configuration (timeout, retry, etc.)

**Regular requests**: Use for one-off requests or when state shouldn't be shared
- No cookie persistence
- Each request is independent
- No shared configuration

```javascript
// Regular requests (independent)
request.get('/api/login').auth('user', 'pass').end(...);
request.get('/api/data').end(...); // No auth, no cookies from login

// Agent (stateful)
const agent = request.agent();
agent.get('/api/login').auth('user', 'pass').end(...);
agent.get('/api/data').end(...); // Includes cookies from login
```
