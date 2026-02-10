# Browser Specific Features

Browser-only capabilities including CORS credentials, XHR access, and browser-specific serialization.

## Capabilities

### CORS Credentials

Enable sending cookies and authentication credentials with cross-origin requests.

```javascript { .api }
/**
 * Enable or disable credentials for cross-origin requests
 * @param enable - True to enable credentials (default if no argument)
 * @returns Request instance for chaining
 */
Request.prototype.withCredentials(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable credentials for CORS requests
request.get('https://api.example.com/users')
  .withCredentials()
  .then(res => console.log(res.body));

// Explicitly enable
request.get('https://api.example.com/users')
  .withCredentials(true)
  .then(res => console.log(res.body));

// Disable credentials
request.get('https://api.example.com/users')
  .withCredentials(false)
  .then(res => console.log(res.body));

// Required for authenticated cross-origin requests
request.get('https://api.example.com/protected')
  .withCredentials() // Sends cookies with request
  .then(res => console.log('Protected data:', res.body))
  .catch(err => console.error('Auth failed:', err));
```

**Important CORS Requirements:**

For `.withCredentials()` to work:
1. Server must respond with `Access-Control-Allow-Origin` matching the origin (not `*`)
2. Server must include `Access-Control-Allow-Credentials: true` header
3. Server must include appropriate `Access-Control-Allow-Headers` if custom headers are used

### XMLHttpRequest Access

Get the underlying XMLHttpRequest object (browser only).

```javascript { .api }
/**
 * Get a new XMLHttpRequest instance
 * @returns XMLHttpRequest instance
 */
function request.getXHR(): XMLHttpRequest;
```

**Usage Examples:**

```javascript
// Get XHR instance
const xhr = request.getXHR();
console.log('XHR:', xhr);

// Check XHR support
if (request.getXHR) {
  console.log('XMLHttpRequest is available');
}

// Manual XHR usage (advanced)
const xhr = request.getXHR();
xhr.open('GET', 'https://api.example.com/users');
xhr.onload = () => {
  console.log('Response:', xhr.responseText);
};
xhr.send();
```

### Form Serialization

Serialize objects to form-urlencoded format.

```javascript { .api }
/**
 * Serialize object to query string (browser)
 * @param obj - Object to serialize
 * @returns URL-encoded string
 */
function request.serializeObject(obj: object): string;
```

**Usage Examples:**

```javascript
// Serialize object
const params = { name: 'John', age: 30, tags: ['a', 'b'] };
const query = request.serializeObject(params);
console.log(query);
// 'name=John&age=30&tags=a&tags=b'

// Use in query string
const url = 'https://api.example.com/users?' + request.serializeObject({
  page: 1,
  limit: 10,
  sort: 'name'
});

// Nested objects
const nested = { filter: { status: 'active', role: 'admin' } };
const serialized = request.serializeObject(nested);
console.log(serialized);
// 'filter[status]=active&filter[role]=admin'
```

### String Parsing

Parse URL-encoded strings to objects.

```javascript { .api }
/**
 * Parse URL-encoded string to object (browser)
 * @param str - URL-encoded string
 * @returns Parsed object
 */
function request.parseString(str: string): object;
```

**Usage Examples:**

```javascript
// Parse query string
const str = 'name=John&age=30&active=true';
const obj = request.parseString(str);
console.log(obj);
// { name: 'John', age: '30', active: 'true' }

// Parse from URL
const url = new URL('https://api.example.com/users?page=1&limit=10');
const params = request.parseString(url.search.substring(1));
console.log(params);
// { page: '1', limit: '10' }

// Empty values
const str2 = 'key1=&key2=value';
const obj2 = request.parseString(str2);
console.log(obj2);
// { key1: '', key2: 'value' }
```

### File Input Upload

Upload files from HTML file input elements.

**Usage Examples:**

