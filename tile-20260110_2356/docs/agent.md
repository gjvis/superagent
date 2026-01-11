# Agent

Agent class maintains persistent configuration and cookies across multiple requests, ideal for test suites and authenticated sessions.

## Capabilities

### Create Agent

Factory function and constructor for creating Agent instances.

```javascript { .api }
/**
 * Create a new Agent instance
 * @param {object} [options] - Configuration options
 * @param {Buffer|string} [options.ca] - CA certificate
 * @param {Buffer|string} [options.key] - Client private key
 * @param {Buffer|string} [options.cert] - Client certificate
 * @param {Buffer|string|object} [options.pfx] - PFX/PKCS12 certificate
 * @returns {Agent} Agent instance
 */
function request.agent(options);

/**
 * Agent class constructor
 * @param {object} [options] - Configuration options
 * @returns {Agent} Agent instance
 */
function Agent(options);
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Create agent with factory function
const agent = request.agent();

// Create agent with constructor
const agent = new request.Agent();

// Create agent with TLS options (Node.js)
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem')
});
```

### Agent Properties

Access agent internal properties.

```javascript { .api }
/**
 * Agent properties
 */
interface Agent {
  jar: CookieJar;  // Cookie jar for automatic cookie handling (Node.js only)
}
```

**Usage Examples:**

```javascript
// Access cookie jar (Node.js)
const agent = request.agent();

// CookieJar stores cookies automatically
agent
  .get('/api/login')
  .auth('user', 'pass')
  .end((err, res) => {
    console.log('Cookies stored in jar:', agent.jar);
  });
```

### HTTP Method Shortcuts

Make requests with agent defaults applied.

```javascript { .api }
/**
 * GET request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.get = function(url, callback);

/**
 * POST request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.post = function(url, callback);

/**
 * PUT request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.put = function(url, callback);

/**
 * PATCH request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.patch = function(url, callback);

/**
 * DELETE request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.delete = function(url, callback);

/**
 * DELETE request with agent configuration (alias)
 */
Agent.prototype.del = function(url, callback);

/**
 * HEAD request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.head = function(url, callback);

/**
 * OPTIONS request with agent configuration
 * @param {string} url - Request URL
 * @param {Function} [callback] - Optional callback (err, res)
 * @returns {Request} Request instance
 */
Agent.prototype.options = function(url, callback);
```

**Usage Examples:**

```javascript
const agent = request.agent();

// All requests made through agent share configuration
agent.get('/api/users').end(callback);
agent.post('/api/users').send({ name: 'John' }).end(callback);
agent.put('/api/users/123').send({ name: 'Jane' }).end(callback);
agent.patch('/api/users/123').send({ email: 'new@example.com' }).end(callback);
agent.delete('/api/users/123').end(callback);
agent.head('/api/users/123').end(callback);
agent.options('/api/users').end(callback);

// With callbacks
agent.get('/api/users', (err, res) => {
  console.log(res.body);
});
```

### Configure Agent

Set default configuration that applies to all requests made through the agent.

```javascript { .api }
/**
 * Set default header(s)
 * @param {string|object} field - Header name or object of headers
 * @param {string} [value] - Header value (if field is string)
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.set = function(field, value);

/**
 * Set default authentication
 * @param {string} user - Username or token
 * @param {string} [pass] - Password
 * @param {object} [options] - Authentication options
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.auth = function(user, pass, options);

/**
 * Register plugin for all requests
 * @param {Function} plugin - Plugin function
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.use = function(plugin);
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Set default headers
agent.set('API-Key', 'secret123');
agent.set({
  'User-Agent': 'MyApp/1.0',
  'Accept-Language': 'en-US'
});

// All requests include these headers
agent.get('/api/users').end(callback);
agent.post('/api/data').send({}).end(callback);

// Set default authentication
agent.auth('username', 'password');

// All requests include authentication
agent.get('/api/protected').end(callback);
agent.post('/api/protected').send({}).end(callback);

// Use plugin for all requests
function apiVersionPlugin(req) {
  req.set('API-Version', '2.0');
}

agent.use(apiVersionPlugin);

// All requests go through plugin
agent.get('/api/users').end(callback);
```

