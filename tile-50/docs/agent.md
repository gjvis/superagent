# Agent and Cookies

Persistent agent for maintaining cookies across requests and setting default request configurations.

## Capabilities

### Agent Creation

Create an agent to persist cookies and default settings across multiple requests.

```javascript { .api }
/**
 * Create agent with persistent cookies (Node.js only)
 * Browser agent does not persist cookies (browser handles cookies automatically)
 * @param options - Optional configuration {ca, key, pfx, cert}
 * @returns Agent instance
 */
request.agent(options?: {
  ca?: string | Buffer,
  key?: string | Buffer,
  pfx?: string | Buffer | {pfx: Buffer, passphrase: string},
  cert?: string | Buffer
}): Agent;

interface Agent {
  // Cookie jar (Node.js only)
  jar: CookieJar;

  // HTTP verb methods with automatic cookie handling
  get(url: string, callback?: Function): Request;
  post(url: string, callback?: Function): Request;
  put(url: string, callback?: Function): Request;
  patch(url: string, callback?: Function): Request;
  delete(url: string, callback?: Function): Request;
  del(url: string, callback?: Function): Request;
  head(url: string, callback?: Function): Request;
  options(url: string, callback?: Function): Request;

  // Default configuration methods (all Request methods available)
  use(plugin: Function): Agent;
  on(event: string, listener: Function): Agent;
  once(event: string, listener: Function): Agent;
  set(field: string | object, value?: string): Agent;
  query(params: string | object): Agent;
  type(type: string): Agent;
  accept(type: string): Agent;
  auth(user: string, pass?: string, options?: object): Agent;
  withCredentials(on?: boolean): Agent;
  sortQuery(fn?: Function): Agent;
  retry(count: number, fn?: Function): Agent;
  ok(callback: Function): Agent;
  redirects(count: number): Agent;
  timeout(ms: number | object): Agent;
  buffer(enable?: boolean): Agent;
  serialize(fn: Function): Agent;
  parse(fn: Function): Agent;
  ca(cert: string | Buffer): Agent;
  key(cert: string | Buffer): Agent;
  pfx(cert: string | Buffer | object): Agent;
  cert(cert: string | Buffer): Agent;
}
```

### Basic Agent Usage

```javascript
const agent = request.agent();

// All requests from this agent share cookies
await agent
  .post('/login')
  .send({ username: 'user', password: 'pass' });

// Session cookie automatically included
const profile = await agent.get('/profile');
const settings = await agent.get('/settings');

// Logout
await agent.post('/logout');
```

### Agent with TLS Configuration

```javascript
const fs = require('fs');

const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem')
});

// All requests use these certificates
const res = await agent.get('https://secure-api.example.com/data');
```

## Cookie Persistence (Node.js)

The agent automatically manages cookies across requests.

### Login Flow with Cookies

```javascript
const agent = request.agent();

// Login - receives Set-Cookie header
const loginRes = await agent
  .post('https://api.example.com/login')
  .send({ email: 'user@example.com', password: 'secret' });

console.log('Logged in:', loginRes.body);

// Subsequent requests automatically include cookies
const userData = await agent
  .get('https://api.example.com/user/profile');

const orders = await agent
  .get('https://api.example.com/user/orders');

// Cookies persist across all requests
const cart = await agent
  .post('https://api.example.com/cart/add')
  .send({ productId: '123', quantity: 2 });
```

### Cookie Jar Access

```javascript
const agent = request.agent();

await agent
  .post('/login')
  .send({ username: 'user', password: 'pass' });

// Access cookie jar (Node.js only)
console.log(agent.jar);

// Cookies are automatically:
// - Saved from Set-Cookie headers
// - Sent with matching domain/path
// - Updated on subsequent Set-Cookie headers
// - Removed when expired
```

### Multi-Step Authentication

```javascript
const agent = request.agent();

// Step 1: Get CSRF token
const csrfRes = await agent.get('/api/csrf-token');
const csrfToken = csrfRes.body.token;

// Step 2: Login with CSRF token
await agent
  .post('/api/login')
  .set('X-CSRF-Token', csrfToken)
  .send({ username: 'user', password: 'pass' });

// Step 3: Access protected resources
const data = await agent.get('/api/protected-data');
```

## Default Configuration

Set default configuration that applies to all requests from the agent.

### Default Headers

```javascript
const agent = request.agent()
  .set('Accept', 'application/json')
  .set('X-API-Key', 'your-api-key');

// All requests include these headers
await agent.get('/api/users');
await agent.get('/api/posts');
await agent.post('/api/comments').send({ text: 'Nice!' });
```

### Default Authentication

```javascript
const agent = request.agent()
  .auth('api-token', { type: 'bearer' });

// All requests include Authorization header
await agent.get('/api/data');
await agent.post('/api/resource').send({ name: 'Test' });
```

### Default Timeout

```javascript
const agent = request.agent()
  .timeout({ response: 5000, deadline: 10000 });

// All requests use these timeouts
await agent.get('/api/slow-endpoint');
```

### Default Retry

```javascript
const agent = request.agent()
  .retry(3);

// All requests retry up to 3 times on failure
await agent.get('/api/unstable-endpoint');
```

### Multiple Defaults

```javascript
const agent = request.agent()
  .set('Accept', 'application/json')
  .set('X-API-Version', '2')
  .auth('api-key-123', { type: 'bearer' })
  .timeout(10000)
  .retry(2)
  .redirects(5);

// All configuration applied to every request
const users = await agent.get('/api/users');
const posts = await agent.get('/api/posts');
```

## Agent Patterns

### API Client with Agent

