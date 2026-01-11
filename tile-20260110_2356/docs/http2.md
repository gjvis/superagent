# HTTP/2

Enable HTTP/2 protocol for improved performance with multiplexing, header compression, and server push in Node.js.

## Capabilities

### Enable HTTP/2

Enable HTTP/2 protocol for requests.

```javascript { .api }
/**
 * Enable or disable HTTP/2
 * @param {boolean} [enable] - Enable flag (default: true)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.http2 = function(enable);
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Enable HTTP/2 for single request
request
  .get('https://api.example.com/data')
  .http2()
  .end(callback);

// Explicitly enable
request
  .get('https://api.example.com/data')
  .http2(true)
  .end(callback);

// Disable HTTP/2 (use HTTP/1.1)
request
  .get('https://api.example.com/data')
  .http2(false)
  .end(callback);

// With async/await
try {
  const res = await request
    .get('https://api.example.com/data')
    .http2();

  console.log('Data:', res.body);
} catch (err) {
  console.error('Request failed:', err);
}
```

### HTTP/2 with Agent

Configure HTTP/2 for all requests through an agent.

**Usage Examples:**

```javascript
const request = require('superagent');

// Create agent with HTTP/2 enabled
const agent = request.agent();

// Note: HTTP/2 must be enabled per request
// There's no agent-level HTTP/2 configuration

// Each request needs http2() call
agent.get('https://api.example.com/users').http2().end(callback);
agent.get('https://api.example.com/posts').http2().end(callback);
agent.get('https://api.example.com/comments').http2().end(callback);

// Helper function for HTTP/2 agent
function createHTTP2Client(baseUrl) {
  const agent = request.agent();

  return {
    async get(path) {
      const res = await agent
        .get(baseUrl + path)
        .http2();
      return res.body;
    },

    async post(path, data) {
      const res = await agent
        .post(baseUrl + path)
        .http2()
        .send(data);
      return res.body;
    },

    async put(path, data) {
      const res = await agent
        .put(baseUrl + path)
        .http2()
        .send(data);
      return res.body;
    },

    async delete(path) {
      const res = await agent
        .delete(baseUrl + path)
        .http2();
      return res.body;
    }
  };
}

const client = createHTTP2Client('https://api.example.com');

await client.get('/users');
await client.post('/users', { name: 'John' });
await client.put('/users/123', { name: 'Jane' });
await client.delete('/users/123');
```

### HTTP/2 Benefits

HTTP/2 provides several performance improvements over HTTP/1.1.

**Usage Examples:**

```javascript
const request = require('superagent');

// 1. Multiplexing - Multiple requests over single connection
// HTTP/1.1 approach (sequential or multiple connections)
async function fetchWithHTTP1() {
  const [users, posts, comments] = await Promise.all([
    request.get('https://api.example.com/users'),
    request.get('https://api.example.com/posts'),
    request.get('https://api.example.com/comments')
  ]);

  return { users: users.body, posts: posts.body, comments: comments.body };
}

// HTTP/2 approach (multiplexed over single connection)
async function fetchWithHTTP2() {
  const [users, posts, comments] = await Promise.all([
    request.get('https://api.example.com/users').http2(),
    request.get('https://api.example.com/posts').http2(),
    request.get('https://api.example.com/comments').http2()
  ]);

  return { users: users.body, posts: posts.body, comments: comments.body };
}

// 2. Header Compression - Reduced overhead
// HTTP/2 automatically compresses headers using HPACK
request
  .get('https://api.example.com/data')
  .http2()
  .set({
    'Authorization': 'Bearer long-token-string',
    'User-Agent': 'MyApp/1.0',
    'Accept': 'application/json',
    'Accept-Language': 'en-US',
    'Accept-Encoding': 'gzip, deflate, br'
  })
  .end(callback);

// 3. Request Priority
// HTTP/2 allows request prioritization (handled by protocol)
const highPriority = request
  .get('https://api.example.com/critical-data')
  .http2();

const lowPriority = request
  .get('https://api.example.com/optional-data')
  .http2();

await Promise.all([highPriority, lowPriority]);
```

### HTTP/2 with TLS

HTTP/2 typically requires TLS (HTTPS).

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// HTTP/2 with custom CA certificate
const caCert = fs.readFileSync('ca-cert.pem');

request
  .get('https://api.example.com/data')
  .http2()
  .ca(caCert)
  .end(callback);

// HTTP/2 with mutual TLS
const clientCert = fs.readFileSync('client-cert.pem');
const clientKey = fs.readFileSync('client-key.pem');

request
  .get('https://api.example.com/protected')
  .http2()
  .ca(caCert)
  .cert(clientCert)
  .key(clientKey)
  .end(callback);

