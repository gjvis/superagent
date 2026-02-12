# Browser Specific Features

Browser-only features including CORS credentials, XHR access, and client-side utilities.

## Capabilities

### CORS Credentials

Enable sending credentials (cookies, authorization headers) with cross-origin requests.

```javascript { .api }
interface Request {
  /**
   * Enable cross-origin credentials
   * @param enable - Enable/disable credentials (default: true)
   * @returns Request instance for chaining
   */
  withCredentials(enable?: boolean): Request;
}
```

**Usage Examples:**

```javascript
// Enable credentials for cross-origin request
request
  .get('https://api.example.com/data')
  .withCredentials()
  .end((err, res) => {
    // Cookies and auth headers are sent
    console.log(res.body);
  });

// Explicitly enable
request
  .get('https://api.example.com/data')
  .withCredentials(true)
  .end((err, res) => {
    console.log(res.body);
  });

// Disable credentials
request
  .get('https://api.example.com/data')
  .withCredentials(false)
  .end((err, res) => {
    console.log(res.body);
  });

// Required for cross-domain cookie sharing
request
  .post('https://auth.example.com/login')
  .withCredentials()
  .send({ username: 'user', password: 'pass' })
  .end((err, res) => {
    // Session cookie is saved
  });

request
  .get('https://api.example.com/profile')
  .withCredentials()
  .end((err, res) => {
    // Session cookie is sent automatically
    console.log(res.body);
  });
```

**Important Notes:**

- Server must respond with `Access-Control-Allow-Credentials: true`
- Server must specify exact origin in `Access-Control-Allow-Origin` (cannot use `*`)
- Only works for cross-origin requests (same-origin requests always include credentials)

### XMLHttpRequest Access

Access the underlying XMLHttpRequest object.

```javascript { .api }
/**
 * Get XMLHttpRequest constructor
 * @returns XMLHttpRequest class
 */
request.getXHR(): typeof XMLHttpRequest;
```

**Usage Examples:**

```javascript
// Get XHR constructor
const XHR = request.getXHR();
console.log(XHR); // XMLHttpRequest constructor

// Check if XHR is available
if (request.getXHR()) {
  console.log('Running in browser');
}

// Access XHR instance (advanced)
const req = request.get('/api/data');
req.on('request', (xhr) => {
  // xhr is the XMLHttpRequest instance
  console.log('XHR opened:', xhr.readyState);
});
```

### Browser File Uploads

Upload files from file inputs or Blob objects.

```javascript { .api }
interface Request {
  /**
   * Attach file from browser
   * @param field - Form field name
   * @param file - File or Blob object
   * @param options - Optional filename and contentType
   * @returns Request instance for chaining
   */
  attach(
    field: string,
    file: File | Blob,
    options?: {filename?: string, contentType?: string}
  ): Request;
}
```

**Usage Examples:**

```javascript
// Upload from file input
const fileInput = document.querySelector('input[type="file"]');

fileInput.addEventListener('change', (e) => {
  const file = e.target.files[0];

  request
    .post('/api/upload')
    .attach('document', file)
    .on('progress', (e) => {
      console.log('Upload progress:', e.percent);
    })
    .end((err, res) => {
      if (err) {
        console.error('Upload failed:', err);
      } else {
        console.log('Upload complete:', res.body);
      }
    });
});

// Upload Blob
const blob = new Blob(['file contents'], {type: 'text/plain'});

request
  .post('/api/upload')
  .attach('file', blob, {filename: 'data.txt'})
  .end((err, res) => {
    console.log('Upload complete');
  });

// Upload canvas as image
const canvas = document.getElementById('myCanvas');
canvas.toBlob((blob) => {
  request
    .post('/api/upload-image')
    .attach('image', blob, {filename: 'canvas.png'})
    .end((err, res) => {
      console.log('Image uploaded');
    });
}, 'image/png');

// Multiple file upload
const fileInput = document.querySelector('input[type="file"][multiple]');

fileInput.addEventListener('change', (e) => {
  const req = request.post('/api/upload-multiple');

  Array.from(e.target.files).forEach((file, index) => {
    req.attach(`file${index}`, file);
  });

  req.end((err, res) => {
    console.log('All files uploaded');
  });
});
```

### Progress Events

Track upload and download progress in the browser.

**Usage Examples:**

