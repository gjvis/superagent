# Response Handling

Access and process HTTP response data, headers, status codes, and parsed body content.

## Capabilities

### Response Properties

Access response data and metadata.

```javascript { .api }
/**
 * Response object properties
 */
interface Response {
  // Status
  status: number;           // HTTP status code (e.g., 200, 404)
  statusCode: number;       // Alias for status
  statusType: number;       // Status category: 1-5 for 1xx-5xx
  statusText: string;       // Status text (Browser only, e.g., "OK", "Not Found")

  // Headers
  header: object;           // Response headers (lowercase keys)
  headers: object;          // Alias for header
  type: string;             // Content-Type without parameters
  charset: string;          // Character set from Content-Type
  links: object;            // Parsed Link header

  // Body
  body: any;                // Parsed response body
  text: string;             // Response as text
  files: object;            // Uploaded files for multipart responses (Node.js only)

  // Redirects
  redirects: string[];      // Array of redirect URLs (Node.js only)
  buffered: boolean;        // Whether response was buffered (Node.js only)

  // Request reference
  req: Request;             // Original request object
  request: Request;         // Alias for req (Node.js only)
  res: any;                 // Native response object (Node.js only)
  xhr: XMLHttpRequest;      // XMLHttpRequest object (Browser only)
}
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Access status code
request
  .get('/api/users')
  .end((err, res) => {
    console.log('Status:', res.status);        // 200
    console.log('Status code:', res.statusCode); // 200
    console.log('Status type:', res.statusType); // 2 (for 2xx)
  });

// Access headers
request
  .get('/api/users')
  .end((err, res) => {
    console.log('All headers:', res.headers);
    console.log('Content type:', res.type);    // 'application/json'
    console.log('Charset:', res.charset);      // 'utf-8'
  });

// Access body
request
  .get('/api/users')
  .end((err, res) => {
    console.log('Parsed body:', res.body);     // { users: [...] }
    console.log('Raw text:', res.text);        // '{"users":[...]}'
  });

// Access redirect history
request
  .get('/api/redirect')
  .end((err, res) => {
    console.log('Redirects:', res.redirects);  // ['/url1', '/url2']
  });

// Browser: Access XMLHttpRequest
request
  .get('/api/data')
  .end((err, res) => {
    console.log('XHR:', res.xhr);              // XMLHttpRequest object
  });
```

### Status Flags

Boolean flags for checking response status categories.

```javascript { .api }
/**
 * Response status flags
 */
interface Response {
  // Category flags
  ok: boolean;              // 2xx status (successful)
  error: Error | false;     // Error object for 4xx/5xx, false otherwise
  info: boolean;            // 1xx status (informational)
  redirect: boolean;        // 3xx status (redirect)
  clientError: boolean;     // 4xx status (client error)
  serverError: boolean;     // 5xx status (server error)

  // Specific status flags
  created: boolean;         // 201 Created
  accepted: boolean;        // 202 Accepted
  noContent: boolean;       // 204 No Content
  badRequest: boolean;      // 400 Bad Request
  unauthorized: boolean;    // 401 Unauthorized
  forbidden: boolean;       // 403 Forbidden
  notFound: boolean;        // 404 Not Found
  notAcceptable: boolean;   // 406 Not Acceptable
  unprocessableEntity: boolean; // 422 Unprocessable Entity
}
```

**Usage Examples:**

```javascript
// Check if response is successful
request
  .get('/api/users')
  .end((err, res) => {
    if (res.ok) {
      console.log('Success!');
    }
  });

// Check for errors
request
  .get('/api/users')
  .end((err, res) => {
    if (res.error) {
      console.log('Error occurred:', res.error.message);

      if (res.clientError) {
        console.log('Client error (4xx)');
      }

      if (res.serverError) {
        console.log('Server error (5xx)');
      }
    }
  });

// Check specific status codes
request
  .get('/api/users/123')
  .end((err, res) => {
    if (res.notFound) {
      console.log('User not found');
    }

    if (res.unauthorized) {
      console.log('Authentication required');
    }

    if (res.forbidden) {
      console.log('Access denied');
    }
  });

// Check for redirects
request
  .get('/api/resource')
  .end((err, res) => {
    if (res.redirect) {
      console.log('Response was redirected');
      console.log('Redirect chain:', res.redirects);
    }
  });

// Check for specific success statuses
request
  .post('/api/users')
  .send({ name: 'John' })
  .end((err, res) => {
    if (res.created) {
      console.log('User created successfully');
    }
  });

request
  .delete('/api/users/123')
  .end((err, res) => {
    if (res.noContent) {
      console.log('User deleted (no content returned)');
    }
  });

// Informational responses
request
  .get('/api/processing')
  .end((err, res) => {
    if (res.info) {
      console.log('Informational response (1xx)');
    }
  });
```

