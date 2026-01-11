# Streaming

Stream request and response data for large files and memory-efficient data transfer in Node.js.

## Capabilities

### Pipe Response to Stream

Pipe response data to a writable stream.

```javascript { .api }
/**
 * Pipe response to writable stream
 * @param {Stream} stream - Writable stream
 * @param {object} [options] - Pipe options
 * @returns {Stream} The destination stream
 */
Request.prototype.pipe = function(stream, options);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Download file and save to disk
request
  .get('https://example.com/large-file.zip')
  .pipe(fs.createWriteStream('download.zip'));

// Pipe to stdout
request
  .get('https://api.example.com/data')
  .pipe(process.stdout);

// Pipe with options
request
  .get('https://example.com/file.dat')
  .pipe(fs.createWriteStream('file.dat'), { end: true });

// Pipe to multiple streams
const file = fs.createWriteStream('download.zip');
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

const req = request.get('https://example.com/file.zip');

req.pipe(file);
req.pipe(hash);

req.on('end', () => {
  console.log('Download complete');
  console.log('SHA256:', hash.digest('hex'));
});
```

### Write Request Data

Write raw data to request stream for uploading large files.

```javascript { .api }
/**
 * Write data to request stream
 * @param {Buffer|string} data - Data to write
 * @param {string} [encoding] - Encoding (default: 'utf8')
 * @returns {boolean} False if stream buffer is full
 */
Request.prototype.write = function(data, encoding);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Upload file using streaming
const stream = fs.createReadStream('large-file.zip');

const req = request.post('https://example.com/upload');

stream.on('data', chunk => {
  req.write(chunk);
});

stream.on('end', () => {
  req.end((err, res) => {
    console.log('Upload complete');
  });
});

// Manual chunked upload
const req = request.post('https://example.com/upload');

req.write(Buffer.from('chunk1'));
req.write(Buffer.from('chunk2'));
req.write(Buffer.from('chunk3'));

req.end((err, res) => {
  console.log('Upload complete');
});

// Upload with backpressure handling
const stream = fs.createReadStream('large-file.dat');
const req = request.post('https://example.com/upload');

stream.on('data', chunk => {
  const canContinue = req.write(chunk);

  if (!canContinue) {
    // Pause reading if buffer is full
    stream.pause();

    req.on('drain', () => {
      // Resume when buffer is drained
      stream.resume();
    });
  }
});

stream.on('end', () => {
  req.end(callback);
});
```

### Stream Response Control

Control response stream flow.

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
// Pause and resume download
request
  .get('https://example.com/large-file.zip')
  .buffer(false)
  .end((err, res) => {
    let bytesReceived = 0;

    res.on('data', chunk => {
      bytesReceived += chunk.length;
      console.log('Received:', bytesReceived, 'bytes');

      // Pause every 1MB
      if (bytesReceived % (1024 * 1024) === 0) {
        res.pause();
        console.log('Paused for 1 second');

        setTimeout(() => {
          res.resume();
          console.log('Resumed');
        }, 1000);
      }
    });

    res.on('end', () => {
      console.log('Download complete:', bytesReceived, 'bytes');
    });
  });

// Destroy on error
request
  .get('https://example.com/data')
  .buffer(false)
  .end((err, res) => {
    res.on('data', chunk => {
      try {
        processChunk(chunk);
      } catch (err) {
        console.error('Processing error:', err);
        res.destroy(err);
      }
    });
  });

// Cancel download conditionally
request
  .get('https://example.com/file.zip')
  .buffer(false)
  .end((err, res) => {
    let bytesReceived = 0;
    const maxSize = 10 * 1024 * 1024; // 10MB

    res.on('data', chunk => {
      bytesReceived += chunk.length;

      if (bytesReceived > maxSize) {
        console.error('File too large');
        res.destroy(new Error('File exceeds maximum size'));
      }
    });
  });
