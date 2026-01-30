# Events and Plugins

Listen to request lifecycle events and extend functionality with plugins.

## Capabilities

### Event Listeners

SuperAgent requests extend EventEmitter and emit events during the request lifecycle.

```javascript { .api }
/**
 * Add event listener
 * @param event - Event name
 * @param listener - Event handler function
 * @returns Request for chaining
 */
request.get(url).on(event: string, listener: Function): Request;

/**
 * Add one-time event listener
 * @param event - Event name
 * @param listener - Event handler function
 * @returns Request for chaining
 */
request.get(url).once(event: string, listener: Function): Request;
```

### Available Events

```javascript
// Request lifecycle events
request
  .get('/api/data')
  .on('request', req => {
    // Emitted when request starts
    console.log('Request started');
  })
  .on('response', res => {
    // Emitted when response is received
    console.log('Response status:', res.status);
  })
  .on('end', () => {
    // Emitted when request completes
    console.log('Request ended');
  })
  .on('error', err => {
    // Emitted on error
    console.error('Error:', err.message);
  })
  .on('abort', () => {
    // Emitted when request is aborted
    console.log('Request aborted');
  });

// Node.js only
request
  .get('/api/redirect')
  .on('redirect', res => {
    // Emitted on redirect
    console.log('Redirected to:', res.headers.location);
  });

// Browser only
request
  .post('/api/upload')
  .attach('file', fileData)
  .on('progress', event => {
    // Upload/download progress
    console.log('Progress:', event.percent);
  });

// Node.js streaming
request
  .post('/api/upload')
  .on('drain', () => {
    // Emitted when write buffer is drained (backpressure cleared)
    console.log('Can write more data');
  });
```

## Event Usage Patterns

### Request Lifecycle Logging

```javascript
function loggedRequest(url) {
  return request
    .get(url)
    .on('request', () => {
      console.log('[REQUEST]', url);
    })
    .on('response', res => {
      console.log('[RESPONSE]', res.status, res.get('Content-Type'));
    })
    .on('end', () => {
      console.log('[END]', url);
    })
    .on('error', err => {
      console.error('[ERROR]', url, err.message);
    });
}

await loggedRequest('/api/users');
```

### Progress Tracking (Browser)

```javascript
// Browser only
function uploadWithProgress(file, onProgress) {
  return request
    .post('/api/upload')
    .attach('file', file)
    .on('progress', event => {
      if (event.direction === 'upload') {
        const percent = event.percent;
        const loaded = event.loaded;
        const total = event.total;
        onProgress(percent, loaded, total);
      }
    });
}

// Usage
uploadWithProgress(fileData, (percent, loaded, total) => {
  console.log(`Upload: ${percent}% (${loaded}/${total} bytes)`);
});
```

### Redirect Tracking (Node.js)

```javascript
// Node.js only
const redirects = [];

request
  .get('/api/resource')
  .redirects(10)
  .on('redirect', res => {
    const location = res.headers.location;
    redirects.push(location);
    console.log('Redirect to:', location);
  })
  .on('response', res => {
    console.log('Final URL:', res.url || 'unknown');
    console.log('Followed', redirects.length, 'redirects');
  })
  .then(res => console.log(res.body));
```

### Abort Handling

```javascript
const req = request.get('/api/slow-endpoint');

req.on('abort', () => {
  console.log('Request was cancelled');
});

// Abort after 5 seconds
setTimeout(() => {
  req.abort();
}, 5000);

req.catch(err => {
  if (err.message === 'Aborted') {
    console.log('Timeout or manual abort');
  }
});
```

## Plugin System

Extend SuperAgent functionality with plugins.

```javascript { .api }
/**
 * Apply plugin function
 * Plugin receives request instance and can modify it
 * @param fn - Plugin function(request): void
 * @returns Request for chaining
 */
request.get(url).use(fn: (req: Request) => void): Request;
```

### Plugin Examples

#### Simple Plugin

