# Plugins

Extend SuperAgent functionality with plugins for request transformation, caching, authentication, and custom behaviors.

## Capabilities

### Use Plugin

Apply plugins to modify requests before they are sent.

```javascript { .api }
/**
 * Use plugin on request
 * @param {Function} plugin - Plugin function that receives request
 * @returns {Request} Request instance for chaining
 */
Request.prototype.use = function(plugin);

/**
 * Use plugin on agent (applies to all requests)
 * @param {Function} plugin - Plugin function that receives request
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.use = function(plugin);
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Simple plugin
function myPlugin(req) {
  req.set('X-Custom-Header', 'value');
}

// Use on single request
request
  .get('/api/users')
  .use(myPlugin)
  .end(callback);

// Use on agent (applies to all requests)
const agent = request.agent();
agent.use(myPlugin);

agent.get('/api/users').end(callback);
agent.post('/api/data').send({}).end(callback);
```

### Plugin Function Signature

Plugins are functions that receive and modify the request.

```javascript { .api }
/**
 * Plugin function signature
 * @param {Request} req - Request instance to modify
 * @returns {void} - No return value required
 */
function plugin(req) {
  // Modify request
  req.set('header', 'value');
  req.query({ param: 'value' });
  // etc.
}
```

**Usage Examples:**

```javascript
// Basic plugin structure
function basicPlugin(req) {
  req.set('X-Plugin', 'active');
}

// Plugin with parameters (factory pattern)
function createAuthPlugin(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

const authPlugin = createAuthPlugin('my-token');

request
  .get('/api/protected')
  .use(authPlugin)
  .end(callback);

// Plugin with conditional logic
function conditionalPlugin(req) {
  if (req.method === 'POST' || req.method === 'PUT') {
    req.set('X-Mutation', 'true');
  }
}

// Plugin that modifies multiple aspects
function comprehensivePlugin(req) {
  req.set('X-API-Version', '2.0');
  req.query({ format: 'json' });
  req.timeout(5000);
  req.retry(2);
}
```

### Built-in Plugin Patterns

Common plugin patterns for typical use cases.

**Usage Examples:**

```javascript
// 1. Prefix Plugin - Add base URL to all requests
function prefix(baseUrl) {
  return function(req) {
    req.url = baseUrl + req.url;
  };
}

const api = prefix('https://api.example.com');

request.get('/users').use(api).end(callback);
// Requests https://api.example.com/users

// 2. Authentication Plugin
function bearerAuth(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

const auth = bearerAuth('my-access-token');

request.get('/api/protected').use(auth).end(callback);

// 3. API Key Plugin
function apiKey(key) {
  return function(req) {
    req.set('X-API-Key', key);
  };
}

const api = apiKey('secret123');

request.get('/api/data').use(api).end(callback);

// 4. No-Cache Plugin
function noCache() {
  return function(req) {
    req.set('Cache-Control', 'no-cache');
    req.set('Pragma', 'no-cache');
    req.set('Expires', '0');
  };
}

request.get('/api/fresh-data').use(noCache()).end(callback);

// 5. Timeout Plugin
function timeout(ms) {
  return function(req) {
    req.timeout(ms);
  };
}

request.get('/api/data').use(timeout(5000)).end(callback);

// 6. Retry Plugin
function retry(count) {
  return function(req) {
    req.retry(count);
  };
}

request.get('/api/unreliable').use(retry(3)).end(callback);

// 7. User Agent Plugin
function userAgent(ua) {
  return function(req) {
    req.set('User-Agent', ua);
  };
}

request.get('/api/data').use(userAgent('MyApp/1.0')).end(callback);

// 8. Accept JSON Plugin
function acceptJson() {
  return function(req) {
    req.accept('json');
  };
}

request.get('/api/data').use(acceptJson()).end(callback);

// 9. Request ID Plugin
function requestId() {
  return function(req) {
    req.set('X-Request-ID', generateUUID());
  };
}

function generateUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}

request.get('/api/data').use(requestId()).end(callback);
```

### Multiple Plugins

Chain multiple plugins together.

**Usage Examples:**

```javascript
// Apply multiple plugins to single request
request
  .get('/api/users')
  .use(prefix('https://api.example.com'))
  .use(bearerAuth('token'))
  .use(noCache())
  .use(requestId())
  .end(callback);

// Apply multiple plugins to agent
const agent = request.agent();

agent
  .use(prefix('https://api.example.com'))
  .use(bearerAuth('token'))
  .use(userAgent('MyApp/1.0'))
  .use(timeout(5000))
  .use(retry(2));

// All requests through agent use these plugins
agent.get('/users').end(callback);
agent.post('/data').send({}).end(callback);

// Compose plugins
function compose(...plugins) {
  return function(req) {
    plugins.forEach(plugin => plugin(req));
  };
}

const apiPlugin = compose(
  prefix('https://api.example.com'),
  bearerAuth('token'),
  userAgent('MyApp/1.0'),
  timeout(5000)
);

request.get('/users').use(apiPlugin).end(callback);
```

