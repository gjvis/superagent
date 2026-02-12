# Plugins

Extend request functionality using plugins.

## Capabilities

### Using Plugins

Apply plugins to requests to add cross-cutting functionality.

```javascript { .api }
/**
 * Apply plugin to request
 * @param plugin - Plugin function that receives the request
 * @returns Request instance for chaining
 */
use(plugin: (req: Request) => void): Request;
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Simple plugin that adds a header
function addTimestamp(req) {
  req.set('X-Request-Time', new Date().toISOString());
}

request
  .get('/api/users')
  .use(addTimestamp)
  .end((err, res) => {
    // Request includes X-Request-Time header
  });

// Plugin with configuration
function addApiKey(key) {
  return function(req) {
    req.set('X-API-Key', key);
  };
}

request
  .get('/api/data')
  .use(addApiKey('my-secret-key'))
  .end((err, res) => {
    console.log(res.body);
  });

// Multiple plugins
request
  .post('/api/users')
  .use(addTimestamp)
  .use(addApiKey('key123'))
  .send({ name: 'John' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### Plugin Examples

Common plugin patterns and examples.

**Prefix Plugin:**

```javascript
// Add URL prefix to all requests
function prefix(base) {
  return function(req) {
    req.url = base + req.url;
  };
}

const prefixApi = prefix('https://api.example.com');

request
  .get('/users')
  .use(prefixApi)
  .end((err, res) => {
    // Requests https://api.example.com/users
  });
```

**No-Cache Plugin:**

```javascript
// Prevent caching
function noCache(req) {
  req.set('Cache-Control', 'no-cache');
  req.set('Pragma', 'no-cache');
  req.set('Expires', '0');
}

request
  .get('/api/fresh-data')
  .use(noCache)
  .end((err, res) => {
    // Response won't be cached
  });
```

**Logging Plugin:**

```javascript
// Log all requests
function logger(req) {
  const start = Date.now();

  // Log request
  console.log(`→ ${req.method} ${req.url}`);

  req.on('response', (res) => {
    const duration = Date.now() - start;
    console.log(`← ${res.status} ${req.method} ${req.url} (${duration}ms)`);
  });

  req.on('error', (err) => {
    const duration = Date.now() - start;
    console.log(`✗ ${req.method} ${req.url} (${duration}ms) - ${err.message}`);
  });
}

request
  .get('/api/users')
  .use(logger)
  .end((err, res) => {
    // Logs request and response
  });