```javascript
// Single file upload
const fileInput = document.querySelector('input[type="file"]');
const file = fileInput.files[0];

request.post('https://api.example.com/upload')
  .attach('file', file)
  .then(res => {
    console.log('Upload successful:', res.body);
  })
  .catch(err => {
    console.error('Upload failed:', err.message);
  });

// Multiple files
const fileInput = document.querySelector('input[type="file"][multiple]');

Array.from(fileInput.files).forEach(file => {
  request.post('https://api.example.com/upload')
    .attach('photos', file)
    .then(res => console.log('Uploaded:', file.name));
});

// With progress tracking
const file = fileInput.files[0];
const progressBar = document.querySelector('.progress-bar');

request.post('https://api.example.com/upload')
  .attach('file', file)
  .on('progress', event => {
    if (event.direction === 'upload') {
      progressBar.style.width = event.percent + '%';
    }
  })
  .then(res => console.log('Complete'))
  .catch(err => console.error('Failed'));
```

### Blob and ArrayBuffer

Work with binary data in the browser.

**Usage Examples:**

```javascript
// Download as Blob
request.get('https://api.example.com/image.png')
  .responseType('blob')
  .then(res => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);

    // Display image
    const img = document.createElement('img');
    img.src = url;
    document.body.appendChild(img);

    // Or download file
    const a = document.createElement('a');
    a.href = url;
    a.download = 'image.png';
    a.click();
  });

// Download as ArrayBuffer
request.get('https://api.example.com/data.bin')
  .responseType('arraybuffer')
  .then(res => {
    const buffer = res.body;
    const view = new Uint8Array(buffer);
    console.log('First byte:', view[0]);
  });

// Upload Blob
fetch('https://example.com/image.jpg')
  .then(res => res.blob())
  .then(blob => {
    request.post('https://api.example.com/upload')
      .attach('image', blob, 'downloaded.jpg')
      .then(res => console.log('Uploaded'));
  });

// Create and upload Blob
const data = new Uint8Array([0, 1, 2, 3, 4]);
const blob = new Blob([data], { type: 'application/octet-stream' });

request.post('https://api.example.com/upload')
  .attach('data', blob, 'binary.dat')
  .then(res => console.log('Uploaded'));
```

### Browser Storage Integration

Integrate with localStorage and sessionStorage.

**Usage Examples:**

```javascript
// Save response to localStorage
request.get('https://api.example.com/config')
  .then(res => {
    localStorage.setItem('config', JSON.stringify(res.body));
    console.log('Config saved to localStorage');
  });

// Load from localStorage and sync with server
const cachedConfig = JSON.parse(localStorage.getItem('config') || '{}');

request.get('https://api.example.com/config')
  .set('If-None-Match', cachedConfig.etag)
  .then(res => {
    if (res.status === 304) {
      console.log('Using cached config');
      return cachedConfig;
    }
    localStorage.setItem('config', JSON.stringify(res.body));
    return res.body;
  });

// Session-based caching
async function getCachedOrFetch(url, key) {
  const cached = sessionStorage.getItem(key);
  if (cached) {
    return JSON.parse(cached);
  }

  const res = await request.get(url);
  sessionStorage.setItem(key, JSON.stringify(res.body));
  return res.body;
}

// Usage
const users = await getCachedOrFetch('https://api.example.com/users', 'users');
```

## Browser Patterns

### File Upload with Preview

```javascript
function uploadWithPreview(file) {
  // Show preview
  const reader = new FileReader();
  reader.onload = (e) => {
    const preview = document.querySelector('#preview');
    preview.src = e.target.result;
  };
  reader.readAsDataURL(file);

  // Upload file
  return request.post('https://api.example.com/upload')
    .attach('file', file)
    .on('progress', event => {
      if (event.direction === 'upload') {
        document.querySelector('#progress').textContent =
          Math.round(event.percent) + '%';
      }
    })
    .then(res => {
      console.log('Upload complete:', res.body);
      return res.body;
    });
}

// Usage
const fileInput = document.querySelector('input[type="file"]');
fileInput.addEventListener('change', (e) => {
  const file = e.target.files[0];
  if (file) {
    uploadWithPreview(file);
  }
});
```