### Advanced Plugin Examples

More sophisticated plugin implementations.

**Usage Examples:**

```javascript
// Logging Plugin
function logger(options = {}) {
  const { verbose = false } = options;

  return function(req) {
    const start = Date.now();

    console.log(`→ ${req.method} ${req.url}`);

    if (verbose) {
      console.log('Headers:', req.header);
    }

    req.on('response', res => {
      const duration = Date.now() - start;
      console.log(`← ${res.status} ${req.method} ${req.url} (${duration}ms)`);

      if (verbose) {
        console.log('Response headers:', res.headers);
      }
    });

    req.on('error', err => {
      const duration = Date.now() - start;
      console.error(`✗ ${req.method} ${req.url} (${duration}ms)`, err.message);
    });
  };
}

agent.use(logger({ verbose: true }));

// Caching Plugin (simple in-memory cache)
function cache(ttl = 60000) {
  const cacheStore = new Map();

  return function(req) {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return;
    }

    const cacheKey = req.url;
    const cached = cacheStore.get(cacheKey);

    if (cached && Date.now() - cached.timestamp < ttl) {
      console.log('Cache hit:', cacheKey);

      // Return cached response
      req.end((err, res) => {
        // Override response with cached data
        Object.assign(res, cached.response);
      });

      return;
    }

    // Store response in cache
    req.on('response', res => {
      cacheStore.set(cacheKey, {
        timestamp: Date.now(),
        response: res
      });
    });
  };
}

agent.use(cache(30000)); // 30 second cache

// Rate Limiting Plugin
function rateLimiter(requestsPerSecond) {
  const queue = [];
  const interval = 1000 / requestsPerSecond;
  let lastRequestTime = 0;

  return function(req) {
    const now = Date.now();
    const timeSinceLastRequest = now - lastRequestTime;

    if (timeSinceLastRequest < interval) {
      const delay = interval - timeSinceLastRequest;

      setTimeout(() => {
        lastRequestTime = Date.now();
      }, delay);
    } else {
      lastRequestTime = now;
    }
  };
}

agent.use(rateLimiter(10)); // Max 10 requests per second

// Circuit Breaker Plugin
function circuitBreaker(options = {}) {
  const {
    threshold = 5,
    timeout = 60000,
    resetTimeout = 30000
  } = options;

  let failures = 0;
  let circuitOpen = false;
  let resetTimer = null;

  return function(req) {
    if (circuitOpen) {
      throw new Error('Circuit breaker is open');
    }

    req.on('error', err => {
      failures++;

      if (failures >= threshold) {
        circuitOpen = true;
        console.error('Circuit breaker opened after', failures, 'failures');

        resetTimer = setTimeout(() => {
          circuitOpen = false;
          failures = 0;
          console.log('Circuit breaker reset');
        }, resetTimeout);
      }
    });

    req.on('response', res => {
      if (res.ok) {
        failures = 0;
      }
    });
  };
}

agent.use(circuitBreaker({ threshold: 3, resetTimeout: 60000 }));

// Mock Plugin (for testing)
function mock(mockResponses) {
  return function(req) {
    const mockKey = `${req.method} ${req.url}`;
    const mockResponse = mockResponses[mockKey];

    if (mockResponse) {
      console.log('Returning mock response for:', mockKey);

      // Simulate async response
      setTimeout(() => {
        req.callback(null, {
          status: mockResponse.status || 200,
          body: mockResponse.body,
          headers: mockResponse.headers || {}
        });
      }, mockResponse.delay || 0);

      // Prevent actual request
      req.abort();
    }
  };
}

const mocks = {
  'GET /api/users': {
    status: 200,
    body: [{ id: 1, name: 'John' }]
  },
  'POST /api/users': {
    status: 201,
    body: { id: 2, name: 'Jane' }
  }
};

const testAgent = request.agent();
testAgent.use(mock(mocks));

// Retry with Backoff Plugin
function retryWithBackoff(maxRetries = 3, baseDelay = 1000) {
  return function(req) {
    let retries = 0;

    req.retry(maxRetries, (err, res) => {
      if (err) {
        retries++;
        const delay = baseDelay * Math.pow(2, retries - 1);

        console.log(`Retry ${retries}/${maxRetries} after ${delay}ms`);

        return new Promise(resolve => {
          setTimeout(() => resolve(true), delay);
        });
      }

      return false;
    });
  };
}

agent.use(retryWithBackoff(3, 1000));
```

### Plugin Composition

Create reusable plugin collections.

**Usage Examples:**