```

**Authentication Plugin:**

```javascript
// Add Bearer token authentication
function bearerToken(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

const authToken = localStorage.getItem('token');

request
  .get('/api/protected')
  .use(bearerToken(authToken))
  .end((err, res) => {
    console.log(res.body);
  });
```

**Retry Plugin:**

```javascript
// Add retry logic with exponential backoff
function retryWithBackoff(maxRetries = 3) {
  return function(req) {
    let retryCount = 0;

    req.retry(maxRetries, (err, res) => {
      if (err && err.status >= 500) {
        retryCount++;
        const delay = Math.pow(2, retryCount) * 1000;
        console.log(`Retry ${retryCount} after ${delay}ms`);
        return true; // retry
      }
      return false; // don't retry
    });
  };
}

request
  .get('/api/unreliable')
  .use(retryWithBackoff(3))
  .end((err, res) => {
    console.log(res.body);
  });
```

**Request ID Plugin:**

```javascript
// Add unique request ID
function requestId(req) {
  const id = Math.random().toString(36).substr(2, 9);
  req.set('X-Request-ID', id);

  req.on('response', (res) => {
    console.log(`Request ${id} completed:`, res.status);
  });
}

request
  .post('/api/users')
  .use(requestId)
  .send({ name: 'John' })
  .end((err, res) => {
    console.log(res.body);
  });
```

### Plugin with Agent

Apply plugins to all requests from an agent.

**Usage Examples:**

```javascript
const agent = request.agent();

// Apply plugin to agent (affects all requests)
agent.use(addTimestamp);
agent.use(addApiKey('key123'));

// All agent requests use these plugins
agent.get('/api/users').end((err, res) => {
  // Includes timestamp and API key
});

agent.post('/api/data').send({...}).end((err, res) => {
  // Also includes timestamp and API key
});

// Combine agent plugins with request-specific plugins
agent
  .get('/api/special')
  .use(noCache) // Request-specific plugin
  .end((err, res) => {
    // Has agent plugins + no-cache plugin
  });
```

### Popular Third-Party Plugins

SuperAgent has an ecosystem of community plugins.

**Examples of popular plugins:**

```javascript
// superagent-no-cache
const nocache = require('superagent-no-cache');
request.get('/api/data').use(nocache).end(...);

// superagent-prefix
const prefix = require('superagent-prefix');
request.get('/users').use(prefix('/api/v1')).end(...);

// superagent-throttle
const throttle = require('superagent-throttle');
const myThrottle = throttle.plugin({
  rate: 5,        // 5 requests
  ratePer: 1000,  // per 1000ms
  concurrent: 2   // max 2 concurrent
});
request.get('/api/data').use(myThrottle).end(...);

// superagent-cache
const cache = require('superagent-cache');
request.get('/api/data').use(cache).end(...);

// superagent-jsonapify
const jsonapify = require('superagent-jsonapify');
request.get('/api/articles').use(jsonapify()).end(...);
```

### Creating Reusable Plugins

Best practices for creating plugins.

**Plugin Template:**

```javascript
// Basic plugin
function myPlugin(req) {
  // Modify request
  req.set('X-Custom-Header', 'value');

  // Listen to events
  req.on('response', (res) => {
    // Handle response
  });

  req.on('error', (err) => {
    // Handle error
  });
}

// Plugin factory (with configuration)
function myConfigurablePlugin(options) {
  return function(req) {
    // Use options to configure behavior
    if (options.addHeader) {
      req.set(options.headerName, options.headerValue);
    }

    if (options.timeout) {
      req.timeout(options.timeout);
    }
  };
}

// Usage
request
  .get('/api/data')
  .use(myConfigurablePlugin({
    addHeader: true,
    headerName: 'X-Custom',
    headerValue: 'value',
    timeout: 5000
  }))
  .end((err, res) => {
    console.log(res.body);
  });
```

**Advanced Plugin:**

```javascript
// Plugin with state and methods
function createAdvancedPlugin(config) {
  let requestCount = 0;

  return function(req) {
    requestCount++;

    // Add request counter
    req.set('X-Request-Count', requestCount.toString());

    // Add timestamp if configured
    if (config.includeTimestamp) {
      req.set('X-Timestamp', Date.now().toString());
    }

    // Log if configured
    if (config.logging) {
      console.log(`Request #${requestCount}: ${req.method} ${req.url}`);
    }

    // Retry logic
    if (config.autoRetry) {
      req.retry(config.maxRetries || 2);
    }

    // Timeout
    if (config.timeout) {
      req.timeout(config.timeout);
    }

    // Event handlers
    req.on('response', (res) => {
      if (config.logging) {
        console.log(`Response #${requestCount}: ${res.status}`);
      }
    });
  };
}

// Create and use plugin instance
const myPlugin = createAdvancedPlugin({
  includeTimestamp: true,
  logging: true,
  autoRetry: true,
  maxRetries: 3,
  timeout: 10000
});

request.get('/api/data').use(myPlugin).end(...);
request.post('/api/users').use(myPlugin).send({...}).end(...);
```

## Plugin Execution Order

Plugins are executed in the order they are added.

```javascript
request
  .get('/api/data')
  .use(plugin1) // Executes first
  .use(plugin2) // Executes second
  .use(plugin3) // Executes third
  .end((err, res) => {
    console.log(res.body);
  });
```