```

### Buffering Control

Disable buffering to enable streaming.

```javascript { .api }
/**
 * Enable or disable response buffering
 * @param {boolean} [enable] - Enable buffering (default: true)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.buffer = function(enable);
```

**Usage Examples:**

```javascript
// Disable buffering for streaming
request
  .get('https://example.com/large-file.zip')
  .buffer(false)  // Required for streaming
  .end((err, res) => {
    // res.body is undefined when buffering is disabled
    // Must use streaming events
  });

// Enable buffering (default)
request
  .get('https://api.example.com/data')
  .buffer(true)
  .end((err, res) => {
    console.log('Body:', res.body);  // Parsed body available
  });
```

### Download Files

Stream download examples for various scenarios.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');
const path = require('path');

// Simple file download
function downloadFile(url, destination) {
  return new Promise((resolve, reject) => {
    const file = fs.createWriteStream(destination);

    request
      .get(url)
      .on('error', reject)
      .pipe(file)
      .on('finish', resolve)
      .on('error', reject);
  });
}

await downloadFile('https://example.com/file.zip', 'download.zip');

// Download with progress tracking
function downloadWithProgress(url, destination) {
  return new Promise((resolve, reject) => {
    const file = fs.createWriteStream(destination);
    let receivedBytes = 0;

    request
      .get(url)
      .on('response', res => {
        const totalBytes = parseInt(res.headers['content-length'], 10);
        console.log('Total size:', totalBytes, 'bytes');

        res.on('data', chunk => {
          receivedBytes += chunk.length;
          const progress = (receivedBytes / totalBytes * 100).toFixed(2);
          console.log(`Progress: ${progress}% (${receivedBytes}/${totalBytes})`);
        });
      })
      .on('error', reject)
      .pipe(file)
      .on('finish', resolve)
      .on('error', reject);
  });
}

await downloadWithProgress('https://example.com/large.zip', 'download.zip');

// Download multiple files concurrently
async function downloadMultiple(downloads) {
  const promises = downloads.map(({ url, destination }) =>
    downloadFile(url, destination)
  );

  return Promise.all(promises);
}

await downloadMultiple([
  { url: 'https://example.com/file1.zip', destination: 'file1.zip' },
  { url: 'https://example.com/file2.zip', destination: 'file2.zip' },
  { url: 'https://example.com/file3.zip', destination: 'file3.zip' }
]);

// Download with retry
async function downloadWithRetry(url, destination, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await downloadFile(url, destination);
      console.log('Download successful');
      return;
    } catch (err) {
      console.error(`Attempt ${i + 1} failed:`, err.message);

      if (i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw new Error('Download failed after ' + maxRetries + ' attempts');
}

await downloadWithRetry('https://example.com/file.zip', 'download.zip', 3);

// Resumable download (basic implementation)
async function resumableDownload(url, destination) {
  const tempFile = destination + '.part';
  let startByte = 0;

  if (fs.existsSync(tempFile)) {
    const stats = fs.statSync(tempFile);
    startByte = stats.size;
    console.log('Resuming from byte:', startByte);
  }

  return new Promise((resolve, reject) => {
    const file = fs.createWriteStream(tempFile, {
      flags: startByte > 0 ? 'a' : 'w'
    });

    request
      .get(url)
      .set('Range', `bytes=${startByte}-`)
      .on('error', reject)
      .pipe(file)
      .on('finish', () => {
        fs.renameSync(tempFile, destination);
        resolve();
      })
      .on('error', reject);
  });
}
```

### Upload Files

Stream upload examples for various scenarios.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Simple file upload
function uploadFile(file, url) {
  return new Promise((resolve, reject) => {
    const stream = fs.createReadStream(file);

    const req = request.post(url);

    stream.on('data', chunk => {
      req.write(chunk);
    });

    stream.on('end', () => {
      req.end((err, res) => {
        if (err) {
          reject(err);
        } else {
          resolve(res.body);
        }
      });
    });

    stream.on('error', reject);
  });
}

await uploadFile('large-file.zip', 'https://example.com/upload');

// Upload with progress tracking
function uploadWithProgress(file, url) {
  return new Promise((resolve, reject) => {
    const stats = fs.statSync(file);
    const totalBytes = stats.size;
    let uploadedBytes = 0;

    const stream = fs.createReadStream(file);
    const req = request.post(url);

    stream.on('data', chunk => {
      uploadedBytes += chunk.length;
      const progress = (uploadedBytes / totalBytes * 100).toFixed(2);
      console.log(`Progress: ${progress}% (${uploadedBytes}/${totalBytes})`);

      req.write(chunk);
    });

    stream.on('end', () => {
      req.end((err, res) => {
        if (err) {
          reject(err);
        } else {
          resolve(res.body);
        }
      });
    });

    stream.on('error', reject);
  });
}

await uploadWithProgress('large.zip', 'https://example.com/upload');

// Upload with multipart
async function uploadMultipart(file, url) {
  return request
    .post(url)
    .attach('file', fs.createReadStream(file))
    .field('name', path.basename(file))
    .field('timestamp', Date.now().toString());
}

await uploadMultipart('document.pdf', 'https://example.com/upload');

// Chunked upload
async function chunkedUpload(file, url, chunkSize = 1024 * 1024) {
  const stats = fs.statSync(file);
  const totalBytes = stats.size;
  let uploadedBytes = 0;

  const stream = fs.createReadStream(file, { highWaterMark: chunkSize });

  for await (const chunk of stream) {
    await request
      .post(url)
      .set('Content-Range', `bytes ${uploadedBytes}-${uploadedBytes + chunk.length - 1}/${totalBytes}`)
      .send(chunk);

    uploadedBytes += chunk.length;
    console.log(`Uploaded: ${uploadedBytes}/${totalBytes} bytes`);
  }

  console.log('Upload complete');
}

await chunkedUpload('large-file.dat', 'https://example.com/upload', 5 * 1024 * 1024);
```

### Stream Transformations

Transform streams during upload or download.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');
const { Transform } = require('stream');

// Download and decompress on the fly
request
  .get('https://example.com/file.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('file.txt'));

// Download, decrypt, and save
const decipher = crypto.createDecipheriv('aes-256-cbc', key, iv);

request
  .get('https://example.com/encrypted-file')
  .pipe(decipher)
  .pipe(fs.createWriteStream('decrypted-file.txt'));

// Upload with compression
const gzip = zlib.createGzip();
const file = fs.createReadStream('large-file.txt');
const req = request.post('https://example.com/upload');

file
  .pipe(gzip)
  .on('data', chunk => {
    req.write(chunk);
  })
  .on('end', () => {
    req.end(callback);
  });

// Custom transformation stream
class ProgressStream extends Transform {
  constructor(totalBytes) {
    super();
    this.uploadedBytes = 0;
    this.totalBytes = totalBytes;
  }

  _transform(chunk, encoding, callback) {
    this.uploadedBytes += chunk.length;
    const progress = (this.uploadedBytes / this.totalBytes * 100).toFixed(2);
    console.log(`Progress: ${progress}%`);
    this.push(chunk);
    callback();
  }
}

const stats = fs.statSync('file.dat');
const progress = new ProgressStream(stats.size);

fs.createReadStream('file.dat')
  .pipe(progress)
  .pipe(request.post('https://example.com/upload'));

// Hash calculation during upload
const hash = crypto.createHash('sha256');
const file = fs.createReadStream('file.dat');
const req = request.post('https://example.com/upload');

file
  .on('data', chunk => {
    hash.update(chunk);
    req.write(chunk);
  })
  .on('end', () => {
    const checksum = hash.digest('hex');
    req.set('X-Checksum', checksum);
    req.end(callback);
  });
```

### Complete Streaming Example

Comprehensive example showing streaming capabilities.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');
const zlib = require('zlib');

class FileTransfer {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }

  // Download file with progress and verification
  async download(remotePath, localPath) {
    return new Promise((resolve, reject) => {
      const file = fs.createWriteStream(localPath);
      const hash = crypto.createHash('sha256');
      let receivedBytes = 0;

      request
        .get(`${this.baseUrl}${remotePath}`)
        .on('response', res => {
          const totalBytes = parseInt(res.headers['content-length'], 10);
          const checksum = res.headers['x-checksum'];

          console.log('Downloading:', remotePath);
          console.log('Size:', totalBytes, 'bytes');
          console.log('Expected checksum:', checksum);

          res.on('data', chunk => {
            receivedBytes += chunk.length;
            hash.update(chunk);

            const progress = (receivedBytes / totalBytes * 100).toFixed(2);
            process.stdout.write(`\rProgress: ${progress}%`);
          });

          res.on('end', () => {
            const actualChecksum = hash.digest('hex');

            if (checksum && actualChecksum !== checksum) {
              reject(new Error('Checksum mismatch'));
            } else {
              console.log('\nDownload complete');
              console.log('Checksum:', actualChecksum);
            }
          });
        })
        .on('error', reject)
        .pipe(file)
        .on('finish', resolve)
        .on('error', reject);
    });
  }

  // Upload file with progress and compression
  async upload(localPath, remotePath, compress = false) {
    return new Promise((resolve, reject) => {
      const stats = fs.statSync(localPath);
      const totalBytes = stats.size;
      let uploadedBytes = 0;

      const hash = crypto.createHash('sha256');
      let stream = fs.createReadStream(localPath);

      // Add compression if requested
      if (compress) {
        stream = stream.pipe(zlib.createGzip());
      }

      const req = request
        .post(`${this.baseUrl}${remotePath}`)
        .set('Content-Type', 'application/octet-stream');

      if (compress) {
        req.set('Content-Encoding', 'gzip');
      }

      stream.on('data', chunk => {
        hash.update(chunk);
        uploadedBytes += chunk.length;

        const progress = (uploadedBytes / totalBytes * 100).toFixed(2);
        process.stdout.write(`\rProgress: ${progress}%`);

        req.write(chunk);
      });

      stream.on('end', () => {
        const checksum = hash.digest('hex');
        req.set('X-Checksum', checksum);

        req.end((err, res) => {
          if (err) {
            reject(err);
          } else {
            console.log('\nUpload complete');
            console.log('Checksum:', checksum);
            resolve(res.body);
          }
        });
      });

      stream.on('error', reject);
    });
  }

  // Stream processing (download, transform, upload)
  async processAndUpload(downloadPath, uploadPath, transform) {
    return new Promise((resolve, reject) => {
      const req = request.post(`${this.baseUrl}${uploadPath}`);

      request
        .get(`${this.baseUrl}${downloadPath}`)
        .on('response', res => {
          console.log('Processing file...');

          let processedBytes = 0;

          res
            .pipe(transform)
            .on('data', chunk => {
              processedBytes += chunk.length;
              console.log('Processed:', processedBytes, 'bytes');
              req.write(chunk);
            })
            .on('end', () => {
              req.end((err, res) => {
                if (err) {
                  reject(err);
                } else {
                  console.log('Processing complete');
                  resolve(res.body);
                }
              });
            })
            .on('error', reject);
        })
        .on('error', reject);
    });
  }
}

// Usage
const transfer = new FileTransfer('https://api.example.com');

// Download
await transfer.download('/files/large.zip', 'download.zip');

// Upload with compression
await transfer.upload('large-file.txt', '/files/upload', true);

// Process: download, compress, upload
const gzip = zlib.createGzip();
await transfer.processAndUpload(
  '/files/data.txt',
  '/files/compressed.gz',
  gzip
);
```

### Response Stream Control

Control the flow of response data when streaming.

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

/**
 * Set response stream encoding
 * @param {string} encoding - Encoding (e.g., 'utf8', 'ascii', 'base64')
 * @returns {Response} Response instance
 */