### Additional Configuration Methods

All Request configuration methods are available on Agent.

```javascript { .api }
/**
 * Additional agent configuration methods
 */
Agent.prototype.query = function(params);        // Default query parameters
Agent.prototype.type = function(type);           // Default Content-Type
Agent.prototype.accept = function(type);         // Default Accept header
Agent.prototype.withCredentials = function(on);  // Default CORS credentials
Agent.prototype.sortQuery = function(fn);        // Default query sorting
Agent.prototype.retry = function(count, fn);     // Default retry behavior
Agent.prototype.ok = function(callback);         // Default OK validation
Agent.prototype.redirects = function(count);     // Default max redirects
Agent.prototype.timeout = function(options);     // Default timeout
Agent.prototype.buffer = function(enable);       // Default buffering (Node.js)
Agent.prototype.serialize = function(fn);        // Default serializer
Agent.prototype.parse = function(fn);            // Default parser
Agent.prototype.ca = function(cert);             // Default CA certificate (Node.js)
Agent.prototype.key = function(key);             // Default client key (Node.js)
Agent.prototype.cert = function(cert);           // Default client cert (Node.js)
Agent.prototype.pfx = function(pfx);             // Default PFX certificate (Node.js)
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Set default timeout
agent.timeout({ response: 5000, deadline: 10000 });

// Set default retry
agent.retry(2);

// Set default query parameters
agent.query({ api_version: '2.0' });

// Set default content type
agent.type('json');

// Set default accept
agent.accept('json');

// All requests inherit these settings
agent.get('/api/users').end(callback);
agent.post('/api/data').send({ value: 1 }).end(callback);
```

### Cookie Persistence

Agent automatically maintains cookies across requests in Node.js.

**Usage Examples:**

```javascript
const agent = request.agent();

// Login request sets cookies
agent
  .post('/api/login')
  .send({ username: 'user', password: 'pass' })
  .end((err, res) => {
    console.log('Logged in, session cookie saved');

    // Subsequent requests automatically include cookies
    agent
      .get('/api/profile')
      .end((err, res) => {
        console.log('Profile:', res.body);
        // Request includes session cookie automatically
      });

    agent
      .get('/api/settings')
      .end((err, res) => {
        console.log('Settings:', res.body);
        // Request includes session cookie automatically
      });
  });

// Logout clears cookies
agent
  .post('/api/logout')
  .end((err, res) => {
    console.log('Logged out, cookies cleared by server');
  });
```

### Event Listeners

Register event listeners for all agent requests.

```javascript { .api }
/**
 * Register event listener for all requests
 * @param {string} event - Event name
 * @param {Function} callback - Event handler
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.on = function(event, callback);

/**
 * Register one-time event listener
 * @param {string} event - Event name
 * @param {Function} callback - Event handler
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.once = function(event, callback);
```

**Usage Examples:**

```javascript
const agent = request.agent();

// Listen to all request events
agent.on('request', req => {
  console.log('Request started:', req.method, req.url);
});

// Listen to all response events
agent.on('response', res => {
  console.log('Response received:', res.status);
});

// Listen to all error events
agent.on('error', err => {
  console.error('Request error:', err.message);
});

// One-time listener
agent.once('response', res => {
  console.log('First response:', res.status);
});

// Make requests - all trigger events
agent.get('/api/users').end(callback);
agent.post('/api/data').send({}).end(callback);
```

### Use Cases

Common patterns for using agents.

**Usage Examples:**

