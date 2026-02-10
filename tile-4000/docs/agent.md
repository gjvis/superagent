# Agent

Manage request defaults and cookie persistence across multiple requests.

## Capabilities

### Agent Creation

Create an agent to manage request defaults and state.

```javascript { .api }
/**
 * Create a new agent
 * @param options - Optional SSL certificate options (Node.js only)
 * @returns Agent instance
 */
function request.agent(options?: object): Agent;
```

**Options object (Node.js):**

```javascript { .api }
interface AgentOptions {
  ca?: string | Buffer | Array;    // CA certificate(s)
  key?: string | Buffer;            // Private key
  pfx?: string | Buffer | object;   // PFX certificate
  cert?: string | Buffer;           // Client certificate
}
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Basic agent
const agent = request.agent();

// Agent with SSL certificates (Node.js)
const agent = request.agent({
  ca: fs.readFileSync('/path/to/ca.pem'),
  key: fs.readFileSync('/path/to/key.pem'),
  cert: fs.readFileSync('/path/to/cert.pem')
});
```

### Cookie Persistence (Node.js)

Agents automatically persist cookies across requests.

```javascript { .api }
interface Agent {
  /** Cookie jar instance (Node.js only) */
  jar: CookieJar;
}
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Login (sets session cookie)
await agent.post('https://api.example.com/login')
  .send({ username: 'john', password: 'secret' });

// Subsequent requests automatically include cookies
const profileRes = await agent.get('https://api.example.com/profile');
console.log('Profile:', profileRes.body);

const settingsRes = await agent.get('https://api.example.com/settings');
console.log('Settings:', settingsRes.body);

// Logout
await agent.post('https://api.example.com/logout');

// Access cookie jar directly
console.log('Cookies:', agent.jar.getCookies());
```

### Setting Request Defaults

Configure default settings for all requests made through the agent.

```javascript { .api }
/**
 * Set default header
 */
Agent.prototype.set(field: string, value: string): Agent;

/**
 * Set default query parameters
 */
Agent.prototype.query(params: object): Agent;

/**
 * Set default Content-Type
 */
Agent.prototype.type(contentType: string): Agent;

/**
 * Set default Accept header
 */
Agent.prototype.accept(acceptType: string): Agent;

/**
 * Set default authentication
 */
Agent.prototype.auth(user: string, pass: string, options?: object): Agent;

/**
 * Set default timeout
 */
Agent.prototype.timeout(ms: number | object): Agent;

/**
 * Set default retry behavior
 */
Agent.prototype.retry(count?: number, callback?: Function): Agent;

/**
 * Set default response validator
 */
Agent.prototype.ok(callback: (res: Response) => boolean): Agent;

/**
 * Set default max redirects
 */
Agent.prototype.redirects(count: number): Agent;

/**
 * Set default CORS credentials
 */
Agent.prototype.withCredentials(enable?: boolean): Agent;

/**
 * Set default query sorting
 */
Agent.prototype.sortQuery(compareFn?: Function): Agent;

/**
 * Set default buffering (Node.js)
 */
Agent.prototype.buffer(enable?: boolean): Agent;

/**
 * Set default serializer
 */
Agent.prototype.serialize(fn: Function): Agent;

/**
 * Set default parser
 */
Agent.prototype.parse(fn: Function): Agent;
```

**SSL/TLS Methods (Node.js):**

```javascript { .api }
/**
 * Set default CA certificate
 */
Agent.prototype.ca(cert: string | Buffer | Array): Agent;

/**
 * Set default private key
 */
Agent.prototype.key(key: string | Buffer): Agent;

/**
 * Set default PFX certificate
 */
Agent.prototype.pfx(pfx: string | Buffer | object): Agent;

/**
 * Set default client certificate
 */
Agent.prototype.cert(cert: string | Buffer): Agent;
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Set default headers for all requests
agent
  .set('Authorization', 'Bearer token123')
  .set('X-API-Version', 'v2')
  .accept('json');

// All requests from this agent include the headers
const users = await agent.get('https://api.example.com/users');
const posts = await agent.get('https://api.example.com/posts');

// Set default query parameters
agent.query({ api_key: 'secret123', format: 'json' });

// Every request will include ?api_key=secret123&format=json
const res = await agent.get('https://api.example.com/users');
// Actual URL: https://api.example.com/users?api_key=secret123&format=json

// Set default timeout and retry
agent
  .timeout(5000)
  .retry(2);

// All requests will have 5s timeout and retry up to 2 times
```

### Agent HTTP Methods

Make requests using the agent's configured defaults.

```javascript { .api }
/**
 * Make a GET request with agent defaults
 */
Agent.prototype.get(url: string, callback?: Function): Request;

/**
 * Make a POST request with agent defaults
 */
Agent.prototype.post(url: string, callback?: Function): Request;

/**
 * Make a PUT request with agent defaults
 */
Agent.prototype.put(url: string, callback?: Function): Request;

/**
 * Make a PATCH request with agent defaults
 */
Agent.prototype.patch(url: string, callback?: Function): Request;

/**
 * Make a HEAD request with agent defaults
 */
Agent.prototype.head(url: string, callback?: Function): Request;

/**
 * Make a DELETE request with agent defaults
 */
Agent.prototype.delete(url: string, callback?: Function): Request;

/**
 * Make a DELETE request (alias)
 */
Agent.prototype.del(url: string, callback?: Function): Request;

/**
 * Make an OPTIONS request with agent defaults
 */
Agent.prototype.options(url: string, callback?: Function): Request;
```

**Usage Examples:**

