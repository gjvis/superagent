# Response

Response object properties and methods for accessing response data, headers, and status information.

## Capabilities

### Response Object

The Response object is returned by successful requests and contains all response data and metadata.

```javascript { .api }
interface Response {
  // Response body data
  body: any;              // Parsed response body (object, string, or Buffer)
  text: string;           // Response body as text
  header: object;         // Response headers (lowercase keys)
  headers: object;        // Alias for header
  type: string;           // Content-Type without parameters

  // Status information
  status: number;         // HTTP status code
  statusCode: number;     // Alias for status
  statusText: string;     // HTTP status text (e.g., "OK", "Not Found")
  statusType: number;     // Status class (1-5)

  // Status boolean flags
  ok: boolean;            // True for 2xx status
  error: Error | false;   // Error object for 4xx/5xx, false otherwise
  info: boolean;          // True for 1xx status
  redirect: boolean;      // True for 3xx status
  clientError: boolean;   // True for 4xx status
  serverError: boolean;   // True for 5xx status

  // Specific status flags
  created: boolean;       // True for 201 status
  accepted: boolean;      // True for 202 status
  noContent: boolean;     // True for 204 status
  badRequest: boolean;    // True for 400 status
  unauthorized: boolean;  // True for 401 status
  notAcceptable: boolean; // True for 406 status
  forbidden: boolean;     // True for 403 status
  notFound: boolean;      // True for 404 status
  unprocessableEntity: boolean; // True for 422 status

  // Additional properties
  links: object;          // Parsed Link header
  redirects: string[];    // Array of redirect URLs followed (Node.js)
  xhr: XMLHttpRequest;    // Underlying XHR object (browser only)
  req: Request;           // Original request object
  files: object;          // Uploaded files in multipart responses (Node.js)
  buffered: boolean;      // Whether response was buffered

  // Methods
  get(field: string): string;  // Get response header (case-insensitive)
  toError(): Error;       // Convert response to Error object
  toJSON(): object;       // Convert response to plain object

  // Streaming methods (Node.js only)
  pause(): Response;      // Pause response stream
  resume(): Response;     // Resume response stream
  destroy(err?: Error): Response;  // Destroy response stream
  setEncoding(encoding: string): Response;  // Set response encoding
}
```

### Response Body

Access parsed response data.

**Usage Examples:**

```javascript
// JSON response
request
  .get('/api/users')
  .then(res => {
    console.log(res.body);  // Parsed JSON object
    console.log(res.text);  // Raw JSON string
  });

// Text response
request
  .get('/api/data.txt')
  .then(res => {
    console.log(res.text);  // Plain text content
    console.log(res.body);  // Same as text for text responses
  });

// Binary response (Node.js)
request
  .get('/api/image.png')
  .then(res => {
    console.log(res.body);  // Buffer
    fs.writeFileSync('image.png', res.body);
  });

// Form-urlencoded response
request
  .get('/api/form-data')
  .then(res => {
    console.log(res.body);  // Parsed object
    // { name: 'John', email: 'john@example.com' }
  });
```

### Response Headers

Access response headers with case-insensitive field names.

```javascript { .api }
/**
 * Get response header value (case-insensitive)
 * @param field - Header field name
 * @returns Header value or undefined
 */
res.get(field: string): string | undefined;
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    // Get specific header (case-insensitive)
    const contentType = res.get('Content-Type');
    const serverHeader = res.get('server');
    const etag = res.get('ETag');

    // Access all headers
    console.log(res.header);
    // { 'content-type': 'application/json', 'content-length': '1234', ... }

    console.log(res.headers);  // Same as res.header

    // Common headers
    const cacheControl = res.get('Cache-Control');
    const lastModified = res.get('Last-Modified');
    const location = res.get('Location');
  });
```

### Content Type

Get parsed content type information.

**Usage Examples:**

```javascript
request
  .get('/api/data')
  .then(res => {
    // Content-Type without parameters
    console.log(res.type);
    // 'application/json' (from 'application/json; charset=utf-8')

    // Full Content-Type header
    console.log(res.get('Content-Type'));
    // 'application/json; charset=utf-8'

    // Content-Type parameters
    // (automatically set as properties on response object)
    console.log(res.charset);  // 'utf-8'
  });
```