```javascript
// 1. Test Suite with Shared Session
const agent = request.agent();

describe('API Tests', () => {
  before(done => {
    // Login once
    agent
      .post('/api/login')
      .send({ username: 'test', password: 'test' })
      .end(done);
  });

  it('should get user profile', done => {
    agent
      .get('/api/profile')
      .expect(200)
      .end(done);
  });

  it('should update profile', done => {
    agent
      .put('/api/profile')
      .send({ name: 'New Name' })
      .expect(200)
      .end(done);
  });
});

// 2. API Client with Default Configuration
class APIClient {
  constructor(apiKey) {
    this.agent = request.agent();
    this.agent.set('Authorization', `Bearer ${apiKey}`);
    this.agent.set('User-Agent', 'MyApp/1.0');
    this.agent.accept('json');
    this.agent.timeout(5000);
    this.agent.retry(2);
  }

  getUsers() {
    return this.agent.get('/api/users');
  }

  createUser(data) {
    return this.agent.post('/api/users').send(data);
  }

  updateUser(id, data) {
    return this.agent.put(`/api/users/${id}`).send(data);
  }

  deleteUser(id) {
    return this.agent.delete(`/api/users/${id}`);
  }
}

const client = new APIClient('my-api-key');

// All methods use agent configuration
await client.getUsers();
await client.createUser({ name: 'John' });

// 3. Authenticated Session Manager
class SessionManager {
  constructor() {
    this.agent = null;
  }

  async login(username, password) {
    this.agent = request.agent();

    const res = await this.agent
      .post('/api/login')
      .send({ username, password });

    console.log('Logged in:', res.body.user);
    return res.body;
  }

  async request(method, url, data) {
    if (!this.agent) {
      throw new Error('Not logged in');
    }

    const req = this.agent[method.toLowerCase()](url);

    if (data) {
      req.send(data);
    }

    return req;
  }

  async logout() {
    if (this.agent) {
      await this.agent.post('/api/logout');
      this.agent = null;
    }
  }
}

const session = new SessionManager();
await session.login('user', 'pass');
await session.request('get', '/api/profile');
await session.request('post', '/api/data', { value: 1 });
await session.logout();

// 4. Multi-tenant API Client
class MultiTenantClient {
  constructor() {
    this.agents = new Map();
  }

  getAgent(tenantId) {
    if (!this.agents.has(tenantId)) {
      const agent = request.agent();
      agent.set('X-Tenant-ID', tenantId);
      this.agents.set(tenantId, agent);
    }
    return this.agents.get(tenantId);
  }

  async getTenantData(tenantId) {
    const agent = this.getAgent(tenantId);
    const res = await agent.get('/api/data');
    return res.body;
  }
}

const client = new MultiTenantClient();
await client.getTenantData('tenant-1');
await client.getTenantData('tenant-2');
```

### Complete Agent Example

Comprehensive example showing all agent features.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Create agent with TLS configuration
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem')
});

// Configure agent defaults
agent
  .set('User-Agent', 'MyApp/1.0')
  .set('Accept-Language', 'en-US')
  .accept('json')
  .timeout({ response: 5000, deadline: 10000 })
  .retry(2);

// Add plugin
agent.use(req => {
  req.set('X-Request-ID', generateRequestId());
});

// Listen to events
agent.on('request', req => {
  console.log('→', req.method, req.url);
});

agent.on('response', res => {
  console.log('←', res.status, res.get('Content-Length'));
});

agent.on('error', err => {
  console.error('✗', err.message);
});

// Authenticate
const loginRes = await agent
  .post('/api/login')
  .send({ username: 'user', password: 'pass' });

console.log('Logged in:', loginRes.body.user);

// Make authenticated requests
// Cookies and defaults are automatically applied
const users = await agent.get('/api/users');
console.log('Users:', users.body);

const profile = await agent.get('/api/profile');
console.log('Profile:', profile.body);

const updated = await agent
  .put('/api/profile')
  .send({ name: 'New Name' });
console.log('Updated:', updated.body);

// Logout
await agent.post('/api/logout');
console.log('Logged out');
```

### Important Notes

- **Cookie Persistence**: Agent automatically maintains cookies in Node.js through the internal CookieJar. In browsers, cookies are handled by the browser itself.
- **Default Configuration**: All configuration methods called on the agent apply to every request made through that agent.
- **Plugins**: Plugins registered with `agent.use()` are applied to all requests made through the agent.
- **Event Listeners**: Events registered on the agent fire for every request made through that agent.
- **Independent Agents**: Each agent instance is independent with its own cookies and configuration.
- **Test Suites**: Agents are particularly useful in test suites for maintaining authenticated sessions across multiple test cases.
- **Request Override**: Individual requests can override agent defaults by calling configuration methods on the request itself.