```javascript
// Add timestamp header
function timestamp(req) {
  req.set('X-Request-Timestamp', Date.now().toString());
}

request
  .get('/api/data')
  .use(timestamp)
  .then(res => console.log(res.body));
```

#### Authentication Plugin

```javascript
// Reusable auth plugin
function bearerAuth(token) {
  return function(req) {
    req.set('Authorization', `Bearer ${token}`);
  };
}

const authToken = 'your-api-token';

request
  .get('/api/protected')
  .use(bearerAuth(authToken))
  .then(res => console.log(res.body));
```

#### Logging Plugin

```javascript
// Request/response logging
function logger(req) {
  const start = Date.now();

  req.on('request', () => {
    console.log('[LOG]', req.method, req.url);
  });

  req.on('response', res => {
    const duration = Date.now() - start;
    console.log('[LOG]', res.status, `${duration}ms`);
  });

  req.on('error', err => {
    console.error('[LOG] Error:', err.message);
  });
}

request
  .get('/api/users')
  .use(logger)
  .then(res => console.log(res.body));
```

#### Retry Plugin

```javascript
// Custom retry logic plugin
function retryOnTimeout(req) {
  req.retry(3, (err, res) => {
    if (err && err.timeout) {
      console.log('Timeout, retrying...');
      return true;
    }
  });
}

request
  .get('/api/slow-endpoint')
  .use(retryOnTimeout)
  .then(res => console.log(res.body));
```

#### Prefix Plugin

```javascript
// Add base URL prefix
function prefix(baseUrl) {
  return function(req) {
    if (!req.url.startsWith('http')) {
      req.url = baseUrl + req.url;
    }
  };
}

const apiPrefix = prefix('https://api.example.com');

request
  .get('/users')
  .use(apiPrefix)
  .then(res => console.log(res.body));
// Requests: https://api.example.com/users
```

#### Cache Plugin

```javascript
// Simple cache plugin
const cache = new Map();

function cachePlugin(req) {
  const key = `${req.method}:${req.url}`;

  // Check cache
  const cached = cache.get(key);
  if (cached && Date.now() - cached.timestamp < 60000) {
    console.log('Using cached response');
    // Note: This is simplified; real implementation would need
    // to handle promise resolution differently
  }

  // Cache response
  req.on('response', res => {
    cache.set(key, {
      data: res.body,
      timestamp: Date.now()
    });
  });
}

request
  .get('/api/users')
  .use(cachePlugin)
  .then(res => console.log(res.body));
```

#### Headers Plugin

```javascript
// Common headers plugin
function commonHeaders(headers) {
  return function(req) {
    for (const [key, value] of Object.entries(headers)) {
      req.set(key, value);
    }
  };
}

const apiHeaders = commonHeaders({
  'Accept': 'application/json',
  'X-API-Version': '2',
  'X-Client': 'MyApp/1.0'
});

request
  .get('/api/users')
  .use(apiHeaders)
  .then(res => console.log(res.body));
```

### Plugin Composition

```javascript
// Combine multiple plugins
function compose(...plugins) {
  return function(req) {
    plugins.forEach(plugin => plugin(req));
  };
}

const apiPlugin = compose(
  timestamp,
  bearerAuth('token123'),
  logger,
  apiHeaders
);

request
  .get('/api/data')
  .use(apiPlugin)
  .then(res => console.log(res.body));
```

## Plugin Patterns

### Reusable API Client

```javascript
class APIClient {
  constructor(baseUrl, token) {
    this.baseUrl = baseUrl;
    this.token = token;
  }

  plugin(req) {
    // Base URL
    if (!req.url.startsWith('http')) {
      req.url = this.baseUrl + req.url;
    }

    // Authentication
    req.set('Authorization', `Bearer ${this.token}`);

    // Common headers
    req.set('Accept', 'application/json');
    req.set('X-API-Version', '2');

    // Logging
    req.on('request', () => {
      console.log('[API]', req.method, req.url);
    });

    req.on('response', res => {
      console.log('[API]', res.status);
    });

    // Timeout and retry
    req.timeout(10000);
    req.retry(2);
  }

  request(method, path) {
    return request[method](path).use(this.plugin.bind(this));
  }

  get(path) {
    return this.request('get', path);
  }

  post(path, data) {
    return this.request('post', path).send(data);
  }
}

// Usage
const api = new APIClient('https://api.example.com', 'token123');

const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'John' });
```