```javascript
// Track download progress
request
  .get('/api/large-file')
  .on('progress', (event) => {
    if (event.direction === 'download') {
      const percent = Math.round(event.percent);
      console.log(`Downloaded: ${percent}%`);

      // Update progress bar
      const progressBar = document.getElementById('progress');
      progressBar.style.width = percent + '%';
      progressBar.textContent = percent + '%';
    }
  })
  .end((err, res) => {
    console.log('Download complete');
  });

// Track upload progress
const fileInput = document.querySelector('input[type="file"]');

fileInput.addEventListener('change', (e) => {
  const file = e.target.files[0];

  request
    .post('/api/upload')
    .attach('file', file)
    .on('progress', (event) => {
      if (event.direction === 'upload') {
        const percent = Math.round(event.percent);
        console.log(`Uploaded: ${percent}%`);

        // Update UI
        document.getElementById('uploadProgress').textContent =
          `Uploading: ${percent}% (${event.loaded} / ${event.total} bytes)`;
      }
    })
    .end((err, res) => {
      if (err) {
        document.getElementById('uploadProgress').textContent = 'Upload failed';
      } else {
        document.getElementById('uploadProgress').textContent = 'Upload complete!';
      }
    });
});
```

### Binary Response Types

Specify how binary responses should be handled.

```javascript { .api }
interface Request {
  /**
   * Set binary response type
   * @param type - Response type ('blob' or 'arraybuffer')
   * @returns Request instance for chaining
   */
  responseType(type: 'blob' | 'arraybuffer'): Request;
}
```

**Usage Examples:**

```javascript
// Download as Blob
request
  .get('/api/image.png')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);

    // Display image
    const img = document.createElement('img');
    img.src = url;
    document.body.appendChild(img);
  });

// Download file as Blob
request
  .get('/api/download/report.pdf')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const url = URL.createObjectURL(blob);

    // Trigger download
    const a = document.createElement('a');
    a.href = url;
    a.download = 'report.pdf';
    a.click();
    URL.revokeObjectURL(url);
  });

// Get binary data as ArrayBuffer
request
  .get('/api/binary-data')
  .responseType('arraybuffer')
  .end((err, res) => {
    const buffer = res.body; // ArrayBuffer
    const bytes = new Uint8Array(buffer);

    // Process binary data
    console.log('First byte:', bytes[0]);
  });

// Process image data
request
  .get('/api/image.jpg')
  .responseType('blob')
  .end((err, res) => {
    const blob = res.body;
    const reader = new FileReader();

    reader.onload = (e) => {
      const dataUrl = e.target.result;
      document.getElementById('preview').src = dataUrl;
    };

    reader.readAsDataURL(blob);
  });
```

### Utility Functions

Browser-specific utility functions.

```javascript { .api }
/**
 * Serialize object to query string
 * @param obj - Object to serialize
 * @returns Query string
 */
request.serializeObject(obj: object): string;

/**
 * Parse URL-encoded string to object
 * @param str - URL-encoded string
 * @returns Parsed object
 */
request.parseString(str: string): object;
```

**Usage Examples:**

```javascript
// Serialize object to query string
const params = {
  name: 'John',
  age: 30,
  tags: ['developer', 'nodejs']
};

const queryString = request.serializeObject(params);
console.log(queryString);
// name=John&age=30&tags[0]=developer&tags[1]=nodejs

// Parse query string
const str = 'name=John&age=30&active=true';
const obj = request.parseString(str);
console.log(obj);
// { name: 'John', age: '30', active: 'true' }
```

### MIME Type Shortcuts

Browser-specific MIME type shortcuts.

```javascript { .api }
// Type shortcuts (Browser only)
request.types: {
  html: 'text/html',
  json: 'application/json',
  xml: 'text/xml',
  urlencoded: 'application/x-www-form-urlencoded',
  form: 'application/x-www-form-urlencoded',
  'form-data': 'application/x-www-form-urlencoded'
};
```

**Usage Examples:**

```javascript
// Access type shortcuts
console.log(request.types.json); // 'application/json'
console.log(request.types.form); // 'application/x-www-form-urlencoded'

// Shortcuts are used automatically
request
  .post('/api/data')
  .type('json') // Uses request.types.json
  .send({ data: 'value' });
```

### Abort with User Interaction

Cancel requests based on user actions.

**Usage Examples:**

```javascript
// Cancel button
const downloadBtn = document.getElementById('download');
const cancelBtn = document.getElementById('cancel');

let currentRequest = null;

downloadBtn.addEventListener('click', () => {
  currentRequest = request
    .get('/api/large-file')
    .on('progress', (e) => {
      console.log('Progress:', e.percent);
    })
    .end((err, res) => {
      if (err && err.code === 'ABORTED') {
        console.log('Download cancelled by user');
      } else {
        console.log('Download complete');
      }
      currentRequest = null;
    });
});

cancelBtn.addEventListener('click', () => {
  if (currentRequest) {
    currentRequest.abort();
    console.log('Download cancelled');
  }
});

// Timeout with abort
const req = request.get('/api/data');

setTimeout(() => {
  req.abort();
  console.log('Request timeout');
}, 5000);

req.end((err, res) => {
  if (err && err.code === 'ABORTED') {
    alert('Request took too long and was cancelled');
  }
});
```
