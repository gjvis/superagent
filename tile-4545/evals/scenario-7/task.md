# Large File Downloader

Build a file download utility that streams large responses directly to disk without buffering the entire response in memory.

## Requirements

Your implementation should:

1. Download a file from a URL
2. Stream the response data directly to a file on disk
3. Disable response buffering to enable streaming
4. Handle the stream completion event
5. Support downloading large files efficiently

## Test Cases

- Downloading a file and piping to a writable stream saves the file to disk @test
- Response buffering is disabled to enable streaming @test
- The stream end event is handled to detect download completion @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