```javascript
const agent = request.agent()
  .set('Authorization', 'Bearer token123')
  .timeout(10000);

// All requests include Authorization header and 10s timeout
const users = await agent.get('https://api.example.com/users');

const newUser = await agent.post('https://api.example.com/users')
  .send({ name: 'John' });

const updated = await agent.put('https://api.example.com/users/123')
  .send({ name: 'Jane' });

await agent.delete('https://api.example.com/users/123');

// Override defaults per-request
const special = await agent.get('https://api.example.com/special')
  .set('X-Special-Header', 'value')
  .timeout(30000); // Override timeout for this request
```

### Agent Event Listeners

Attach event listeners to all requests made through the agent.

```javascript { .api }
/**
 * Add event listener to all requests
 */
Agent.prototype.on(event: string, handler: Function): Agent;

/**
 * Add one-time event listener to all requests
 */
Agent.prototype.once(event: string, handler: Function): Agent;

/**
 * Apply plugin/middleware to all requests
 */
Agent.prototype.use(plugin: Function): Agent;
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Log all requests
agent.on('request', req => {
  console.log('Request:', req.method, req.url);
});

// Log all responses
agent.on('response', res => {
  console.log('Response:', res.status, res.type);
});

// Apply plugin to all requests
function loggerPlugin(req) {
  console.log('Starting request to:', req.url);
  req.on('response', res => {
    console.log('Got response:', res.status);
  });
}

agent.use(loggerPlugin);

// All requests will use the plugin
await agent.get('https://api.example.com/users');
await agent.post('https://api.example.com/posts');
```

## Usage Patterns

### API Client

```javascript
class APIClient {
  constructor(baseURL, token) {
    this.baseURL = baseURL;
    this.agent = request.agent()
      .set('Authorization', `Bearer ${token}`)
      .set('Content-Type', 'application/json')
      .accept('json')
      .timeout(10000)
      .retry(2);
  }

  get(path) {
    return this.agent.get(this.baseURL + path);
  }

  post(path, data) {
    return this.agent.post(this.baseURL + path).send(data);
  }

  put(path, data) {
    return this.agent.put(this.baseURL + path).send(data);
  }

  delete(path) {
    return this.agent.delete(this.baseURL + path);
  }
}

// Usage
const api = new APIClient('https://api.example.com', 'token123');

const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'John' });
const updated = await api.put('/users/123', { name: 'Jane' });
await api.delete('/users/123');
```

### Session Management

```javascript
const agent = request.agent();

async function login(username, password) {
  const res = await agent.post('https://api.example.com/login')
    .send({ username, password });

  console.log('Logged in, session cookie set');
  return res.body;
}

async function getProfile() {
  const res = await agent.get('https://api.example.com/profile');
  return res.body;
}

async function logout() {
  await agent.post('https://api.example.com/logout');
  console.log('Logged out, session cookie cleared');
}

// Usage
await login('john', 'secret');
const profile = await getProfile();
await logout();
```

### Multi-Tenant API Access

```javascript
// Create separate agents for different tenants
const tenantAgents = {};

function getAgentForTenant(tenantId, apiKey) {
  if (!tenantAgents[tenantId]) {
    tenantAgents[tenantId] = request.agent()
      .set('X-Tenant-ID', tenantId)
      .set('X-API-Key', apiKey)
      .timeout(10000);
  }
  return tenantAgents[tenantId];
}

// Usage
const tenant1Agent = getAgentForTenant('tenant1', 'key1');
const tenant2Agent = getAgentForTenant('tenant2', 'key2');

const tenant1Data = await tenant1Agent.get('https://api.example.com/data');
const tenant2Data = await tenant2Agent.get('https://api.example.com/data');
```

### Request Monitoring

```javascript
const agent = request.agent();

let requestCount = 0;
let totalTime = 0;

agent.on('request', req => {
  requestCount++;
  req.startTime = Date.now();
});

agent.on('response', res => {
  const duration = Date.now() - res.req.startTime;
  totalTime += duration;

  console.log(`Request #${requestCount}: ${res.req.method} ${res.req.url}`);
  console.log(`  Status: ${res.status}, Duration: ${duration}ms`);
  console.log(`  Average: ${Math.round(totalTime / requestCount)}ms`);
});

// Make requests
await agent.get('https://api.example.com/users');
await agent.get('https://api.example.com/posts');
await agent.get('https://api.example.com/comments');
```

### Centralized Error Handling

```javascript
const agent = request.agent();

agent.on('response', res => {
  if (res.unauthorized) {
    console.error('Session expired, redirecting to login...');
    window.location = '/login';
  }

  if (res.serverError) {
    console.error('Server error:', res.status);
    // Show error notification
  }
});

// All requests benefit from centralized error handling
await agent.get('https://api.example.com/users');
```

## Important Notes

### Agent vs Request

- **Request**: Single HTTP request with specific configuration
- **Agent**: Manages defaults and state across multiple requests
- Use agents when making multiple requests with shared configuration
- Use direct request methods for one-off requests

### Cookie Persistence

- **Node.js**: Agents automatically persist cookies in a cookie jar
- **Browser**: Browser handles cookies automatically (no agent needed for cookie persistence)
- Cookies are scoped to the agent instance - different agents have separate cookie jars

### Immutability

- Agent configuration methods return the agent for chaining
- Configuration applies to all future requests, not retroactively
- Per-request configuration overrides agent defaults

### Performance

- Reuse agents instead of creating new ones for each request
- Agents maintain connection pools (Node.js) for better performance
- Consider using a single agent instance per API/service