### Drag and Drop Upload

```javascript
const dropZone = document.querySelector('#drop-zone');

dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('drag-over');
});

dropZone.addEventListener('drop', (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');

  const files = Array.from(e.dataTransfer.files);

  files.forEach(file => {
    request.post('https://api.example.com/upload')
      .attach('file', file)
      .then(res => {
        console.log('Uploaded:', file.name);
      })
      .catch(err => {
        console.error('Failed to upload:', file.name, err);
      });
  });
});
```

### Image Download and Display

```javascript
async function downloadAndDisplayImage(url, selector) {
  const res = await request.get(url)
    .responseType('blob');

  const blob = res.body;
  const objectUrl = URL.createObjectURL(blob);

  const img = document.querySelector(selector);
  img.src = objectUrl;

  // Clean up object URL when image loads
  img.onload = () => {
    URL.revokeObjectURL(objectUrl);
  };
}

// Usage
downloadAndDisplayImage('https://api.example.com/image.png', '#my-image');
```

### Authenticated CORS Request

```javascript
async function authenticatedRequest(url, token) {
  try {
    const res = await request.get(url)
      .set('Authorization', `Bearer ${token}`)
      .withCredentials() // Send cookies
      .then(res => res.body);

    return res;
  } catch (err) {
    if (err.status === 401) {
      console.error('Authentication failed');
      // Redirect to login
      window.location = '/login';
    }
    throw err;
  }
}

// Usage
const data = await authenticatedRequest(
  'https://api.example.com/protected',
  'user-token-123'
);
```

### Cancellable File Download

```javascript
let downloadRequest = null;

function startDownload(url, filename) {
  downloadRequest = request.get(url)
    .responseType('blob')
    .on('progress', event => {
      if (event.direction === 'download') {
        const progress = Math.round(event.percent);
        document.querySelector('#progress').textContent = progress + '%';
      }
    })
    .then(res => {
      const blob = res.body;
      const url = URL.createObjectURL(blob);

      const a = document.createElement('a');
      a.href = url;
      a.download = filename;
      a.click();

      URL.revokeObjectURL(url);
      downloadRequest = null;
    })
    .catch(err => {
      if (err.message === 'Aborted') {
        console.log('Download cancelled');
      } else {
        console.error('Download failed:', err);
      }
      downloadRequest = null;
    });
}

function cancelDownload() {
  if (downloadRequest) {
    downloadRequest.abort();
  }
}

// Usage
document.querySelector('#download-btn').addEventListener('click', () => {
  startDownload('https://api.example.com/file.pdf', 'document.pdf');
});

document.querySelector('#cancel-btn').addEventListener('click', () => {
  cancelDownload();
});
```

## Important Notes

### CORS Limitations

- `.withCredentials()` requires proper CORS headers from server
- Cannot use wildcard (`*`) for `Access-Control-Allow-Origin` with credentials
- Pre-flight OPTIONS requests may be required for custom headers
- Some headers (e.g., `Cookie`, `Set-Cookie`) are restricted by browsers

### Browser Restrictions

- File access is limited to user-selected files (security)
- Cannot read arbitrary file paths from disk
- Large file downloads may cause memory issues (use streaming with caution)
- Same-origin policy applies unless CORS is properly configured

### Browser Compatibility

- Modern browsers: Chrome, Firefox, Safari, Edge (latest versions)
- IE10+: Limited support (requires polyfills for some features)
- IE9: Requires polyfills for `Promise`, `FormData`, etc.
- Mobile browsers: iOS Safari, Android Chrome (latest versions)

### Performance Considerations

- Binary data (Blob, ArrayBuffer) is more memory-efficient than Base64
- Object URLs should be revoked after use to free memory
- Consider chunked uploads for large files
- Use compression for large JSON payloads
