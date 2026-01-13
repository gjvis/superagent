# Agent and Cookie Management

Agents in SuperAgent provide a way to persist settings and cookies across multiple HTTP requests. An agent instance acts as a container that maintains default configurations and, in Node.js environments, automatically manages cookies using a cookie jar. This is particularly useful for maintaining sessions, reusing authentication credentials, and setting common headers or timeouts that apply to all requests.

## Core Concepts

**Agent Instances**: Agents are created using the `agent()` function and provide HTTP verb methods (get, post, put, etc.) that create requests with pre-configured defaults.

**Cookie Persistence (Node.js)**: In Node.js, agents automatically save cookies from responses and attach them to subsequent requests, mimicking browser behavior for session management.

**Default Configuration**: Any configuration method called on an agent (like `.set()`, `.timeout()`, `.auth()`) sets a default that will be applied to all requests created by that agent.

**Browser vs Node.js**: Browser agents provide default configuration but do not manage cookies (browsers handle cookies automatically). Node.js agents include cookie jar functionality using the `cookiejar` package.

## Creating Agents

```javascript { .api }
/**
 * Create a new agent instance
 *
 * In Node.js, creates an agent with a cookie jar for automatic cookie management.
 * In browsers, creates an agent that applies default settings to requests.
 *
 * @param options - Configuration options (Node.js only)
 * @param options.ca - CA certificate for HTTPS (Buffer or string)
 * @param options.key - Client key for HTTPS (Buffer or string)
 * @param options.pfx - PFX/PKCS12 certificate (Buffer or string)
 * @param options.cert - Client certificate for HTTPS (Buffer or string)
 * @returns Agent instance with HTTP verb methods and configuration methods
 */
function agent(options?: {
  ca?: Buffer | string;
  key?: Buffer | string;
  pfx?: Buffer | string;
  cert?: Buffer | string;
}): Agent;
```

### Node.js Agent Creation

```javascript
const request = require('superagent');

// Basic agent with cookie jar
const agent = request.agent();

// Agent with HTTPS certificates
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem')
});
```

### Browser Agent Creation

```javascript
import request from 'superagent';

// Browser agent (no cookie management needed)
const agent = request.agent();
```

## Agent HTTP Methods

Agents provide the same HTTP verb methods as the main `request` object. Each method creates a new request with the agent's default settings applied.

```javascript { .api }
/**
 * HTTP verb methods on Agent
 *
 * Each method creates a new Request instance with agent defaults applied.
 * In Node.js, automatically manages cookies (saves from responses, attaches to requests).
 *
 * @param url - Request URL
 * @param callback - Optional callback function (err, res) => void
 * @returns Request instance with agent defaults applied
 */
interface Agent {
  get(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  post(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  put(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  patch(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  delete(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  del(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  head(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  options(url: string, callback?: (err: Error | null, res: Response) => void): Request;
}
```

### Example Usage

```javascript
const agent = request.agent();

// Make requests with the agent
const users = await agent.get('/api/users');
const user = await agent.post('/api/users').send({ name: 'Alice' });
const updated = await agent.put('/api/users/1').send({ name: 'Alice Smith' });
await agent.delete('/api/users/1');

// With callbacks
agent.get('/api/users', (err, res) => {
  if (err) console.error(err);
  else console.log(res.body);
});
```

## Setting Agent Defaults

All configuration methods available on Request instances are also available on Agent instances. When called on an agent, they set defaults that will be applied to every request created by that agent.