### Status Information

Check HTTP status codes and response success/failure.

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    // Status code
    console.log(res.status);      // 200
    console.log(res.statusCode);  // 200 (alias)
    console.log(res.statusText);  // 'OK'

    // Status class (1-5)
    console.log(res.statusType);  // 2 (for 2xx)

    // Check success
    if (res.ok) {
      console.log('Success!');
    }

    // Check status class
    if (res.info) {
      console.log('Informational 1xx');
    }
    if (res.ok) {
      console.log('Success 2xx');
    }
    if (res.redirect) {
      console.log('Redirect 3xx');
    }
    if (res.clientError) {
      console.log('Client error 4xx');
    }
    if (res.serverError) {
      console.log('Server error 5xx');
    }
  })
  .catch(err => {
    // Error response (4xx or 5xx)
    const res = err.response;
    if (res) {
      console.log('Error status:', res.status);
      console.log('Error body:', res.body);
    }
  });
```

### Specific Status Checks

Use boolean flags for common status codes.

**Usage Examples:**

```javascript
request
  .post('/api/users')
  .send({ name: 'John' })
  .then(res => {
    if (res.created) {
      console.log('Resource created (201)');
    }
  });

request
  .delete('/api/users/123')
  .then(res => {
    if (res.noContent) {
      console.log('Successfully deleted (204)');
    }
  });

request
  .get('/api/protected')
  .catch(err => {
    const res = err.response;
    if (res) {
      if (res.unauthorized) {
        console.log('Authentication required (401)');
        // Redirect to login
      } else if (res.forbidden) {
        console.log('Access denied (403)');
      } else if (res.notFound) {
        console.log('Resource not found (404)');
      } else if (res.badRequest) {
        console.log('Invalid request (400)');
        console.log('Validation errors:', res.body);
      } else if (res.unprocessableEntity) {
        console.log('Validation failed (422)');
      }
    }
  });

request
  .put('/api/async-task')
  .send({ data: 'value' })
  .then(res => {
    if (res.accepted) {
      console.log('Task accepted for processing (202)');
    }
  });
```

### Error Response

Access error response data from failed requests.

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    // Success: res.error is false
    console.log(res.error);  // false
  })
  .catch(err => {
    // Failure: err.response contains Response object
    console.error('Error:', err.message);
    console.error('Status:', err.status);

    if (err.response) {
      const res = err.response;

      // Error flag
      console.log(res.error);  // Error object

      // Status information
      console.log('Status:', res.status);
      console.log('Status text:', res.statusText);

      // Error body
      console.log('Error details:', res.body);

      // Error headers
      console.log('Error headers:', res.header);

      // Check error type
      if (res.clientError) {
        console.log('Client error (4xx)');
        // Handle validation errors, authentication, etc.
      }
      if (res.serverError) {
        console.log('Server error (5xx)');
        // Handle server failures, retry, etc.
      }
    } else {
      // Network error (no response)
      console.log('Network error:', err.message);
    }
  });
```

### Link Header

Parse Link header for pagination and related resources.

**Usage Examples:**

```javascript
request
  .get('/api/users?page=2')
  .then(res => {
    // Parsed Link header
    console.log(res.links);
    // {
    //   next: 'https://api.example.com/users?page=3',
    //   prev: 'https://api.example.com/users?page=1',
    //   first: 'https://api.example.com/users?page=1',
    //   last: 'https://api.example.com/users?page=10'
    // }

    // Navigate to next page
    if (res.links.next) {
      return request.get(res.links.next);
    }
  });

// Pagination example
async function getAllUsers() {
  const users = [];
  let url = '/api/users?page=1';

  while (url) {
    const res = await request.get(url);
    users.push(...res.body);
    url = res.links.next;
  }

  return users;
}
```

### Redirect Information

Access redirect history for requests (Node.js only).

**Usage Examples:**

