# Download File with Progress Tracking

Build a function that downloads a large file while reporting download progress.

## Requirements

Create a function `downloadWithProgress(url, progressCallback)` that:

- Accepts a URL string and a progress callback function
- Makes an HTTP GET request to download the file
- Calls the progressCallback with progress information during the download
- The progress info should include percent complete and bytes downloaded
- Returns a promise that resolves with the complete file data

## Test Cases

- Calling `downloadWithProgress('https://api.example.com/file.zip', callback)` should invoke callback multiple times with progress updates @test
- The progress callback should receive an object with `percent` and `loaded` properties @test
- The function should complete and return the full response data @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
