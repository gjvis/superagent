# Request Configuration

Request configuration methods in SuperAgent provide a fluent interface for setting headers, query parameters, request bodies, and controlling request behavior. All methods return the Request instance for method chaining, allowing you to build complex requests declaratively.

## Header Management

### Setting Headers

Set request headers using `.set()` with either individual field/value pairs or multiple headers at once with an object. Header names are case-insensitive and stored internally in lowercase.

```javascript { .api }
/**
 * Set request header field to value
 * @param field - Header field name (case-insensitive)
 * @param value - Header field value
 * @returns Request instance for chaining
 */
set(field: string, value: string): Request;

/**
 * Set multiple headers at once
 * @param fields - Object with header field/value pairs
 * @returns Request instance for chaining
 */
set(fields: object): Request;
```

**Usage Examples:**

```javascript
// Set individual header
request
  .get('/api/users')
  .set('Accept', 'application/json')
  .set('X-API-Key', 'foobar');

// Set multiple headers at once
request
  .get('/api/users')
  .set({
    'Accept': 'application/json',
    'X-API-Key': 'foobar',
    'X-Requested-With': 'XMLHttpRequest'
  });

// Headers are case-insensitive
request
  .get('/api/users')
  .set('content-type', 'application/json')  // stored as lowercase
  .set('Content-Type', 'text/html');        // overwrites previous value
```

### Getting Headers

Retrieve a previously set request header value. Returns undefined if header is not set.

```javascript { .api }
/**
 * Get request header field value
 * @param field - Header field name (case-insensitive)
 * @returns Header value or undefined if not set
 */
get(field: string): string | undefined;
```

**Usage Examples:**

```javascript
request
  .post('/api/users')
  .set('Content-Type', 'application/json')
  .set('Authorization', 'Bearer token123');

const contentType = request.get('content-type');  // 'application/json'
const auth = request.get('Authorization');         // 'Bearer token123'
const missing = request.get('X-Custom');           // undefined
```

### Removing Headers

Remove a previously set header from the request.

```javascript { .api }
/**
 * Remove request header field
 * @param field - Header field name (case-insensitive)
 * @returns Request instance for chaining
 */
unset(field: string): Request;
```

**Usage Examples:**

```javascript
// Remove a header
request
  .get('/api/users')
  .set('X-Custom-Header', 'value')
  .unset('X-Custom-Header');  // header removed

// Useful for removing default headers
request
  .get('/api/users')
  .unset('User-Agent');  // removes default User-Agent header
```

### Content-Type Shorthand

Set the Content-Type header using short type names or full MIME types. Short names are automatically mapped to their corresponding MIME types.

```javascript { .api }
/**
 * Set Content-Type header
 * @param type - Short type name ('json', 'xml', 'form', etc.) or full MIME type
 * @returns Request instance for chaining
 */
type(type: string): Request;
```

**Supported Short Type Names:**
- `html` → `text/html`
- `json` → `application/json`
- `xml` → `text/xml`
- `urlencoded` or `form` → `application/x-www-form-urlencoded`
- `form-data` → `application/x-www-form-urlencoded`

**Usage Examples:**

```javascript
// Using short type names
request
  .post('/api/users')
  .type('json')  // sets Content-Type: application/json
  .send({ name: 'Alice' });

request
  .post('/api/form')
  .type('form')  // sets Content-Type: application/x-www-form-urlencoded
  .send('name=Alice&email=alice@example.com');

// Using full MIME type
request
  .post('/api/data')
  .type('application/vnd.api+json')
  .send({ data: { type: 'users', id: '1' } });

// For XML data
request
  .post('/api/soap')
  .type('xml')
  .send('<user><name>Alice</name></user>');

// Custom MIME types
request
  .post('/api/custom')
  .type('application/x-custom-format')
  .send(customData);
```

### Accept Header Shorthand

Set the Accept header to specify the preferred response content type using short type names or full MIME types.

