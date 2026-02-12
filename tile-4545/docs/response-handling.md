# Response Handling

Parse and process HTTP responses with automatic content-type detection and status helpers.

## Capabilities

### Response Object

The Response object is passed to callbacks and promise handlers after a request completes.

```javascript { .api }
interface Response {
  // HTTP status
  status: number;
  statusCode: number; // alias for status
  statusType: number; // 1, 2, 3, 4, or 5

  // Status category booleans
  info: boolean; // 1xx
  ok: boolean; // 2xx
  redirect: boolean; // 3xx
  clientError: boolean; // 4xx
  serverError: boolean; // 5xx
  error: Error | false; // Error object if request failed

  // Specific status code helpers
  created: boolean; // 201
  accepted: boolean; // 202
  noContent: boolean; // 204
  badRequest: boolean; // 400
  unauthorized: boolean; // 401
  forbidden: boolean; // 403
  notFound: boolean; // 404
  notAcceptable: boolean; // 406
  unprocessableEntity: boolean; // 422

  // Response content
  header: object; // response headers
  headers: object; // alias for header
  body: any; // parsed response body
  text: string; // raw response text (Node.js)
  type: string; // content-type without parameters
  charset: string; // charset from Content-Type header
  links: object; // parsed Link header

  // Node.js specific
  files: any; // uploaded files in multipart response
  buffered: boolean; // whether response was buffered

  // Methods
  get(field: string): string; // get header value
  toError(): Error; // convert to Error object
  toJSON(): object; // serialize to plain object
}
```

### Status Code Checking

Check response status using helper properties.

**Usage Examples:**

```javascript
const request = require('superagent');

request
  .get('/api/users')
  .end((err, res) => {
    // Check for 2xx success
    if (res.ok) {
      console.log('Success:', res.body);
    }

    // Check specific status codes
    if (res.created) {
      console.log('Resource created (201)');
    }

    if (res.notFound) {
      console.log('Resource not found (404)');
    }

    if (res.unauthorized) {
      console.log('Authentication required (401)');
    }

    // Check error categories
    if (res.clientError) {
      console.log('Client error (4xx):', res.status);
    }

    if (res.serverError) {
      console.log('Server error (5xx):', res.status);
    }

    // Check status type (1-5)
    console.log('Status type:', res.statusType);
    // 1 = informational, 2 = success, 3 = redirection,
    // 4 = client error, 5 = server error

    // Direct status code
    console.log('Status code:', res.status); // or res.statusCode
  });
```

### Error Handling

Handle errors from failed requests.

```javascript { .api }
interface SuperAgentError extends Error {
  // HTTP status code (if response received)
  status: number;

  // Response object (if available)
  response: Response;

  // Retries remaining (if retry was configured)
  retries: number;

  // Timeout duration (if timeout error)
  timeout: number;

  // Error code (e.g., 'ECONNRESET', 'ETIMEDOUT', 'EADDRINFO', 'ECONNABORTED')
  code: string;

  // HTTP method
  method: string;

  // Request URL
  url: string;

  // CORS error flag (Browser only)
  crossDomain: boolean;

  // Body parsing error flag
  parse: boolean;

  // Original parse error (if parse failed)
  original: Error;

  // Raw response data (if parse failed)
  rawResponse: string | Buffer;
}
```

**Usage Examples:**

```javascript
// Callback style
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error('Request failed:', err.message);
      console.error('Status:', err.status);
      console.error('Response:', err.response);
      return;
    }
    console.log('Success:', res.body);
  });

// Promise style
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    if (err.response) {
      // Server responded with error status
      console.error('Server error:', err.response.status);
      console.error('Body:', err.response.body);
    } else {
      // Network error or other issue
      console.error('Request error:', err.message);
    }
  });

// Async/await style
async function getUsers() {
  try {
    const res = await request.get('/api/users');
    return res.body;
  } catch (err) {
    if (err.status === 404) {
      console.log('Not found');
    } else if (err.status >= 500) {
      console.log('Server error');
    } else {
      console.log('Other error:', err.message);
    }
    throw err;
  }
}

// Detailed error handling
request
  .get('/api/data')
  .retry(3)
  .timeout(5000)
  .end((err, res) => {
    if (err) {
      // Check error type
      if (err.timeout) {
        console.error('Request timed out after', err.timeout, 'ms');
      } else if (err.code === 'ECONNRESET') {
        console.error('Connection reset by peer');
      } else if (err.code === 'ETIMEDOUT') {
        console.error('Connection timed out');
      } else if (err.code === 'ENOTFOUND') {
        console.error('DNS lookup failed');
      } else if (err.code === 'ECONNABORTED') {
        console.error('Request aborted');
      } else if (err.parse) {
        console.error('Failed to parse response:', err.original);
        console.error('Raw response:', err.rawResponse);
      } else if (err.crossDomain) {
        console.error('CORS error');
      }

      // Check retry info
      if (err.retries !== undefined) {
        console.error('Retries remaining:', err.retries);
      }

      // Request details
      console.error('Method:', err.method);
      console.error('URL:', err.url);
    }
  });
```

