# Progress-Tracking Downloader

Create a download utility with real-time progress monitoring.

## Requirements

Implement a function `downloadWithProgress(url, onProgress)` that:

1. Initiates a GET request to download content from the URL
2. Listens for progress events during the download
3. Calls the `onProgress` callback with progress information including:
   - Direction (upload or download)
   - Bytes loaded so far
   - Total bytes (if known)
   - Percentage complete
4. Returns the final downloaded data

Example usage:
```javascript
await downloadWithProgress('https://cdn.example.com/large-file.zip', (progress) => {
  console.log(`Downloaded: ${progress.percent}%`)
  console.log(`${progress.loaded} / ${progress.total} bytes`)
})
```

Also implement `uploadWithProgress(url, filePath, onProgress)` that tracks upload progress.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Download 1MB file with progress callback
Expected: Progress callback invoked multiple times with increasing loaded values

@test Input: Upload file with progress tracking
Expected: Progress events show upload direction and track upload bytes

@test Input: Progress callback receives percent property
Expected: Percentage calculated and provided in progress events