```javascript { .api }
/**
 * Set Accept header
 * @param type - Short type name ('json', 'xml', 'html', etc.) or full MIME type
 * @returns Request instance for chaining
 */
accept(type: string): Request;
```

**Usage Examples:**

```javascript
// Request JSON response
request
  .get('/api/users')
  .accept('json');  // sets Accept: application/json

// Request XML response
request
  .get('/api/data')
  .accept('xml');  // sets Accept: text/xml

// Request HTML response
request
  .get('/page')
  .accept('html');  // sets Accept: text/html

// Using full MIME type with vendor-specific format
request
  .get('/api/v2/users')
  .accept('application/vnd.api+json');

// Specify multiple accepted types manually
request
  .get('/api/data')
  .set('Accept', 'application/json, text/plain, */*');
```

## Query Parameters

### Adding Query Parameters

Add query string parameters to the request URL. Accepts either a query string or an object. Multiple calls to `.query()` accumulate parameters.

```javascript { .api }
/**
 * Add query string parameters
 * @param params - Query parameters as object or string
 * @returns Request instance for chaining
 */
query(params: object | string): Request;
```

**Usage Examples:**

```javascript
// Using object notation
request
  .get('/api/search')
  .query({ q: 'nodejs', limit: 10, offset: 0 });
// Results in: /api/search?q=nodejs&limit=10&offset=0

// Using string notation
request
  .get('/api/search')
  .query('q=nodejs&limit=10');
// Results in: /api/search?q=nodejs&limit=10

// Multiple query calls accumulate
request
  .get('/api/search')
  .query({ q: 'nodejs' })
  .query({ limit: 10 })
  .query({ offset: 0 });
// Results in: /api/search?q=nodejs&limit=10&offset=0

// Mixing object and string notation
request
  .get('/api/search')
  .query({ q: 'nodejs' })
  .query('sort=relevance')
  .query({ limit: 10 });
// Results in: /api/search?q=nodejs&sort=relevance&limit=10

// Array values are properly encoded
request
  .get('/api/items')
  .query({ tags: ['javascript', 'nodejs', 'web'] });
// Results in: /api/items?tags=javascript&tags=nodejs&tags=web

// Nested objects are serialized
request
  .get('/api/users')
  .query({
    filter: {
      status: 'active',
      role: 'admin'
    }
  });
// Results in: /api/users?filter[status]=active&filter[role]=admin

// Empty or null values
request
  .get('/api/data')
  .query({ key: 'value', empty: '', nullValue: null });
// Results in: /api/data?key=value&empty=&nullValue=
```

### Sorting Query Parameters

Enable alphabetical sorting of query string parameters. Useful for API keys that rely on query parameter order or for consistent URL generation.

```javascript { .api }
/**
 * Sort query string parameters alphabetically
 * @param comparator - Optional custom sort function (a, b) => number
 * @returns Request instance for chaining
 */
sortQuery(comparator?: Function): Request;
```

**Usage Examples:**

```javascript
// Default alphabetical sorting
request
  .get('/api/data')
  .query({ z: 'last', a: 'first', m: 'middle' })
  .sortQuery();
// Results in: /api/data?a=first&m=middle&z=last

// Custom sort function - sort by length
request
  .get('/api/data')
  .query({ name: 'Alice', id: '123', email: 'alice@example.com' })
  .sortQuery((a, b) => a.length - b.length);
// Results in: /api/data?id=123&name=Alice&email=alice@example.com

// Sort for consistent cache keys
const cacheKey = request
  .get('/api/search')
  .query({ term: 'nodejs', page: 2, limit: 10 })
  .sortQuery()
  .url;
// Always produces: /api/search?limit=10&page=2&term=nodejs

// Useful for API signature generation
request
  .get('/api/protected')
  .query({
    timestamp: Date.now(),
    nonce: generateNonce(),
    apiKey: 'key123'
  })
  .sortQuery()  // ensures consistent order for signature
  .set('X-Signature', generateSignature(request.url));
```

## Request Body

### Sending Data