### Plugin Factory

```javascript
function createPlugin(config) {
  return function(req) {
    // Apply configuration
    if (config.baseUrl && !req.url.startsWith('http')) {
      req.url = config.baseUrl + req.url;
    }

    if (config.auth) {
      req.auth(config.auth.user, config.auth.pass);
    }

    if (config.headers) {
      req.set(config.headers);
    }

    if (config.timeout) {
      req.timeout(config.timeout);
    }

    if (config.retry) {
      req.retry(config.retry);
    }

    if (config.logger) {
      req.on('request', () => {
        config.logger.log('Request:', req.method, req.url);
      });
    }
  };
}

// Usage
const myPlugin = createPlugin({
  baseUrl: 'https://api.example.com',
  auth: { user: 'user', pass: 'pass' },
  headers: { 'X-API-Key': 'key123' },
  timeout: 10000,
  retry: 3,
  logger: console
});

request
  .get('/users')
  .use(myPlugin)
  .then(res => console.log(res.body));
```

## Global Plugin Application

While SuperAgent doesn't have built-in global plugin support, you can create a wrapper:

```javascript
// Wrapper with global plugins
function createRequest(method, url) {
  const req = request[method](url);

  // Apply global plugins
  globalPlugins.forEach(plugin => req.use(plugin));

  return req;
}

const globalPlugins = [
  timestamp,
  logger,
  bearerAuth('token')
];

// Usage
createRequest('get', '/api/users').then(res => console.log(res.body));
createRequest('post', '/api/users').send({ name: 'John' });
```

## Best Practices

### Modular Plugins

```javascript
// Keep plugins focused and reusable
const plugins = {
  auth: token => req => req.auth(token, { type: 'bearer' }),
  timeout: ms => req => req.timeout(ms),
  retry: count => req => req.retry(count),
  logger: logger => req => {
    req.on('request', () => logger.log('Request'));
    req.on('response', res => logger.log('Response', res.status));
  },
  headers: headers => req => req.set(headers)
};

// Compose as needed
request
  .get('/api/data')
  .use(plugins.auth('token123'))
  .use(plugins.timeout(5000))
  .use(plugins.retry(2))
  .use(plugins.logger(console));
```

### Error Handling in Plugins

```javascript
function safePlugin(req) {
  try {
    // Plugin logic that might throw
    req.set('X-Custom', computeHeader());
  } catch (err) {
    console.error('Plugin error:', err);
    // Don't throw - let request continue
  }
}
```

### Plugin Documentation

```javascript
/**
 * Adds request ID tracking
 * @param {Function} req - SuperAgent request
 * @example
 * request.get('/api/data').use(requestId)
 */
function requestId(req) {
  const id = generateId();
  req.set('X-Request-ID', id);
  req.on('response', res => {
    console.log('Request ID:', id, 'Status:', res.status);
  });
}
```

### Drain Event (Node.js Streaming)

When writing large amounts of data with `.write()`, listen to the 'drain' event to handle backpressure.

```javascript
// Handle backpressure when writing
const req = request.post('/api/upload');

function writeChunk(chunk) {
  const canContinue = req.write(chunk);
  
  if (!canContinue) {
    // Buffer is full, wait for drain
    req.once('drain', () => {
      console.log('Buffer drained, can write more');
      // Continue writing more chunks
    });
  }
}

// Stream large file with backpressure handling
const fs = require('fs');
const readStream = fs.createReadStream('large-file.dat');

readStream.on('data', chunk => {
  const canWrite = req.write(chunk);
  
  if (!canWrite) {
    // Pause reading until buffer drains
    readStream.pause();
    
    req.once('drain', () => {
      readStream.resume();
    });
  }
});

readStream.on('end', () => {
  req.end();
});
```