```javascript { .api }
/**
 * Agent configuration methods
 *
 * These methods set default values that apply to all requests created by the agent.
 * All methods return the agent instance for chaining.
 *
 * Available methods (same as Request configuration):
 * - use(fn): Apply plugin to all requests
 * - on(event, handler): Add event listener to all requests
 * - once(event, handler): Add one-time event listener to all requests
 * - set(field, value): Set default header
 * - set(headers): Set multiple default headers
 * - query(params): Add default query parameters
 * - type(contentType): Set default Content-Type
 * - accept(type): Set default Accept header
 * - auth(user, pass, options): Set default authentication
 * - withCredentials(on): Enable credentials for all requests (browser)
 * - sortQuery(comparator): Enable query sorting for all requests
 * - retry(count, callback): Set default retry behavior
 * - ok(callback): Set default success validator
 * - redirects(n): Set default redirect limit
 * - timeout(ms): Set default timeout
 * - buffer(bool): Set default buffering behavior
 * - serialize(fn): Set default request serializer
 * - parse(fn): Set default response parser
 * - ca(cert): Set default CA certificate (Node.js)
 * - key(cert): Set default client key (Node.js)
 * - pfx(cert): Set default PFX certificate (Node.js)
 * - cert(cert): Set default client certificate (Node.js)
 *
 * @returns Agent instance for method chaining
 */
interface Agent {
  use(plugin: (req: Request) => void): Agent;
  on(event: string, handler: Function): Agent;
  once(event: string, handler: Function): Agent;
  set(field: string, value: string): Agent;
  set(headers: object): Agent;
  query(params: object | string): Agent;
  type(contentType: string): Agent;
  accept(type: string): Agent;
  auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Agent;
  withCredentials(on?: boolean): Agent;
  sortQuery(comparator?: (a: string, b: string) => number): Agent;
  retry(count?: number, callback?: Function): Agent;
  ok(callback: (res: Response) => boolean): Agent;
  redirects(n: number): Agent;
  timeout(ms: number | { response?: number; deadline?: number }): Agent;
  buffer(bool?: boolean): Agent;
  serialize(fn: (data: any) => string): Agent;
  parse(fn: (res: Response, callback: Function) => void): Agent;
  ca(cert: Buffer | string): Agent;
  key(cert: Buffer | string): Agent;
  pfx(cert: Buffer | string | { pfx: Buffer | string; passphrase: string }): Agent;
  cert(cert: Buffer | string): Agent;
}
```

### Common Default Settings Examples

```javascript
const agent = request.agent();

// Set default headers
agent.set('Authorization', 'Bearer token123');
agent.set({
  'User-Agent': 'MyApp/1.0',
  'X-API-Key': 'secret123'
});

// Set default timeout
agent.timeout(5000);

// Set default query parameters
agent.query({ apiVersion: 'v2' });

// Set default authentication
agent.auth('username', 'password');

// Set default retry behavior
agent.retry(3);

// Chain multiple defaults
agent
  .set('Accept', 'application/json')
  .timeout(10000)
  .retry(2)
  .redirects(5);

// All subsequent requests inherit these settings
await agent.get('/api/endpoint1'); // has auth header and 5s timeout
await agent.post('/api/endpoint2').send(data); // has auth header and 5s timeout
```

## Cookie Management (Node.js)

In Node.js, agents automatically manage cookies using a cookie jar from the `cookiejar` package. Cookies are saved from responses and attached to subsequent requests, similar to how browsers handle cookies.

```javascript { .api }
/**
 * Cookie jar property (Node.js only)
 *
 * The jar property is a CookieJar instance that stores cookies.
 * Cookies are automatically managed by the agent:
 * - Saved from Set-Cookie headers in responses
 * - Attached to requests based on domain, path, and security settings
 * - Persisted across redirects
 *
 * @property jar - CookieJar instance from the 'cookiejar' package
 */
interface Agent {
  jar: CookieJar; // Node.js only
}

/**
 * Internal cookie management methods (Node.js only)
 *
 * These methods are called automatically by the agent and typically
 * should not be called directly by users.
 */
interface Agent {
  /**
   * Save cookies from response to the agent's cookie jar
   * Called automatically on 'response' and 'redirect' events
   * @param res - Response object containing Set-Cookie headers
   * @private
   */
  _saveCookies(res: Response): void;

  /**
   * Attach cookies to request from the agent's cookie jar
   * Called automatically before sending requests
   * @param req - Request object to attach cookies to
   * @private
   */
  _attachCookies(req: Request): void;

  /**
   * Apply default settings to a request
   * Called automatically when creating requests via agent HTTP methods
   * @param req - Request object to configure
   * @private
   */
  _setDefaults(req: Request): void;
}
```