Send request body data with automatic Content-Type detection and serialization. Objects are automatically JSON-stringified unless a different Content-Type is set.

```javascript { .api }
/**
 * Send request body data
 * @param data - Data to send (object, string, Buffer, etc.)
 * @returns Request instance for chaining
 */
send(data: any): Request;
```

**Automatic Content-Type Behavior:**
- **Object without Content-Type**: Automatically sets `Content-Type: application/json` and JSON-stringifies the data
- **Object with `form` type**: Serializes as `application/x-www-form-urlencoded`
- **String without Content-Type**: Sets `Content-Type: application/x-www-form-urlencoded`
- **String with existing type**: Uses the specified Content-Type

**Usage Examples:**

```javascript
// Automatic JSON serialization
request
  .post('/api/users')
  .send({ name: 'Alice', email: 'alice@example.com' });
// Automatically sets Content-Type: application/json
// Body: {"name":"Alice","email":"alice@example.com"}

// Multiple send() calls merge objects
request
  .post('/api/users')
  .send({ name: 'Alice' })
  .send({ email: 'alice@example.com' })
  .send({ role: 'admin' });
// Body: {"name":"Alice","email":"alice@example.com","role":"admin"}

// Sending form-encoded data with string
request
  .post('/api/login')
  .type('form')
  .send('username=alice')
  .send('password=secret');
// Content-Type: application/x-www-form-urlencoded
// Body: username=alice&password=secret

// Sending form-encoded data with object
request
  .post('/api/login')
  .type('form')
  .send({ username: 'alice', password: 'secret' });
// Content-Type: application/x-www-form-urlencoded
// Body: username=alice&password=secret

// Sending plain text
request
  .post('/api/log')
  .type('text/plain')
  .send('Log entry: System started');

// Sending XML
request
  .post('/api/soap')
  .type('application/xml')
  .send('<user><name>Alice</name></user>');

// Sending arrays
request
  .post('/api/batch')
  .send([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' }
  ]);
// Body: [{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]

// Sending Buffer (Node.js)
const buffer = Buffer.from('binary data');
request
  .post('/api/upload')
  .type('application/octet-stream')
  .send(buffer);

// Cannot mix send() with attach() or field()
request
  .post('/api/upload')
  .send({ data: 'value' })
  .attach('file', './doc.pdf');  // Error: can't mix .send() with .attach()
```

## Redirects

### Setting Redirect Limits

Control the maximum number of HTTP redirects to follow. By default, GET requests follow up to 5 redirects, while HEAD requests don't follow redirects.

```javascript { .api }
/**
 * Set maximum number of redirects to follow
 * @param n - Maximum redirect count (0 to disable)
 * @returns Request instance for chaining
 */
redirects(n: number): Request;
```

**Usage Examples:**

```javascript
// Disable redirects
request
  .get('/api/endpoint')
  .redirects(0);  // don't follow any redirects

// Follow up to 10 redirects
request
  .get('/api/endpoint')
  .redirects(10);

// Useful for strict redirect checking
request
  .get('/api/resource')
  .redirects(0)
  .then(res => {
    if (res.status === 301 || res.status === 302) {
      console.log('Redirect to:', res.header.location);
    }
  });

// Allow deep redirect chains
request
  .get('/api/nested/redirect')
  .redirects(20)  // handle complex redirect chains
  .end(callback);
```

## Response Configuration

### Buffer Control

Enable or disable response body buffering. When buffering is disabled, the response body won't be stored in memory, useful for large responses or streaming scenarios.

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param enable - True to buffer (default), false to disable
 * @returns Request instance for chaining
 */
buffer(enable: boolean): Request;
```

**Usage Examples:**

```javascript
// Disable buffering for large files (Node.js)
request
  .get('/api/large-file.zip')
  .buffer(false)
  .pipe(fs.createWriteStream('./download.zip'));

// Enable buffering explicitly (default behavior)
request
  .get('/api/data')
  .buffer(true)
  .then(res => {
    console.log(res.body);  // buffered in memory
  });