// HTTP/2 agent with TLS
const agent = request.agent({
  ca: caCert,
  cert: clientCert,
  key: clientKey
});

await agent
  .get('https://api.example.com/data')
  .http2();
```

### Compatibility and Fallback

Handle HTTP/2 compatibility and fallback scenarios.

**Usage Examples:**

```javascript
const request = require('superagent');

// Try HTTP/2, fallback to HTTP/1.1 on error
async function fetchWithFallback(url) {
  try {
    const res = await request.get(url).http2();
    console.log('Using HTTP/2');
    return res.body;
  } catch (err) {
    if (err.code === 'ERR_HTTP2_ERROR') {
      console.log('HTTP/2 not supported, falling back to HTTP/1.1');
      const res = await request.get(url).http2(false);
      return res.body;
    }
    throw err;
  }
}

await fetchWithFallback('https://api.example.com/data');

// Check HTTP version in response
request
  .get('https://api.example.com/data')
  .http2()
  .end((err, res) => {
    if (res) {
      console.log('HTTP version:', res.request.req.httpVersion);
      // HTTP/2: '2.0'
      // HTTP/1.1: '1.1'
    }
  });

// Conditional HTTP/2 based on environment
const useHTTP2 = process.env.HTTP2_ENABLED === 'true';

async function fetch(url) {
  const req = request.get(url);

  if (useHTTP2) {
    req.http2();
  }

  return req;
}

await fetch('https://api.example.com/data');
```

### Performance Optimization

Optimize performance using HTTP/2 features.

**Usage Examples:**

```javascript
const request = require('superagent');

// 1. Parallel requests (multiplexing)
async function loadDashboard() {
  const start = Date.now();

  // All requests multiplexed over single connection
  const [user, stats, notifications, settings] = await Promise.all([
    request.get('https://api.example.com/user').http2(),
    request.get('https://api.example.com/stats').http2(),
    request.get('https://api.example.com/notifications').http2(),
    request.get('https://api.example.com/settings').http2()
  ]);

  const duration = Date.now() - start;
  console.log(`Loaded dashboard in ${duration}ms`);

  return {
    user: user.body,
    stats: stats.body,
    notifications: notifications.body,
    settings: settings.body
  };
}

await loadDashboard();

// 2. Paginated data loading
async function loadAllPages(baseUrl, totalPages) {
  const requests = [];

  for (let page = 1; page <= totalPages; page++) {
    requests.push(
      request
        .get(`${baseUrl}?page=${page}`)
        .http2()
    );
  }

  const responses = await Promise.all(requests);
  return responses.flatMap(r => r.body);
}

const allData = await loadAllPages('https://api.example.com/data', 10);

// 3. Batch operations
async function batchCreate(items) {
  const requests = items.map(item =>
    request
      .post('https://api.example.com/items')
      .http2()
      .send(item)
  );

  const responses = await Promise.all(requests);
  return responses.map(r => r.body);
}

const newItems = await batchCreate([
  { name: 'Item 1' },
  { name: 'Item 2' },
  { name: 'Item 3' }
]);

// 4. Resource prefetching
async function prefetchResources(urls) {
  const requests = urls.map(url =>
    request.get(url).http2()
  );

  // Fire all requests in parallel
  const responses = await Promise.all(requests);

  console.log('Prefetched', responses.length, 'resources');
  return responses.map(r => r.body);
}