Response.prototype.setEncoding = function(encoding);
```

**Usage Examples:**

```javascript
// Pause and resume response
request
  .get('https://example.com/large-file.zip')
  .on('response', res => {
    console.log('Response started');

    // Pause to slow down data flow
    res.pause();

    setTimeout(() => {
      console.log('Resuming...');
      res.resume(); // Resume reading data
    }, 1000);
  })
  .pipe(fs.createWriteStream('download.zip'));

// Destroy stream on error
request
  .get('https://example.com/file.zip')
  .on('response', res => {
    res.on('data', chunk => {
      // Check chunk validity
      if (!isValidChunk(chunk)) {
        res.destroy(new Error('Invalid chunk'));
      }
    });
  })
  .pipe(fs.createWriteStream('file.zip'));

// Set encoding for text streams
request
  .get('https://api.example.com/data.txt')
  .on('response', res => {
    res.setEncoding('utf8');

    res.on('data', chunk => {
      console.log('Text chunk:', chunk); // String, not Buffer
    });
  })
  .buffer(false)
  .end();
```

### Important Notes

- **Buffering**: Must disable buffering (`.buffer(false)`) to enable streaming for responses.
- **Memory Efficiency**: Streaming is memory-efficient for large files as data is processed in chunks.
- **Backpressure**: Handle backpressure when writing to requests by checking the return value of `.write()` and listening to `drain` event.
- **Error Handling**: Always handle errors on both the request/response and stream to prevent crashes.
- **Node.js Only**: Streaming APIs are only available in Node.js, not in browsers.
- **Piping**: `.pipe()` automatically calls `.end()` when the stream finishes.
- **Progress Tracking**: Use `data` events to track progress during streaming operations.
- **Stream Control**: Use `.pause()`, `.resume()`, and `.destroy()` to control response stream flow.