// Disable buffering to handle response manually
request
  .get('/api/stream')
  .buffer(false)
  .on('response', res => {
    res.on('data', chunk => {
      // process chunk by chunk
      processChunk(chunk);
    });
  });

// Control buffering based on response type
const shouldBuffer = contentType.includes('application/json');
request
  .get('/api/endpoint')
  .buffer(shouldBuffer)
  .end(callback);
```

### Response Type

Set the desired format of the binary response body. In browsers, valid values are 'blob' and 'arraybuffer'. In Node.js, all values result in a Buffer.

```javascript { .api }
/**
 * Set format of binary response body
 * @param type - Response type ('blob', 'arraybuffer', etc.)
 * @returns Request instance for chaining
 */
responseType(type: string): Request;
```

**Browser Response Types:**
- `'blob'` - Returns a Blob object
- `'arraybuffer'` - Returns an ArrayBuffer object

**Usage Examples:**

```javascript
// Browser: Get response as Blob for file download
request
  .get('/api/image.png')
  .responseType('blob')
  .then(res => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    downloadLink.href = url;
  });

// Browser: Get response as ArrayBuffer for binary processing
request
  .get('/api/audio.mp3')
  .responseType('arraybuffer')
  .then(res => {
    const audioContext = new AudioContext();
    audioContext.decodeAudioData(res.body);
  });

// Node.js: All types return Buffer
request
  .get('/api/binary-data')
  .responseType('buffer')
  .then(res => {
    const buffer = res.body;  // Buffer instance
    processBuffer(buffer);
  });

// Handle images in browser
request
  .get('/api/photo.jpg')
  .responseType('blob')
  .then(res => {
    const img = document.createElement('img');
    img.src = URL.createObjectURL(res.body);
    document.body.appendChild(img);
  });
```

### Maximum Response Size

Set the maximum allowed size of the response body in bytes. Prevents memory issues from unexpectedly large responses. The default limit is 200MB (209,715,200 bytes).

```javascript { .api }
/**
 * Set maximum response body size in bytes
 * @param bytes - Maximum size in bytes
 * @returns Request instance for chaining
 * @throws TypeError if argument is not a number
 */
maxResponseSize(bytes: number): Request;
```

**Usage Examples:**

```javascript
// Limit response to 10MB
request
  .get('/api/data')
  .maxResponseSize(10 * 1024 * 1024);  // 10MB in bytes

// Strict limit for API responses
request
  .get('/api/users')
  .maxResponseSize(1024 * 1024)  // 1MB max
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    if (err.message.includes('maximum')) {
      console.error('Response too large');
    }
  });

// Different limits for different endpoints
const endpoints = {
  '/api/thumbnail': 100 * 1024,        // 100KB
  '/api/image': 5 * 1024 * 1024,       // 5MB
  '/api/video': 50 * 1024 * 1024       // 50MB
};

request
  .get('/api/image')
  .maxResponseSize(endpoints['/api/image'])
  .end(callback);

// Prevent abuse from untrusted sources
request
  .get(userProvidedUrl)
  .maxResponseSize(512 * 1024)  // 512KB max for user URLs
  .timeout(5000)
  .end(callback);
```

## Success Validation

### Custom Success Criteria

Define a custom function to determine if a response should be considered successful. By default, 2xx status codes are considered successful.

```javascript { .api }
/**
 * Set custom success validation function
 * @param callback - Function that receives response and returns true if successful
 * @returns Request instance for chaining
 * @throws Error if callback is not a function
 */
ok(callback: (res: Response) => boolean): Request;
```

**Usage Examples:**

```javascript
// Accept 404 as success for existence checks
request
  .get('/api/users/123')
  .ok(res => res.status === 200 || res.status === 404)
  .then(res => {
    if (res.status === 404) {
      console.log('User does not exist');
    } else {
      console.log('User found:', res.body);
    }
  });