await prefetchResources([
  'https://api.example.com/resource1',
  'https://api.example.com/resource2',
  'https://api.example.com/resource3'
]);
```

### Complete HTTP/2 Example

Comprehensive example showing HTTP/2 usage.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

class HTTP2Client {
  constructor(config) {
    this.baseUrl = config.baseUrl;
    this.agent = request.agent();

    // Configure TLS if provided
    if (config.ca) {
      this.agent.ca(config.ca);
    }
    if (config.cert && config.key) {
      this.agent.cert(config.cert);
      this.agent.key(config.key);
    }

    // Set defaults
    this.agent.set('User-Agent', config.userAgent || 'HTTP2Client/1.0');
    this.agent.timeout(config.timeout || 10000);
    this.agent.retry(config.retries || 2);
  }

  async request(method, path, data) {
    const url = this.baseUrl + path;
    const req = this.agent[method.toLowerCase()](url).http2();

    if (data) {
      req.send(data);
    }

    try {
      const res = await req;
      return res.body;
    } catch (err) {
      // Fallback to HTTP/1.1 on HTTP/2 errors
      if (err.code === 'ERR_HTTP2_ERROR') {
        console.warn('HTTP/2 failed, falling back to HTTP/1.1');

        const req = this.agent[method.toLowerCase()](url).http2(false);

        if (data) {
          req.send(data);
        }

        const res = await req;
        return res.body;
      }

      throw err;
    }
  }

  async get(path) {
    return this.request('GET', path);
  }

  async post(path, data) {
    return this.request('POST', path, data);
  }

  async put(path, data) {
    return this.request('PUT', path, data);
  }

  async delete(path) {
    return this.request('DELETE', path);
  }

  async parallel(...paths) {
    const requests = paths.map(path =>
      this.agent.get(this.baseUrl + path).http2()
    );

    const responses = await Promise.all(requests);
    return responses.map(r => r.body);
  }

  async batch(operations) {
    const requests = operations.map(op => {
      const req = this.agent[op.method.toLowerCase()](this.baseUrl + op.path)
        .http2();

      if (op.data) {
        req.send(op.data);
      }

      return req;
    });

    const responses = await Promise.all(requests);
    return responses.map(r => r.body);
  }
}

// Usage
const client = new HTTP2Client({
  baseUrl: 'https://api.example.com',
  ca: fs.readFileSync('ca-cert.pem'),
  timeout: 5000,
  retries: 3,
  userAgent: 'MyApp/1.0'
});

// Single request
const users = await client.get('/users');
console.log('Users:', users);

// Create resource
const newUser = await client.post('/users', {
  name: 'John Doe',
  email: 'john@example.com'
});
console.log('Created:', newUser);

// Parallel requests (multiplexed)
const [users, posts, comments] = await client.parallel(
  '/users',
  '/posts',
  '/comments'
);
console.log('Loaded all resources');

// Batch operations
const results = await client.batch([
  { method: 'POST', path: '/users', data: { name: 'User 1' } },
  { method: 'POST', path: '/users', data: { name: 'User 2' } },
  { method: 'GET', path: '/stats' }
]);
console.log('Batch results:', results);

// Performance comparison
async function comparePerformance() {
  const urls = Array.from({ length: 10 }, (_, i) =>
    `https://api.example.com/data/${i}`
  );

  // HTTP/1.1
  const http1Start = Date.now();
  const http1Requests = urls.map(url => request.get(url).http2(false));
  await Promise.all(http1Requests);
  const http1Duration = Date.now() - http1Start;

  // HTTP/2
  const http2Start = Date.now();
  const http2Requests = urls.map(url => request.get(url).http2(true));
  await Promise.all(http2Requests);
  const http2Duration = Date.now() - http2Start;

  console.log('HTTP/1.1 duration:', http1Duration + 'ms');
  console.log('HTTP/2 duration:', http2Duration + 'ms');
  console.log('Improvement:', ((http1Duration - http2Duration) / http1Duration * 100).toFixed(2) + '%');
}

await comparePerformance();
```

### Important Notes

- **Node.js Only**: HTTP/2 support is only available in Node.js, not in browsers. Browsers handle HTTP/2 automatically.
- **Node.js Version**: Requires Node.js with HTTP/2 support (Node.js 8.4.0+).
- **TLS Required**: HTTP/2 typically requires TLS (HTTPS). Most servers require HTTPS for HTTP/2.
- **Server Support**: The server must support HTTP/2. If not, requests will fall back to HTTP/1.1 automatically.
- **Multiplexing**: Multiple HTTP/2 requests over the same connection are multiplexed, reducing connection overhead.
- **Header Compression**: HTTP/2 uses HPACK compression for headers, reducing bandwidth usage.
- **Performance**: HTTP/2 is most beneficial when making multiple requests to the same server in parallel.
- **Connection Reuse**: Use agents to reuse connections and maximize HTTP/2 benefits.
- **Debugging**: Check response HTTP version to confirm HTTP/2 is being used.
- **Compatibility**: Always implement fallback logic for servers that don't support HTTP/2.

### HTTP/2 vs HTTP/1.1

Key differences and when to use HTTP/2:

**Use HTTP/2 when:**
- Making multiple parallel requests to the same server
- Transferring many small resources
- Server supports HTTP/2
- Latency is a concern

**Stick with HTTP/1.1 when:**
- Making single requests
- Server doesn't support HTTP/2
- Compatibility is more important than performance
- Using very old Node.js versions

**Performance characteristics:**
- HTTP/2: Single connection, multiplexed streams, compressed headers
- HTTP/1.1: Multiple connections (typically 6-8), head-of-line blocking, larger headers

The performance improvement from HTTP/2 is most noticeable when:
- Loading many resources in parallel
- High latency networks
- Mobile connections
- Server is far from client geographically