### Cookie Persistence Example

```javascript
const request = require('superagent');
const agent = request.agent();

// Login request - server responds with Set-Cookie header
const loginRes = await agent
  .post('/api/login')
  .send({ username: 'alice', password: 'secret' });

// The session cookie is automatically saved to agent.jar
console.log('Logged in, session cookie saved');

// Subsequent requests automatically include the session cookie
const profileRes = await agent.get('/api/profile');
// Request includes: Cookie: session=abc123xyz...

// The cookie is maintained across multiple requests
const dataRes = await agent.get('/api/user-data');
// Request includes: Cookie: session=abc123xyz...

// Cookies are also maintained through redirects
const redirectRes = await agent.get('/api/old-endpoint');
// Follows redirect with cookies preserved
```

### Cookie Jar Details

The cookie jar automatically handles:

- **Domain matching**: Cookies are only sent to matching domains
- **Path matching**: Cookies are only sent to matching paths
- **Secure cookies**: HTTPS-only cookies are only sent over secure connections
- **Cookie expiration**: Expired cookies are not sent
- **Multiple cookies**: Multiple cookies can be stored and sent together

```javascript
const agent = request.agent();

// Make a request that sets multiple cookies
await agent.get('/api/init');
// Response headers:
// Set-Cookie: session=abc123; Path=/; HttpOnly
// Set-Cookie: preferences=theme:dark; Path=/; Max-Age=31536000

// Access the cookie jar if needed (advanced usage)
console.log(agent.jar); // CookieJar instance

// All subsequent requests include both cookies
await agent.get('/api/data');
// Request includes: Cookie: session=abc123; preferences=theme:dark
```

## Browser vs Node.js Differences

While agents in both environments share the same API for setting defaults, there are key differences in their implementation and behavior.

### Browser Agent

```javascript
// Browser: Uses lib/agent-base.js only
const agent = request.agent(); // No options parameter

// Stores default settings
agent.set('Authorization', 'Bearer token');

// Creates requests with defaults applied
const req = agent.get('/api/data');
// - Has Authorization header from agent
// - No cookie management (browser handles cookies automatically)

// The browser's native cookie handling works normally
// - Cookies are managed by the browser
// - Same-origin policy applies
// - Use withCredentials() for cross-origin cookies
```

### Node.js Agent

```javascript
// Node.js: Uses lib/node/agent.js (extends lib/agent-base.js)
const agent = request.agent({
  ca: caCert, // Optional HTTPS certificates
  cert: clientCert,
  key: clientKey
});

// Stores default settings (same as browser)
agent.set('Authorization', 'Bearer token');

// Creates requests with defaults AND cookie management
const req = agent.get('/api/data');
// - Has Authorization header from agent
// - Includes cookies from agent.jar
// - Automatically saves cookies from response
// - Maintains cookie jar across requests

// Has a cookie jar property
console.log(agent.jar); // CookieJar instance

// Cookies are automatically managed
// - Saved from Set-Cookie headers
// - Attached to subsequent requests
// - Respect domain, path, and security constraints
```

### Key Differences Summary

| Feature | Browser | Node.js |
|---------|---------|---------|
| Cookie management | Browser native | Automatic via cookie jar |
| Cookie jar property | Not available | `agent.jar` CookieJar instance |
| Constructor options | None | `{ ca, key, pfx, cert }` |
| Implementation | `lib/agent-base.js` | `lib/node/agent.js` + `lib/agent-base.js` |
| Dependencies | None | `cookiejar` package, `url.parse()` |
| Event listeners | None | Listens to 'response' and 'redirect' for cookies |

## Complete Usage Examples

### Session Management (Node.js)