```javascript
class APIClient {
  constructor(baseUrl, apiKey) {
    this.baseUrl = baseUrl;
    this.agent = request.agent()
      .set('Authorization', `Bearer ${apiKey}`)
      .set('Accept', 'application/json')
      .timeout(10000)
      .retry(2);
  }

  async get(path) {
    const res = await this.agent.get(`${this.baseUrl}${path}`);
    return res.body;
  }

  async post(path, data) {
    const res = await this.agent
      .post(`${this.baseUrl}${path}`)
      .send(data);
    return res.body;
  }

  async put(path, data) {
    const res = await this.agent
      .put(`${this.baseUrl}${path}`)
      .send(data);
    return res.body;
  }

  async delete(path) {
    const res = await this.agent.delete(`${this.baseUrl}${path}`);
    return res.body;
  }
}

// Usage
const api = new APIClient('https://api.example.com', 'your-api-key');

const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'John' });
await api.put('/users/123', { name: 'Jane' });
await api.delete('/users/123');
```

### Authenticated Session

```javascript
class Session {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.agent = request.agent();
    this.authenticated = false;
  }

  async login(email, password) {
    const res = await this.agent
      .post(`${this.baseUrl}/login`)
      .send({ email, password });

    this.authenticated = true;
    return res.body;
  }

  async request(method, path, data) {
    if (!this.authenticated) {
      throw new Error('Not authenticated');
    }

    const req = this.agent[method.toLowerCase()](`${this.baseUrl}${path}`);

    if (data) {
      req.send(data);
    }

    return (await req).body;
  }

  async logout() {
    await this.agent.post(`${this.baseUrl}/logout`);
    this.authenticated = false;
  }
}

// Usage
const session = new Session('https://api.example.com');

await session.login('user@example.com', 'password');
const profile = await session.request('GET', '/profile');
const updated = await session.request('PUT', '/profile', { name: 'New Name' });
await session.logout();
```

### Rate-Limited API Client

```javascript
class RateLimitedClient {
  constructor(baseUrl, apiKey, requestsPerSecond) {
    this.agent = request.agent()
      .set('Authorization', `Bearer ${apiKey}`)
      .timeout(30000)
      .retry(3, this.retryStrategy.bind(this));

    this.queue = [];
    this.interval = 1000 / requestsPerSecond;
    this.processing = false;
  }

  retryStrategy(err, res) {
    // Retry on rate limit
    if (res && res.status === 429) {
      const retryAfter = res.get('Retry-After');
      if (retryAfter) {
        console.log(`Rate limited, retry after ${retryAfter}s`);
      }
      return true;
    }
  }

  async request(method, url, data) {
    return new Promise((resolve, reject) => {
      this.queue.push({ method, url, data, resolve, reject });
      if (!this.processing) {
        this.processQueue();
      }
    });
  }

  async processQueue() {
    this.processing = true;

    while (this.queue.length > 0) {
      const { method, url, data, resolve, reject } = this.queue.shift();

      try {
        const req = this.agent[method](url);
        if (data) req.send(data);
        const res = await req;
        resolve(res.body);
      } catch (err) {
        reject(err);
      }

      if (this.queue.length > 0) {
        await new Promise(r => setTimeout(r, this.interval));
      }
    }

    this.processing = false;
  }
}

// Usage: max 10 requests per second
const client = new RateLimitedClient('https://api.example.com', 'key', 10);

// Queue multiple requests
const promises = [];
for (let i = 0; i < 50; i++) {
  promises.push(client.request('get', `/items/${i}`));
}

// Automatically rate-limited
const results = await Promise.all(promises);
```

### Parallel Requests with Shared Agent

```javascript
const agent = request.agent();

// Login once
await agent
  .post('/login')
  .send({ username: 'user', password: 'pass' });

// Make parallel requests with shared cookies
const [users, posts, comments] = await Promise.all([
  agent.get('/api/users'),
  agent.get('/api/posts'),
  agent.get('/api/comments')
]);

console.log('Users:', users.body);
console.log('Posts:', posts.body);
console.log('Comments:', comments.body);
```

## Browser vs Node.js

### Node.js Agent
- Persists cookies automatically via CookieJar
- Supports TLS configuration (ca, key, pfx, cert)
- Requires manual agent creation
- Full control over cookie storage

```javascript
// Node.js
const agent = request.agent();

await agent.post('/login').send({ user: 'name', pass: 'secret' });
// Cookies saved in agent.jar

const data = await agent.get('/data');
// Cookies automatically sent
```

### Browser Agent
- Browser handles cookies automatically
- Agent provides default configuration only
- No cookie jar (browser manages cookies)
- Subject to browser cookie policies

```javascript
// Browser
const agent = request.agent();

// Agent sets defaults but browser manages cookies
agent.set('Accept', 'application/json');

await agent.post('/login').send({ user: 'name', pass: 'secret' });
// Browser saves cookies

const data = await agent.get('/data');
// Browser automatically sends cookies
```

## Default Configuration Reference

All Request methods can be used to set defaults:

```javascript
const agent = request.agent()
  // Headers
  .set('Header', 'value')

  // Authentication
  .auth('user', 'pass')

  // Query parameters
  .query({ key: 'value' })

  // Content type
  .type('json')
  .accept('json')

  // Timeouts
  .timeout(10000)

  // Retries
  .retry(3)

  // Redirects (Node.js)
  .redirects(5)

  // Buffering (Node.js)
  .buffer(true)

  // Parsers and serializers
  .parse(customParser)
  .serialize(customSerializer)

  // TLS (Node.js)
  .ca(cert)
  .key(key)
  .cert(cert)
  .pfx(pfx)

  // Plugins
  .use(plugin)

  // Events
  .on('response', handler)

  // Custom OK check
  .ok(res => res.status < 500)

  // CORS (browser)
  .withCredentials()

  // Query sorting
  .sortQuery();
```

All of these configuration methods apply to every request made through the agent.