### Response Body Parsing

SuperAgent automatically parses response bodies based on Content-Type.

```javascript { .api }
// Automatic parsing based on Content-Type:
// - application/json → parsed JSON object
// - application/x-www-form-urlencoded → parsed object
// - text/* → string in .text property
// - image/*, application/pdf → Buffer (Node.js) or Blob (Browser)
```

**Usage Examples:**

```javascript
// JSON response (application/json)
request
  .get('/api/users')
  .end((err, res) => {
    console.log(res.body); // parsed JSON object
    console.log(res.type); // 'application/json'
  });

// Text response
request
  .get('/api/status')
  .end((err, res) => {
    console.log(res.text); // raw text
    console.log(res.type); // e.g., 'text/plain'
  });

// Form data response
request
  .get('/api/form')
  .end((err, res) => {
    console.log(res.body); // parsed form data as object
    console.log(res.type); // 'application/x-www-form-urlencoded'
  });

// Binary response (Node.js)
request
  .get('/api/file.pdf')
  .end((err, res) => {
    console.log(res.body); // Buffer
    console.log(res.type); // 'application/pdf'
    fs.writeFileSync('file.pdf', res.body);
  });
```

### Response Headers

Access response headers using the `header` object or `get()` method.

```javascript { .api }
/**
 * Get response header value (case-insensitive)
 * @param field - Header name
 * @returns Header value or undefined
 */
get(field: string): string | undefined;
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    // Access headers object
    console.log(res.header);
    console.log(res.headers); // alias

    // Get specific header (case-insensitive)
    console.log(res.get('Content-Type'));
    console.log(res.get('content-type')); // same result

    // Common headers
    console.log(res.get('content-length'));
    console.log(res.get('last-modified'));
    console.log(res.get('etag'));
    console.log(res.get('cache-control'));

    // Custom headers
    console.log(res.get('X-Rate-Limit-Remaining'));
  });
```

### Response Type and Charset

Access content type and character encoding information.

**Usage Examples:**

```javascript
request
  .get('/api/data')
  .end((err, res) => {
    // Full Content-Type header
    console.log(res.get('content-type'));
    // e.g., 'application/json; charset=utf-8'

    // Type without parameters
    console.log(res.type);
    // e.g., 'application/json'

    // Charset
    console.log(res.charset);
    // e.g., 'utf-8'
  });
```

### Link Header Parsing

Parse HTTP Link headers into a convenient object.

**Usage Examples:**

```javascript
request
  .get('/api/users?page=2')
  .end((err, res) => {
    // Link header example:
    // Link: <https://api.example.com/users?page=1>; rel="prev",
    //       <https://api.example.com/users?page=3>; rel="next"

    console.log(res.links);
    // {
    //   prev: 'https://api.example.com/users?page=1',
    //   next: 'https://api.example.com/users?page=3'
    // }

    // Use parsed links
    if (res.links.next) {
      request.get(res.links.next).end((err, nextRes) => {
        // Load next page
      });
    }
  });
```

### Custom Response Parsing

Override automatic parsing with custom parser.

```javascript { .api }
/**
 * Set custom response parser
 * @param fn - Parser function that receives response and callback
 * @returns Request instance for chaining
 */
parse(fn: (res: Response, callback: (err: Error, body: any) => void) => void): Request;
```

**Usage Examples:**

```javascript
// Custom XML parser
request
  .get('/api/data.xml')
  .parse((res, callback) => {
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      try {
        // Parse XML to JSON (example)
        const parsed = parseXML(data);
        callback(null, parsed);
      } catch (err) {
        callback(err);
      }
    });
  })
  .end((err, res) => {
    console.log(res.body); // parsed XML data
  });
```

### Response Type for Binary Data

Specify how binary responses should be handled (Browser only).

```javascript { .api }
/**
 * Set binary response type (Browser only)
 * @param type - Response type ('blob' or 'arraybuffer')
 * @returns Request instance for chaining
 */
responseType(type: 'blob' | 'arraybuffer'): Request;
```

**Usage Examples (Browser):**

```javascript
// Get response as Blob
request
  .get('/api/image.png')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);
    // Use blob URL for image src, etc.
  });

// Get response as ArrayBuffer
request
  .get('/api/data.bin')
  .responseType('arraybuffer')
  .end((err, res) => {
    const buffer = res.body; // ArrayBuffer
    const view = new DataView(buffer);
    // Process binary data
  });
```