```javascript
const request = require('superagent');
const agent = request.agent();

// Set common defaults
agent
  .set('User-Agent', 'MyApp/1.0')
  .timeout(10000)
  .retry(2);

async function authenticatedFlow() {
  try {
    // Login - receives session cookie
    const loginRes = await agent
      .post('https://api.example.com/login')
      .send({ username: 'alice', password: 'secret' });

    console.log('Logged in:', loginRes.body);

    // Get user profile - cookie automatically included
    const profileRes = await agent.get('https://api.example.com/profile');
    console.log('Profile:', profileRes.body);

    // Update profile - cookie automatically included
    const updateRes = await agent
      .put('https://api.example.com/profile')
      .send({ displayName: 'Alice Smith' });
    console.log('Updated:', updateRes.body);

    // Get protected data - cookie automatically included
    const dataRes = await agent.get('https://api.example.com/user-data');
    console.log('Data:', dataRes.body);

    // Logout - cookie automatically included
    await agent.post('https://api.example.com/logout');
    console.log('Logged out');

  } catch (err) {
    console.error('Error:', err.message);
  }
}

authenticatedFlow();
```

### API Client with Defaults

```javascript
const request = require('superagent');

// Create an agent for a specific API
const apiAgent = request.agent();

// Configure all defaults once
apiAgent
  .set('Authorization', 'Bearer api-token-123')
  .set('User-Agent', 'MyApp/2.0')
  .accept('application/json')
  .timeout({ response: 5000, deadline: 10000 })
  .retry(3)
  .ok(res => res.status < 500); // Retry on 5xx only

// Use the agent for all API calls
async function apiOperations() {
  // All these requests inherit the agent's defaults
  const users = await apiAgent.get('/api/users');

  const newUser = await apiAgent
    .post('/api/users')
    .send({ name: 'Bob', email: 'bob@example.com' });

  const user = await apiAgent.get(`/api/users/${newUser.body.id}`);

  const updated = await apiAgent
    .patch(`/api/users/${newUser.body.id}`)
    .send({ email: 'bob.new@example.com' });

  await apiAgent.delete(`/api/users/${newUser.body.id}`);
}
```

### Multiple Agents for Different Services

```javascript
const request = require('superagent');

// Agent for internal API (with session)
const internalAgent = request.agent()
  .set('X-Internal-Service', 'true')
  .timeout(5000);

// Agent for external API (with API key)
const externalAgent = request.agent()
  .set('Authorization', 'Bearer external-api-key')
  .timeout(15000)
  .retry(5);

// Agent for admin API (with certificates)
const adminAgent = request.agent({
  ca: fs.readFileSync('ca.pem'),
  cert: fs.readFileSync('admin-cert.pem'),
  key: fs.readFileSync('admin-key.pem')
})
  .set('X-Admin-Token', 'admin-secret')
  .timeout(30000);

async function multiServiceWorkflow() {
  // Use different agents for different services
  const internalData = await internalAgent.get('/internal/api/data');
  const externalData = await externalAgent.get('https://external.api/data');
  const adminData = await adminAgent.get('https://admin.internal/data');

  // Each request uses its agent's specific configuration and cookies
}
```

### Plugin with Agent Defaults

```javascript
const request = require('superagent');

// Custom plugin that adds timestamp and request ID
function requestMetadataPlugin(req) {
  req.set('X-Request-ID', Math.random().toString(36).substring(7));
  req.set('X-Timestamp', new Date().toISOString());
}

// Apply plugin to all requests from agent
const agent = request.agent()
  .use(requestMetadataPlugin)
  .set('Authorization', 'Bearer token')
  .timeout(5000);

// Every request from this agent gets:
// - Authorization header
// - 5 second timeout
// - X-Request-ID header
// - X-Timestamp header
await agent.get('/api/data');
await agent.post('/api/data').send({ value: 123 });
```

### Browser Agent Example

