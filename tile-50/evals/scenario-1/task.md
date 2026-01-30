# Resilient API Client with Progress Tracking

Build a resilient HTTP API client that handles file uploads with progress tracking, implements intelligent retry logic, and manages authentication sessions across multiple requests.

## Capabilities

### File Upload with Progress Tracking

- Uploads a file with metadata fields and emits progress events during upload [@test](../test/upload.test.js)
- Progress events include upload percentage and bytes transferred [@test](../test/upload.test.js)

### Custom Response Parsing

- Handles custom XML responses by registering a parser for the 'application/xml' content type [@test](../test/custom-parser.test.js)
- Falls back to default parsing for standard JSON responses [@test](../test/custom-parser.test.js)

### Intelligent Retry Logic

- Retries failed requests with custom retry conditions [@test](../test/retry.test.js)
- Distinguishes between retryable errors (network timeouts, 503) and non-retryable errors (400, 401) [@test](../test/retry.test.js)

### Session Management

- Maintains session state across multiple requests using persistent cookie storage [@test](../test/session.test.js)
- Automatically includes session cookies in subsequent requests after authentication [@test](../test/session.test.js)

## Implementation

[@generates](./src/api-client.js)

## API

```javascript { #api }
/**
 * Creates a resilient API client with session management.
 *
 * @param {string} baseUrl - The base URL for API requests
 * @returns {Object} API client with methods for various operations
 */
function createApiClient(baseUrl) {
  // IMPLEMENTATION HERE
}

/**
 * Uploads a file with metadata and progress tracking.
 *
 * @param {Object} agent - The HTTP agent maintaining session state
 * @param {string} url - The upload endpoint URL
 * @param {string} filePath - Path to the file to upload
 * @param {Object} metadata - Key-value pairs of metadata fields
 * @param {Function} onProgress - Callback for progress events (percent, loaded, total)
 * @returns {Promise<Object>} Response object from the server
 */
function uploadWithProgress(agent, url, filePath, metadata, onProgress) {
  // IMPLEMENTATION HERE
}

/**
 * Registers a custom response parser for a specific content type.
 *
 * @param {Object} request - The request library instance
 * @param {string} contentType - The MIME type to register parser for
 * @param {Function} parserFn - Function that parses response text to object
 */
function registerCustomParser(request, contentType, parserFn) {
  // IMPLEMENTATION HERE
}

/**
 * Performs a request with intelligent retry logic.
 *
 * @param {Object} agent - The HTTP agent
 * @param {string} url - The request URL
 * @param {number} maxRetries - Maximum number of retry attempts
 * @returns {Promise<Object>} Response object
 */
function requestWithRetry(agent, url, maxRetries) {
  // IMPLEMENTATION HERE
}

module.exports = {
  createApiClient,
  uploadWithProgress,
  registerCustomParser,
  requestWithRetry
};
```

## Dependencies { .dependencies }

### superagent { .dependency }

Provides HTTP request functionality with features for file uploads, custom parsing, retry logic, and session management.