```javascript
// Plugin collection for API client
function createApiClientPlugins(config) {
  const {
    baseUrl,
    apiKey,
    timeout = 5000,
    retries = 2,
    userAgent = 'SuperAgent'
  } = config;

  return [
    prefix(baseUrl),
    apiKey(apiKey),
    function(req) {
      req.set('User-Agent', userAgent);
      req.timeout(timeout);
      req.retry(retries);
      req.accept('json');
    },
    logger(),
    requestId()
  ];
}

// Apply plugin collection
const plugins = createApiClientPlugins({
  baseUrl: 'https://api.example.com',
  apiKey: 'secret123',
  timeout: 10000,
  retries: 3
});

const agent = request.agent();
plugins.forEach(plugin => agent.use(plugin));

// Plugin manager
class PluginManager {
  constructor() {
    this.plugins = [];
  }

  add(plugin) {
    this.plugins.push(plugin);
    return this;
  }

  remove(plugin) {
    const index = this.plugins.indexOf(plugin);
    if (index > -1) {
      this.plugins.splice(index, 1);
    }
    return this;
  }

  apply(req) {
    this.plugins.forEach(plugin => req.use(plugin));
    return req;
  }

  createAgent() {
    const agent = request.agent();
    this.plugins.forEach(plugin => agent.use(plugin));
    return agent;
  }
}

const manager = new PluginManager();
manager.add(prefix('https://api.example.com'));
manager.add(bearerAuth('token'));
manager.add(logger());

const agent = manager.createAgent();
```

### Complete Plugin Example

Comprehensive example showing plugin system.

**Usage Examples:**

```javascript
const request = require('superagent');

// Define plugins
function apiBaseUrl(baseUrl) {
  return function(req) {
    req.url = baseUrl + req.url;
  };
}

function authentication(getToken) {
  return function(req) {
    const token = getToken();
    if (token) {
      req.set('Authorization', `Bearer ${token}`);
    }
  };
}

function requestLogger() {
  return function(req) {
    console.log(`→ ${req.method} ${req.url}`);

    req.on('response', res => {
      console.log(`← ${res.status} ${req.method} ${req.url}`);
    });

    req.on('error', err => {
      console.error(`✗ ${req.method} ${req.url}:`, err.message);
    });
  };
}

function defaultHeaders() {
  return function(req) {
    req.set('Accept', 'application/json');
    req.set('Content-Type', 'application/json');
    req.set('User-Agent', 'MyApp/1.0');
  };
}

function errorHandler() {
  return function(req) {
    req.on('error', err => {
      if (err.response) {
        console.error('Response error:', {
          status: err.response.status,
          body: err.response.body
        });
      } else {
        console.error('Network error:', err.message);
      }
    });
  };
}

// Create API client with plugins
class APIClient {
  constructor(config) {
    this.agent = request.agent();
    this.token = config.token;

    // Apply plugins
    this.agent
      .use(apiBaseUrl(config.baseUrl))
      .use(authentication(() => this.token))
      .use(requestLogger())
      .use(defaultHeaders())
      .use(errorHandler());

    // Set defaults
    this.agent.timeout(config.timeout || 5000);
    this.agent.retry(config.retries || 2);
  }

  setToken(token) {
    this.token = token;
  }

  async getUsers() {
    const res = await this.agent.get('/users');
    return res.body;
  }

  async createUser(userData) {
    const res = await this.agent
      .post('/users')
      .send(userData);
    return res.body;
  }

  async updateUser(id, userData) {
    const res = await this.agent
      .put(`/users/${id}`)
      .send(userData);
    return res.body;
  }

  async deleteUser(id) {
    await this.agent.delete(`/users/${id}`);
  }
}

// Usage
const client = new APIClient({
  baseUrl: 'https://api.example.com',
  token: 'initial-token',
  timeout: 10000,
  retries: 3
});

// All requests use plugins
await client.getUsers();
await client.createUser({ name: 'John' });

// Update token
client.setToken('new-token');

// Subsequent requests use new token
await client.getUsers();
```

### Important Notes

- **Plugin Order**: Plugins are applied in the order they are added. Order matters when plugins depend on each other.
- **Request Modification**: Plugins can modify any aspect of the request: headers, query parameters, timeout, retry logic, etc.
- **Event Listeners**: Plugins can register event listeners on the request to track progress, log activity, or handle errors.
- **Factory Pattern**: Use factory functions to create configurable plugins that accept parameters.
- **Agent vs Request**: Plugins added to an agent apply to all requests made through that agent. Plugins added to a request only apply to that specific request.
- **No Return Value**: Plugin functions don't need to return anything. They modify the request in-place.
- **Async Operations**: Be cautious with async operations in plugins. The request may be sent before async operations complete.
- **Reusability**: Design plugins to be reusable across different projects and use cases.