```javascript
// Node.js only
request
  .get('/api/resource')
  .redirects(5)
  .then(res => {
    // Array of redirect URLs followed
    console.log(res.redirects);
    // [
    //   'https://example.com/old-path',
    //   'https://example.com/new-path'
    // ]

    console.log('Redirected', res.redirects.length, 'times');
    console.log('Final URL:', res.redirects[res.redirects.length - 1] || 'No redirects');
  });
```

### XHR Object (Browser)

Access underlying XMLHttpRequest object in browser.

**Usage Examples:**

```javascript
// Browser only
request
  .get('/api/data')
  .then(res => {
    // Access XHR object
    console.log(res.xhr);
    console.log('Ready state:', res.xhr.readyState);
    console.log('Response URL:', res.xhr.responseURL);
  });
```

## Response Processing

### Conditional Processing

```javascript
request
  .get('/api/resource')
  .then(res => {
    if (res.status === 304) {
      console.log('Not modified, using cached version');
      return cachedData;
    }

    if (res.noContent) {
      console.log('No content to process');
      return null;
    }

    return res.body;
  });
```

### Error Recovery

```javascript
async function getDataWithFallback() {
  try {
    const res = await request.get('/api/primary');
    return res.body;
  } catch (err) {
    if (err.response && err.response.notFound) {
      console.log('Resource not found, using default');
      return defaultData;
    }
    throw err;
  }
}
```

### Response Validation

```javascript
request
  .get('/api/users')
  .then(res => {
    // Validate response structure
    if (!Array.isArray(res.body)) {
      throw new Error('Expected array response');
    }

    // Validate content type
    if (res.type !== 'application/json') {
      throw new Error('Expected JSON response');
    }

    // Validate required headers
    if (!res.get('X-Request-ID')) {
      console.warn('Missing request ID header');
    }

    return res.body;
  });
```

### Extract Response Data

```javascript
// Extract specific fields
request
  .get('/api/users')
  .then(res => res.body.map(user => ({
    id: user.id,
    name: user.name
  })));

// Extract metadata
request
  .get('/api/users')
  .then(res => ({
    data: res.body,
    total: parseInt(res.get('X-Total-Count')),
    page: parseInt(res.get('X-Page')),
    hasMore: !!res.links.next
  }));
```

### Response Caching

```javascript
const cache = new Map();

async function getCached(url) {
  // Check cache
  const cached = cache.get(url);
  if (cached) {
    console.log('Using cached response');
    return cached;
  }

  // Fetch and cache
  const res = await request.get(url);

  // Cache based on headers
  const cacheControl = res.get('Cache-Control');
  if (cacheControl && !cacheControl.includes('no-cache')) {
    cache.set(url, res.body);
  }

  return res.body;
}
```

## Content Type Handling

SuperAgent automatically parses responses based on Content-Type:

```javascript
// JSON response (Content-Type: application/json)
request.get('/api/users').then(res => {
  console.log(res.body);  // Parsed object/array
});

// Text response (Content-Type: text/plain)
request.get('/api/data.txt').then(res => {
  console.log(res.text);  // String
  console.log(res.body);  // Same as text
});

// Form-urlencoded (Content-Type: application/x-www-form-urlencoded)
request.get('/api/form').then(res => {
  console.log(res.body);  // Parsed object
});

// Binary response (Content-Type: image/*, application/octet-stream, etc.)
request.get('/api/file.bin').then(res => {
  console.log(res.body);  // Buffer (Node.js) or Blob/ArrayBuffer (browser)
});

// HTML response (Content-Type: text/html)
request.get('/page').then(res => {
  console.log(res.text);  // HTML string
});

// XML response (Content-Type: application/xml)
request.get('/api/data.xml').then(res => {
  console.log(res.text);  // XML string
  // Note: No automatic XML parsing, use custom parser
});
```

## Response Patterns

### API Response Wrapper

```javascript
async function apiRequest(url) {
  try {
    const res = await request.get(url);
    return {
      success: true,
      data: res.body,
      status: res.status
    };
  } catch (err) {
    return {
      success: false,
      error: err.message,
      status: err.status
    };
  }
}
```

### Paginated Response Handler