// Accept specific error codes as success
request
  .post('/api/create')
  .send(data)
  .ok(res => res.status < 500)  // accept all 2xx, 3xx, 4xx
  .then(res => {
    if (res.status === 409) {
      console.log('Resource already exists');
    } else if (res.status === 201) {
      console.log('Resource created');
    }
  });

// Custom success based on response body
request
  .get('/api/status')
  .ok(res => {
    return res.status === 200 &&
           res.body &&
           res.body.success === true;
  })
  .then(res => {
    console.log('Operation successful');
  })
  .catch(err => {
    console.log('Operation failed or invalid response');
  });

// Accept redirects as success
request
  .get('/api/resource')
  .redirects(0)  // don't follow
  .ok(res => res.status >= 200 && res.status < 400)
  .then(res => {
    if (res.status >= 300 && res.status < 400) {
      console.log('Redirect to:', res.header.location);
    }
  });

// Combine with retry logic
request
  .get('/api/eventually-consistent')
  .retry(3)
  .ok(res => res.status === 200 && res.body.ready === true)
  .then(res => {
    console.log('Resource is ready');
  });
```

## Browser-Specific Configuration

### Cross-Domain Credentials

Enable transmission of cookies and authentication credentials with cross-domain requests. Only works in browser environments (no-op in Node.js).

```javascript { .api }
/**
 * Enable cross-domain credentials (browser only)
 * @param enable - True to enable (default), false to disable
 * @returns Request instance for chaining
 */
withCredentials(enable?: boolean): Request;
```

**CORS Requirements:**
For this to work, the server must:
- NOT use wildcard `Access-Control-Allow-Origin: *`
- Set `Access-Control-Allow-Origin` to the specific origin
- Set `Access-Control-Allow-Credentials: true`

**Usage Examples:**

```javascript
// Enable credentials for cross-domain request
request
  .get('https://api.example.com/user')
  .withCredentials()  // enable by default
  .then(res => {
    // Cookies and auth headers will be sent
    console.log(res.body);
  });

// Explicitly enable credentials
request
  .get('https://api.example.com/user')
  .withCredentials(true)
  .end(callback);

// Disable credentials (default behavior)
request
  .get('https://api.example.com/public')
  .withCredentials(false)
  .end(callback);

// Using with authentication
request
  .post('https://api.example.com/login')
  .withCredentials()  // allows Set-Cookie from server
  .send({ username: 'alice', password: 'secret' })
  .then(res => {
    // Cookie is now stored in browser

    // Subsequent requests will include the cookie
    return request
      .get('https://api.example.com/profile')
      .withCredentials();
  });

// Conditional credentials based on URL
const url = 'https://api.example.com/data';
const isSameDomain = new URL(url).origin === window.location.origin;

request
  .get(url)
  .withCredentials(!isSameDomain)  // only for cross-domain
  .end(callback);
```

## Utility Methods

### Converting to JSON

Convert the request configuration to a plain JavaScript object. Useful for debugging, logging, or serializing request state. Note: This method returns a value and cannot be chained.

```javascript { .api }
/**
 * Convert request to plain JavaScript object
 * @returns Object with method, url, data, and headers properties
 */
toJSON(): {
  method: string;
  url: string;
  data: any;
  headers: object;
};
```

**Usage Examples:**

```javascript
// Log request configuration
const req = request
  .post('/api/users')
  .set('Authorization', 'Bearer token')
  .send({ name: 'Alice' });

console.log(req.toJSON());
// {
//   method: 'POST',
//   url: '/api/users',
//   data: { name: 'Alice' },
//   headers: { authorization: 'Bearer token', 'content-type': 'application/json' }
// }

// Serialize request for logging
const requestLog = request
  .get('/api/data')
  .query({ id: 123 })
  .set('X-Request-ID', 'abc')
  .toJSON();

logger.info('Making request:', requestLog);

// Debug request state
const debugInfo = request
  .put('/api/users/1')
  .send({ name: 'Bob' })
  .set('If-Match', 'etag123')
  .toJSON();

