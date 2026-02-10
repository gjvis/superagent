# Extensible HTTP Client with Plugins

Create an HTTP client with custom plugin functionality.

## Requirements

Implement the following:

1. Create a `loggingPlugin` function that logs all outgoing requests with method and URL
2. Create a `authHeaderPlugin(apiKey)` function that adds an API key header to all requests
3. Create a `timingPlugin` function that measures and logs request duration
4. Implement a `makeRequest(url, plugins)` function that applies all provided plugins to the request

Example usage:
```javascript
const loggingPlugin = (request) => {
  console.log(`Request: ${request.method} ${request.url}`)
}

const authPlugin = (apiKey) => (request) => {
  request.set('X-API-Key', apiKey)
}

const timingPlugin = (request) => {
  const start = Date.now()
  request.on('response', () => {
    console.log(`Duration: ${Date.now() - start}ms`)
  })
}

await makeRequest('https://api.com/data', [
  loggingPlugin,
  authPlugin('secret-key-123'),
  timingPlugin
])
```

Plugins should be chainable and applied in order.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Request with logging plugin
Expected: Logs request details before sending

@test Input: Request with auth header plugin
Expected: Request includes API key header

@test Input: Request with multiple chained plugins
Expected: All plugins applied in order