```javascript
async function fetchAllPages(baseUrl) {
  const allData = [];
  let page = 1;

  while (true) {
    const res = await request
      .get(baseUrl)
      .query({ page, per_page: 100 });

    allData.push(...res.body);

    if (!res.links.next || res.body.length === 0) {
      break;
    }

    page++;
  }

  return allData;
}
```

### Streaming Response (Node.js)

```javascript
const fs = require('fs');

request
  .get('/api/large-file')
  .pipe(fs.createWriteStream('output.dat'))
  .on('finish', () => console.log('Download complete'));
```

## Additional Response Properties

### Request Reference

Access the original request object from the response.

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    // Access original request
    console.log('Request method:', res.req.method);
    console.log('Request URL:', res.req.url);
    console.log('Request headers:', res.req.header);
  });
```

### Multipart Files (Node.js)

Access uploaded files in multipart responses.

**Usage Examples:**

```javascript
// Node.js only
request
  .post('/api/form-handler')
  .then(res => {
    // Multipart responses may include files
    if (res.files) {
      console.log('Uploaded files:', res.files);
    }
  });
```

### Buffered Flag

Check whether the response was buffered.

**Usage Examples:**

```javascript
request
  .get('/api/data')
  .buffer(true)
  .then(res => {
    console.log('Response buffered:', res.buffered);  // true
  });

request
  .get('/api/stream')
  .buffer(false)
  .on('response', res => {
    console.log('Response buffered:', res.buffered);  // false
  })
  .pipe(fs.createWriteStream('output.dat'));
```

## Response Conversion Methods

### Convert to Error

Convert a response to an Error object.

```javascript { .api }
/**
 * Convert response to Error object
 * Useful for converting error responses to throwable errors
 * @returns Error object with status and response properties
 */
res.toError(): Error;
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .ok(res => res.status < 500)  // Don't auto-reject on 4xx
  .then(res => {
    if (res.status >= 400) {
      throw res.toError();
    }
    return res.body;
  })
  .catch(err => {
    console.error('Error:', err.message);
    console.error('Status:', err.status);
    console.error('Response:', err.response);
  });
```

### Convert to JSON

Convert response to plain JavaScript object.

```javascript { .api }
/**
 * Convert response to plain object
 * Returns non-reactive object with all response properties
 * @returns Object with body, headers, status, etc.
 */
res.toJSON(): object;
```

**Usage Examples:**

```javascript
request
  .get('/api/users')
  .then(res => {
    const resObj = res.toJSON();
    console.log(resObj);
    // {
    //   body: [...],
    //   headers: {...},
    //   status: 200,
    //   statusText: 'OK',
    //   ...
    // }

    // Useful for logging or serialization
    fs.writeFileSync('response.json', JSON.stringify(resObj, null, 2));
  });
```

## Response Streaming (Node.js)

When using `.buffer(false)` or `.pipe()`, the response object becomes a readable stream with stream methods and events.

### Stream Control Methods

Control the flow of response data.

```javascript { .api }
/**
 * Pause response stream (Node.js only)
 * Stops 'data' events from being emitted
 * @returns Response for chaining
 */
res.pause(): Response;

/**
 * Resume response stream (Node.js only)
 * Resumes 'data' events after pause
 * @returns Response for chaining
 */
res.resume(): Response;

/**
 * Destroy response stream (Node.js only)
 * Closes stream and releases resources
 * @param err - Optional error to emit
 * @returns Response for chaining
 */
res.destroy(err?: Error): Response;

/**
 * Set response encoding (Node.js only)
 * Sets character encoding for string conversion
 * @param encoding - Encoding name (e.g., 'utf8', 'ascii', 'hex')
 * @returns Response for chaining
 */
res.setEncoding(encoding: string): Response;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Manual stream control
request
  .get('/api/large-data')
  .buffer(false)
  .on('response', res => {
    // Set encoding for text data
    res.setEncoding('utf8');

    // Pause initially
    res.pause();

    setTimeout(() => {
      // Resume after delay
      res.resume();
    }, 1000);

    // Handle data
    res.on('data', chunk => {
      console.log('Received chunk:', chunk.length);
    });

    res.on('end', () => {
      console.log('Stream complete');
    });
  })
  .end();