### Get Response Headers

Retrieve individual response header values.

```javascript { .api }
/**
 * Get response header value (case-insensitive)
 * @param {string} field - Header name
 * @returns {string} Header value
 */
Response.prototype.get = function(field);

/**
 * Convert response to Error object
 * @returns {Error} Error object with response information
 */
Response.prototype.toError = function();
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    // Get specific headers (case-insensitive)
    console.log('Content-Type:', res.get('Content-Type'));
    console.log('content-type:', res.get('content-type'));  // Same as above
    console.log('ETag:', res.get('ETag'));
    console.log('Cache-Control:', res.get('Cache-Control'));
    console.log('Last-Modified:', res.get('Last-Modified'));

    // Get custom headers
    console.log('API-Version:', res.get('API-Version'));
    console.log('Rate-Limit:', res.get('X-RateLimit-Remaining'));
  });

// Check if header exists
request
  .get('/api/data')
  .end((err, res) => {
    const etag = res.get('ETag');
    if (etag) {
      console.log('Response has ETag:', etag);
    }
  });
```

### Parse Response Body

SuperAgent automatically parses response bodies based on Content-Type.

**Usage Examples:**

```javascript
// JSON responses (Content-Type: application/json)
request
  .get('/api/users')
  .end((err, res) => {
    console.log(res.body);  // Parsed JavaScript object
    // { users: [{ id: 1, name: 'John' }, ...] }
  });

// Form data (Content-Type: application/x-www-form-urlencoded)
request
  .get('/api/form-data')
  .end((err, res) => {
    console.log(res.body);  // Parsed object
    // { name: 'John', email: 'john@example.com' }
  });

// Text responses (Content-Type: text/plain, text/html, etc.)
request
  .get('/api/text')
  .end((err, res) => {
    console.log(res.text);  // Raw text string
    console.log(res.body);  // Also contains text for text/* types
  });

// Binary responses (images, PDFs, etc.)
request
  .get('/api/image.png')
  .responseType('blob')  // Browser
  .end((err, res) => {
    console.log(res.body);  // Blob (browser) or Buffer (Node.js)
  });

// XML responses (requires custom parser)
request
  .get('/api/data.xml')
  .end((err, res) => {
    console.log(res.text);  // Raw XML string
    // Parse with XML library if needed
  });
```

### Handle Parse Errors

Handle errors that occur during response parsing.

**Usage Examples:**

```javascript
// Handle JSON parse errors
request
  .get('/api/invalid-json')
  .end((err, res) => {
    if (err && err.parse) {
      console.error('Parse error:', err.message);
      console.log('Raw response:', err.rawResponse);  // Browser only
      console.log('Raw text:', res.text);
    }
  });

// Use custom parser for specific content types
request
  .get('/api/data')
  .parse((res, callback) => {
    // Custom parsing logic
    let data = '';
    res.on('data', chunk => {
      data += chunk;
    });
    res.on('end', () => {
      try {
        const parsed = customParser(data);
        callback(null, parsed);
      } catch (err) {
        callback(err);
      }
    });
  })
  .end((err, res) => {
    console.log(res.body);  // Custom parsed data
  });
```

### Response Utilities

Utility methods for working with responses.

```javascript { .api }
/**
 * Convert response to Error object
 * @returns {Error} Error with status, method, and url properties
 */
Response.prototype.toError = function();

/**
 * Convert response to JSON representation
 * @returns {object} JSON object
 */
Response.prototype.toJSON = function();
```

**Usage Examples:**

```javascript
// Convert response to error
request
  .get('/api/users')
  .end((err, res) => {
    if (!res.ok) {
      const error = res.toError();
      console.log('Error:', error.message);
      console.log('Status:', error.status);
      console.log('Method:', error.method);
      console.log('URL:', error.url);
    }
  });

// Convert response to JSON
request
  .get('/api/users')
  .end((err, res) => {
    const json = res.toJSON();
    console.log(json);
    // {
    //   status: 200,
    //   header: { ... },
    //   body: { ... },
    //   ...
    // }
  });
```

### Stream Response (Node.js)

Control response stream flow in Node.js.