```javascript
import request from 'superagent';

// Browser agent for setting common defaults
const agent = request.agent();

// Set common headers for all requests
agent
  .set('X-Requested-With', 'XMLHttpRequest')
  .set('X-App-Version', '1.2.3')
  .timeout(8000)
  .withCredentials(); // Enable sending cookies cross-origin

// Use agent for all API calls
async function fetchData() {
  // All requests include the headers and settings above
  // Browser automatically manages cookies
  const response = await agent.get('/api/data');
  console.log(response.body);
}

// Note: In browsers, cookies are managed natively
// - No cookie jar property
// - No manual cookie management needed
// - Same-origin cookies work automatically
// - Cross-origin requires withCredentials()
```

## Implementation Details

### AgentBase Internal Structure

The `AgentBase` class (used by both browser and Node.js agents) stores default settings in an internal array:

```javascript
// Simplified from lib/agent-base.js
function Agent() {
  this._defaults = []; // Array of { fn, args } objects
}

// When you call agent.set('X-Header', 'value')
// It stores: { fn: 'set', args: ['X-Header', 'value'] }

// When creating a request, _setDefaults applies all stored settings
Agent.prototype._setDefaults = function(req) {
  this._defaults.forEach(def => {
    req[def.fn].apply(req, def.args); // Calls req.set('X-Header', 'value')
  });
};
```

### Node.js Agent Cookie Flow

The Node.js agent extends `AgentBase` and adds cookie management:

```javascript
// Simplified from lib/node/agent.js
const CookieJar = require('cookiejar').CookieJar;
const CookieAccess = require('cookiejar').CookieAccessInfo;

function Agent(options) {
  AgentBase.call(this);
  this.jar = new CookieJar(); // Create cookie jar

  // Apply certificate options if provided
  if (options) {
    if (options.ca) this.ca(options.ca);
    if (options.key) this.key(options.key);
    if (options.pfx) this.pfx(options.pfx);
    if (options.cert) this.cert(options.cert);
  }
}

// Save cookies from Set-Cookie header
Agent.prototype._saveCookies = function(res) {
  const cookies = res.headers['set-cookie'];
  if (cookies) this.jar.setCookies(cookies);
};

// Attach cookies to request
Agent.prototype._attachCookies = function(req) {
  const url = parse(req.url);
  const access = CookieAccess(
    url.hostname,
    url.pathname,
    url.protocol === 'https:'
  );
  const cookies = this.jar.getCookies(access).toValueString();
  req.cookies = cookies; // Set cookies on request
};

// HTTP verb methods
Agent.prototype.get = function(url, fn) {
  const req = new request.Request('GET', url);

  // Set up cookie event listeners
  req.on('response', this._saveCookies.bind(this));
  req.on('redirect', this._saveCookies.bind(this));
  req.on('redirect', this._attachCookies.bind(this, req));

  // Attach cookies for initial request
  this._attachCookies(req);

  // Apply agent defaults
  this._setDefaults(req);

  // Auto-end if callback provided
  if (fn) req.end(fn);

  return req;
};
```

## Type Definitions