// Conditional processing
request
  .get('/api/data')
  .buffer(false)
  .on('response', res => {
    const size = parseInt(res.get('Content-Length'));

    if (size > 1000000) {
      // Large file: stream to disk
      res.pipe(fs.createWriteStream('large-file.dat'));
    } else {
      // Small file: process in memory
      let data = '';
      res.setEncoding('utf8');
      res.on('data', chunk => {
        data += chunk;
      });
      res.on('end', () => {
        console.log('Data:', data);
      });
    }
  })
  .end();

// Error handling with destroy
request
  .get('/api/stream')
  .buffer(false)
  .on('response', res => {
    res.on('data', chunk => {
      try {
        processChunk(chunk);
      } catch (err) {
        console.error('Processing error:', err);
        res.destroy(err);
      }
    });

    res.on('error', err => {
      console.error('Stream error:', err);
    });
  })
  .end();
```

### Stream Events

Listen to streaming events when using `.buffer(false)` or manual stream control.

```javascript { .api }
// Stream events (Node.js only)
res.on('data', (chunk: Buffer | string) => void);   // Data chunk received
res.on('end', () => void);                          // Stream ended
res.on('error', (err: Error) => void);              // Stream error
res.on('close', () => void);                        // Stream closed
```

**Usage Examples:**

```javascript
// Manual data accumulation
request
  .get('/api/data')
  .buffer(false)
  .on('response', res => {
    let data = Buffer.alloc(0);

    res.on('data', chunk => {
      console.log('Chunk size:', chunk.length);
      data = Buffer.concat([data, chunk]);
    });

    res.on('end', () => {
      console.log('Total size:', data.length);
      console.log('Data:', data.toString());
    });

    res.on('error', err => {
      console.error('Stream error:', err);
    });

    res.on('close', () => {
      console.log('Stream closed');
    });
  })
  .end();

// Line-by-line processing
const readline = require('readline');

request
  .get('/api/large-text.txt')
  .buffer(false)
  .on('response', res => {
    const rl = readline.createInterface({
      input: res,
      crlfDelay: Infinity
    });

    let lineCount = 0;

    rl.on('line', line => {
      lineCount++;
      console.log(`Line ${lineCount}:`, line);
    });

    rl.on('close', () => {
      console.log('Total lines:', lineCount);
    });
  })
  .end();

// Progress tracking with data events
request
  .get('/api/large-file')
  .buffer(false)
  .on('response', res => {
    const total = parseInt(res.get('Content-Length'));
    let downloaded = 0;

    res.on('data', chunk => {
      downloaded += chunk.length;
      const percent = (downloaded / total * 100).toFixed(2);
      console.log(`Progress: ${percent}% (${downloaded}/${total})`);
    });

    res.on('end', () => {
      console.log('Download complete');
    });

    res.pipe(fs.createWriteStream('download.bin'));
  })
  .end();
```

## Stream vs Buffered Responses

SuperAgent handles responses in two modes:

### Buffered Mode (Default)

```javascript
// Default: response is buffered in memory
request
  .get('/api/data')
  .then(res => {
    console.log(res.body);  // Fully buffered and parsed
    console.log(res.text);  // Complete text
  });
```

### Streaming Mode (Node.js)

```javascript
// Streaming: response is not buffered
request
  .get('/api/data')
  .buffer(false)
  .on('response', res => {
    // res.body and res.text are not available
    // Must use stream events
    res.on('data', chunk => {
      console.log('Chunk:', chunk);
    });
  })
  .end();

// Or use .pipe()
request
  .get('/api/data')
  .pipe(fs.createWriteStream('output.dat'));
// Automatically sets .buffer(false)
```

**When to use streaming:**
- Large files (images, videos, archives)
- Real-time data processing
- Memory-constrained environments
- Progress tracking for downloads

**When to use buffering:**
- Small JSON/text responses
- Need to access parsed `res.body`
- Simpler error handling
- Most API requests
