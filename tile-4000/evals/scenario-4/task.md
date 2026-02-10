# Timeout-Aware API Client

Create a resilient API client that handles slow or unresponsive servers.

## Requirements

Implement a function `fetchWithTimeout(url, timeoutConfig)` that:

1. Makes a GET request to the provided URL
2. Supports configuring a total request timeout (deadline)
3. Supports configuring a response timeout (time to first byte)
4. Catches and handles timeout errors gracefully
5. Returns an object with `{success: boolean, data?: any, error?: string, timedOut: boolean}`

Example usage:
```javascript
// Simple timeout (5 seconds total)
await fetchWithTimeout('https://slow-api.com/data', 5000)

// Separate response and deadline timeouts
await fetchWithTimeout('https://api.com/large-data', {
  response: 3000,  // 3s to start receiving response
  deadline: 10000  // 10s total
})
```

The function should distinguish between timeout errors and other errors.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Request with 2000ms timeout to a fast endpoint
Expected: Completes successfully before timeout

@test Input: Request with 100ms timeout to a slow endpoint
Expected: Times out and returns appropriate error with timedOut flag

@test Input: Separate response (2000ms) and deadline (5000ms) timeouts
Expected: Applies both timeout constraints appropriately