```javascript { .api }
/**
 * Pause response stream
 * @returns {Response} Response instance
 */
Response.prototype.pause = function();

/**
 * Resume response stream
 * @returns {Response} Response instance
 */
Response.prototype.resume = function();

/**
 * Destroy response stream
 * @param {Error} [err] - Optional error
 * @returns {Response} Response instance
 */
Response.prototype.destroy = function(err);
```

**Usage Examples:**

```javascript
// Pause and resume response stream
request
  .get('/api/large-data')
  .buffer(false)
  .end((err, res) => {
    // Pause reading from stream
    res.pause();

    // Process data in chunks
    res.on('data', chunk => {
      console.log('Received chunk:', chunk.length);

      // Pause if buffer is full
      if (needsBackpressure()) {
        res.pause();

        // Resume when ready
        setTimeout(() => res.resume(), 1000);
      }
    });

    res.on('end', () => {
      console.log('Stream complete');
    });

    // Resume reading
    res.resume();
  });

// Destroy stream on error
request
  .get('/api/stream')
  .buffer(false)
  .end((err, res) => {
    res.on('data', chunk => {
      try {
        processChunk(chunk);
      } catch (err) {
        // Destroy stream on processing error
        res.destroy(err);
      }
    });
  });
```

### Link Header Parsing

Access parsed Link header for pagination and related resources.

**Usage Examples:**

```javascript
// Access link header
request
  .get('/api/users?page=2')
  .end((err, res) => {
    console.log(res.links);
    // {
    //   next: '/api/users?page=3',
    //   prev: '/api/users?page=1',
    //   first: '/api/users?page=1',
    //   last: '/api/users?page=10'
    // }

    // Navigate to next page
    if (res.links.next) {
      request.get(res.links.next).end(callback);
    }
  });
```

### Multipart Response Files

Access uploaded files in multipart responses.

**Usage Examples:**

```javascript
// Server responds with multipart data containing files
request
  .get('/api/download-batch')
  .end((err, res) => {
    console.log(res.files);
    // {
    //   file1: { name: 'doc.pdf', size: 12345, ... },
    //   file2: { name: 'image.jpg', size: 67890, ... }
    // }
  });
```

### Complete Response Handling Example

Comprehensive example showing all response handling features.

**Usage Examples:**

```javascript
const request = require('superagent');

async function fetchUserData() {
  try {
    const res = await request
      .get('/api/users/123')
      .accept('json');

    // Check status
    if (res.ok) {
      console.log('Success! Status:', res.status);

      // Access headers
      console.log('Content-Type:', res.get('Content-Type'));
      console.log('ETag:', res.get('ETag'));

      // Access body
      const user = res.body;
      console.log('User:', user.name);
      console.log('Email:', user.email);

      // Check for specific status
      if (res.created) {
        console.log('User was created');
      }

      return user;
    }

  } catch (err) {
    // Error is set for non-2xx responses and network errors
    console.error('Request failed');

    if (err.response) {
      // Response was received but status was not 2xx
      const res = err.response;

      console.log('Status:', res.status);
      console.log('Body:', res.body);

      if (res.notFound) {
        console.error('User not found');
      } else if (res.unauthorized) {
        console.error('Authentication required');
      } else if (res.forbidden) {
        console.error('Access denied');
      } else if (res.serverError) {
        console.error('Server error occurred');
      }

    } else {
      // Network error or request setup error
      console.error('Network error:', err.message);
    }

    throw err;
  }
}

// Callback-based version
function fetchUserDataCallback(userId, callback) {
  request
    .get(`/api/users/${userId}`)
    .end((err, res) => {
      if (err) {
        // Handle error
        if (err.response) {
          console.log('Status:', err.response.status);
          if (err.response.notFound) {
            return callback(new Error('User not found'));
          }
        }
        return callback(err);
      }

      // Success
      console.log('Headers:', res.headers);
      console.log('Body:', res.body);
      callback(null, res.body);
    });
}
```

### Important Notes

- **Automatic Parsing**: Response body is automatically parsed based on Content-Type header. JSON, form data, and text are parsed automatically.
- **res.body vs res.text**: Use `res.body` for parsed data (objects, arrays) and `res.text` for raw string content.
- **Status Flags**: Use boolean flags like `res.ok`, `res.notFound`, etc. for cleaner status checking instead of comparing `res.status` values.
- **Error Responses**: For non-2xx responses, both `err` and `err.response` are available. The response object is attached to the error.
- **Headers**: Header names are case-insensitive when using `res.get()`, but stored as lowercase in `res.headers`.
- **Browser vs Node.js**: Some properties like `statusText` and `xhr` are browser-only. Stream methods are Node.js only.
