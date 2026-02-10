# Resilient API Client with Retry

Create a fault-tolerant HTTP client that automatically retries failed requests.

## Requirements

Implement a function `fetchWithRetry(url, retryConfig)` that:

1. Makes a GET request to the provided URL
2. Automatically retries the request on failure up to a specified number of times
3. Supports simple retry count configuration
4. Supports custom retry logic with a callback that decides whether to retry based on the error
5. Returns the successful response or the final error after all retries exhausted

Example usage:
```javascript
// Simple: retry up to 3 times on any error
await fetchWithRetry('https://unstable-api.com/data', 3)

// Advanced: custom retry logic
await fetchWithRetry('https://api.com/data', {
  count: 5,
  shouldRetry: (err, res) => {
    // Only retry on 5xx errors or network issues
    return err.status >= 500 || err.code === 'ECONNRESET'
  }
})
```

Track and log the number of retry attempts made.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Flaky endpoint that fails twice then succeeds, retry count 3
Expected: Succeeds after 2 retries and returns data

@test Input: Endpoint returning 500 error, retry count 2
Expected: Retries twice then returns final error

@test Input: Custom retry callback that only retries on specific error codes
Expected: Respects custom retry logic