console.log('Request debug:', JSON.stringify(debugInfo, null, 2));

// Store request configuration
const config = request
  .post('/api/endpoint')
  .set('X-Custom', 'value')
  .send(data)
  .toJSON();

// Later: recreate similar request
const newRequest = request(config.method, config.url)
  .set(config.headers)
  .send(config.data);
```

### Clearing Timeouts

Clear any previously set timeouts on the request. Useful for dynamically adjusting timeout behavior or cleaning up request state.

```javascript { .api }
/**
 * Clear all timeout timers
 * @returns Request instance for chaining
 */
clearTimeout(): Request;
```

**Usage Examples:**

```javascript
// Remove timeout conditionally
const req = request
  .get('/api/data')
  .timeout(5000);

if (slowConnection) {
  req.clearTimeout();  // remove timeout for slow connections
}

req.end(callback);

// Clear timeout before retry
request
  .get('/api/data')
  .timeout(5000)
  .retry(3)
  .on('retry', () => {
    // Clear old timeout before retry
    request.clearTimeout();
  })
  .end(callback);

// Dynamic timeout adjustment
const req = request
  .get('/api/large-file')
  .timeout({ response: 5000, deadline: 30000 });

req.on('progress', event => {
  if (event.percent > 0.5) {
    // Remove deadline timeout once download is 50% complete
    req.clearTimeout();
  }
});

// Cancel timeout if response starts quickly
const req = request
  .get('/api/endpoint')
  .timeout({ response: 2000, deadline: 10000 })
  .on('response', () => {
    // Server responded quickly, clear deadline
    req.clearTimeout();
  })
  .end(callback);
```

## Chaining Pattern Examples

All configuration methods support chaining, allowing you to build complex requests fluently:

```javascript
// Complete request configuration
const response = await request
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('X-Request-ID', generateRequestId())
  .type('json')
  .accept('json')
  .query({ include: 'profile,settings' })
  .send({
    name: 'Alice',
    email: 'alice@example.com',
    role: 'admin'
  })
  .timeout(5000)
  .retry(2)
  .redirects(5)
  .ok(res => res.status < 500)
  .maxResponseSize(10 * 1024 * 1024);

// Browser file upload with progress
request
  .post('/api/upload')
  .set('X-Upload-ID', uploadId)
  .withCredentials()
  .field('title', 'My Document')
  .field('category', 'reports')
  .attach('file', fileBlob, 'document.pdf')
  .responseType('blob')
  .maxResponseSize(50 * 1024 * 1024)
  .on('progress', event => {
    updateProgressBar(event.percent);
  })
  .then(res => {
    console.log('Upload complete');
  });

// Complex search request
request
  .get('/api/search')
  .query({ q: searchTerm })
  .query({ page: currentPage, limit: 20 })
  .query({
    filters: {
      category: ['tech', 'news'],
      date: { from: '2024-01-01', to: '2024-12-31' }
    }
  })
  .sortQuery()  // consistent URL for caching
  .set('Accept-Language', 'en-US')
  .accept('json')
  .timeout({ response: 3000, deadline: 10000 })
  .retry(2)
  .buffer(true)
  .then(res => {
    displayResults(res.body);
  });

// Conditional configuration
const req = request.get(endpoint);

if (needsAuth) {
  req.set('Authorization', `Bearer ${token}`);
}

if (acceptsCompression) {
  req.set('Accept-Encoding', 'gzip, deflate');
}

if (isLargeResponse) {
  req.maxResponseSize(100 * 1024 * 1024);
  req.buffer(false);
}

await req;
```

## Related Documentation

- [Making HTTP Requests](./making-requests.md) - Creating and sending HTTP requests
- [Response Handling](./response-handling.md) - Working with response data
- [Authentication](./authentication.md) - Setting up authentication
- [Timeouts and Retries](./timeouts-retries.md) - Configuring timeout and retry behavior
- [File Uploads](./file-uploads.md) - Uploading files with multipart/form-data