```javascript { .api }
/**
 * Complete Agent type definition
 */
interface Agent {
  // Cookie jar (Node.js only)
  jar?: CookieJar;

  // HTTP verb methods
  get(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  post(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  put(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  patch(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  delete(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  del(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  head(url: string, callback?: (err: Error | null, res: Response) => void): Request;
  options(url: string, callback?: (err: Error | null, res: Response) => void): Request;

  // Configuration methods (return Agent for chaining)
  use(plugin: (req: Request) => void): Agent;
  on(event: string, handler: Function): Agent;
  once(event: string, handler: Function): Agent;
  set(field: string, value: string): Agent;
  set(headers: object): Agent;
  query(params: object | string): Agent;
  type(contentType: string): Agent;
  accept(type: string): Agent;
  auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Agent;
  withCredentials(on?: boolean): Agent;
  sortQuery(comparator?: (a: string, b: string) => number): Agent;
  retry(count?: number, callback?: Function): Agent;
  ok(callback: (res: Response) => boolean): Agent;
  redirects(n: number): Agent;
  timeout(ms: number | { response?: number; deadline?: number }): Agent;
  buffer(bool?: boolean): Agent;
  serialize(fn: (data: any) => string): Agent;
  parse(fn: (res: Response, callback: Function) => void): Agent;

  // Node.js specific configuration methods
  ca(cert: Buffer | string): Agent;
  key(cert: Buffer | string): Agent;
  pfx(cert: Buffer | string | { pfx: Buffer | string; passphrase: string }): Agent;
  cert(cert: Buffer | string): Agent;

  // Internal methods (private, not for direct use)
  _defaults: Array<{ fn: string; args: any[] }>;
  _setDefaults(req: Request): void;
  _saveCookies?(res: Response): void; // Node.js only
  _attachCookies?(req: Request): void; // Node.js only
}

/**
 * CookieJar type from 'cookiejar' package (Node.js only)
 */
interface CookieJar {
  setCookies(cookies: string | string[]): void;
  getCookies(access: CookieAccessInfo): Cookie[];
}

/**
 * Cookie access information for domain/path matching
 */
interface CookieAccessInfo {
  domain: string;
  path: string;
  secure: boolean;
}

/**
 * Individual cookie object
 */
interface Cookie {
  name: string;
  value: string;
  path: string;
  domain: string;
  secure: boolean;
  httponly: boolean;
  toValueString(): string;
}
```

## Best Practices

### When to Use Agents

**Use agents when:**
- You need to maintain session state across multiple requests (Node.js)
- You want to set common defaults for a group of related requests
- You're building an API client with consistent configuration
- You need to authenticate once and reuse credentials across many requests
- You're making requests to multiple endpoints with shared settings

**Don't use agents when:**
- You're only making a single request
- Each request has completely different configuration
- You need isolated cookie spaces (create separate agents instead)

### Configuration Best Practices

```javascript
// Good: Set all defaults at agent creation
const agent = request.agent()
  .set('User-Agent', 'MyApp/1.0')
  .timeout(5000)
  .retry(2);

// Good: Override defaults on individual requests when needed
await agent.get('/api/data').timeout(10000); // This one request gets 10s timeout

// Good: Use multiple agents for different contexts
const adminAgent = request.agent().set('X-Admin', 'true');
const userAgent = request.agent().set('X-User', 'true');

// Avoid: Mixing agent and non-agent requests in session flows
// Bad (loses session):
await agent.post('/login').send(credentials);
await request.get('/profile'); // No session cookie!

// Good (maintains session):
await agent.post('/login').send(credentials);
await agent.get('/profile'); // Has session cookie
```

### Cookie Security (Node.js)

```javascript
// Cookie jar respects security constraints automatically
const agent = request.agent();

// Secure cookies only sent over HTTPS
await agent.get('https://example.com/login'); // Sets secure cookie
await agent.get('http://example.com/data'); // Secure cookie NOT sent

// Cookies respect domain and path
await agent.get('https://api.example.com/v1/login'); // Sets cookie for api.example.com
await agent.get('https://example.com/data'); // Cookie NOT sent (different subdomain)

// HttpOnly cookies are handled correctly
// (cannot be accessed via JavaScript, but sent in requests)
```

### Error Handling with Agents

```javascript
const agent = request.agent()
  .timeout(5000)
  .retry(2);

try {
  const res = await agent.get('/api/data');
  console.log(res.body);
} catch (err) {
  if (err.timeout) {
    console.error('Request timed out after 5s');
  } else if (err.status === 401) {
    console.error('Authentication failed');
    // Maybe login again with agent
  } else {
    console.error('Request failed:', err.message);
  }
}
```

## Related Documentation

- [Making HTTP Requests](./making-requests.md) - Basic request creation and HTTP verbs
- [Request Configuration](./request-configuration.md) - All configuration methods available as agent defaults
- [Authentication](./authentication.md) - Setting authentication defaults on agents
- [Timeouts and Retries](./timeouts-retries.md) - Configuring timeout and retry defaults
- [Advanced Features](./advanced-features.md) - Plugins and other features that work with agents
