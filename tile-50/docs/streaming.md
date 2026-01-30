# Streaming (Node.js)

Stream request and response data for large files and real-time processing. Available in Node.js only.

## Capabilities

### Pipe Response to Stream

Pipe response data to a writable stream.

```javascript { .api }
/**
 * Pipe response to writable stream (Node.js only)
 * Automatically calls .buffer(false) and .end()
 * @param stream - Writable stream destination
 * @param options - Optional stream pipe options
 * @returns Stream
 */
request.get(url).pipe(stream: WritableStream, options?: object): Stream;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Download file to disk
request
  .get('https://example.com/large-file.zip')
  .pipe(fs.createWriteStream('download.zip'))
  .on('finish', () => console.log('Download complete'));

// Pipe to multiple destinations
const req = request.get('https://example.com/data.json');
req.pipe(fs.createWriteStream('data1.json'));
req.pipe(fs.createWriteStream('data2.json'));

// Pipe with response transformation
request
  .get('https://example.com/data.csv')
  .pipe(csvParser())
  .pipe(transformer())
  .pipe(fs.createWriteStream('output.json'));

// Download with progress
const fileStream = fs.createWriteStream('download.dat');
let downloaded = 0;

request
  .get('https://example.com/file.dat')
  .on('response', res => {
    const total = parseInt(res.get('Content-Length'));
    console.log('Total size:', total);
  })
  .on('data', chunk => {
    downloaded += chunk.length;
    console.log('Downloaded:', downloaded);
  })
  .pipe(fileStream);
```

### Write Data to Request Stream

Write raw data to request stream for uploads.

```javascript { .api }
/**
 * Write raw data to request stream (Node.js only)
 * Used for streaming uploads
 * @param data - Data to write (string or Buffer)
 * @param encoding - Optional encoding for strings
 * @returns Boolean indicating backpressure
 */
request.post(url).write(data: string | Buffer, encoding?: string): boolean;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Upload file by streaming
const req = request.post('https://example.com/upload');

req.set('Content-Type', 'application/octet-stream');

fs.createReadStream('large-file.dat')
  .pipe(req)
  .on('response', res => {
    console.log('Upload complete:', res.body);
  });

// Manual write operations
const req = request.post('https://example.com/upload');

req.write('chunk1');
req.write('chunk2');
req.write('chunk3');
req.end();

// Write with backpressure handling
const req = request.post('https://example.com/upload');

function writeData(data) {
  const canContinue = req.write(data);
  if (!canContinue) {
    req.once('drain', () => {
      console.log('Can write more data');
    });
  }
}
```

## Streaming Patterns

### Download Large File

```javascript
const fs = require('fs');

function downloadFile(url, destination) {
  return new Promise((resolve, reject) => {
    const stream = fs.createWriteStream(destination);

    request
      .get(url)
      .on('error', reject)
      .pipe(stream)
      .on('error', reject)
      .on('finish', resolve);
  });
}

await downloadFile('https://example.com/file.zip', './download.zip');
```

### Upload Large File

```javascript
const fs = require('fs');

function uploadFile(url, filePath) {
  return new Promise((resolve, reject) => {
    const stream = fs.createReadStream(filePath);

    stream
      .pipe(request.post(url))
      .on('error', reject)
      .on('response', res => {
        if (res.ok) {
          resolve(res.body);
        } else {
          reject(new Error(`Upload failed: ${res.status}`));
        }
      });
  });
}

await uploadFile('https://example.com/upload', './large-file.dat');
```

### Stream with Progress Tracking

```javascript
const fs = require('fs');

function downloadWithProgress(url, destination, onProgress) {
  return new Promise((resolve, reject) => {
    const fileStream = fs.createWriteStream(destination);
    let downloaded = 0;
    let total = 0;

    request
      .get(url)
      .on('response', res => {
        total = parseInt(res.get('Content-Length')) || 0;
      })
      .on('data', chunk => {
        downloaded += chunk.length;
        if (total > 0 && onProgress) {
          onProgress(downloaded, total, (downloaded / total) * 100);
        }
      })
      .on('error', reject)
      .pipe(fileStream)
      .on('error', reject)
      .on('finish', resolve);
  });
}

// Usage
await downloadWithProgress(
  'https://example.com/large-file.zip',
  './download.zip',
  (downloaded, total, percent) => {
    console.log(`Progress: ${percent.toFixed(2)}% (${downloaded}/${total})`);
  }
);
```

### Process Stream Data

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// Custom transform stream
class LineCounter extends Transform {
  constructor() {
    super();
    this.lineCount = 0;
  }

  _transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n').length - 1;
    this.lineCount += lines;
    this.push(chunk);
    callback();
  }
}

const counter = new LineCounter();

request
  .get('https://example.com/large-text.txt')
  .pipe(counter)
  .pipe(fs.createWriteStream('output.txt'))
  .on('finish', () => {
    console.log('Line count:', counter.lineCount);
  });
```

### Conditional Streaming

```javascript
const fs = require('fs');

// Only stream if response is large
request
  .get('https://example.com/data')
  .buffer(false)
  .on('response', res => {
    const size = parseInt(res.get('Content-Length'));

    if (size > 10 * 1024 * 1024) {
      // Large file: stream to disk
      console.log('Large file, streaming to disk');
      res.pipe(fs.createWriteStream('large-data.bin'));
    } else {
      // Small file: buffer in memory
      console.log('Small file, buffering');
      let data = Buffer.alloc(0);
      res.on('data', chunk => {
        data = Buffer.concat([data, chunk]);
      });
      res.on('end', () => {
        console.log('Buffered data:', data.length, 'bytes');
      });
    }
  })
  .end();
```

## Important Notes

### Buffering and Streaming

- `.pipe()` automatically calls `.buffer(false)` and `.end()`
- When using `.pipe()`, do not call `.then()` or `.end()`
- Response body (`res.body`) is not available when streaming
- Use `.on('data')` event to access streamed data

```javascript
// Correct: streaming
request
  .get('/large-file')
  .pipe(fs.createWriteStream('file.dat'));

// Incorrect: mixing streaming and promises
request
  .get('/large-file')
  .pipe(fs.createWriteStream('file.dat'))
  .then(res => console.log(res.body)); // ERROR: res.body is undefined
```

### Events with Streaming

```javascript
request
  .get('/file')
  .on('request', req => console.log('Request started'))
  .on('response', res => console.log('Response:', res.status))
  .on('data', chunk => console.log('Data chunk:', chunk.length))
  .on('end', () => console.log('Stream ended'))
  .on('error', err => console.error('Error:', err))
  .pipe(fs.createWriteStream('output.dat'));
```

### Error Handling

```javascript
const fs = require('fs');

const stream = fs.createWriteStream('output.dat');

request
  .get('/file')
  .on('error', err => {
    console.error('Request error:', err);
    stream.destroy();
  })
  .pipe(stream)
  .on('error', err => {
    console.error('Stream error:', err);
  })
  .on('finish', () => {
    console.log('Success');
  });
```
