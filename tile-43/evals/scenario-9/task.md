# Add Custom Request Logging

Build a reusable plugin that adds custom logging to all requests.

## Requirements

Create two functions:

1. `loggingPlugin(request)` - A plugin function that:
   - Receives a request object
   - Adds a custom header `X-Request-ID` with a unique ID (can use timestamp or counter)
   - Adds a custom header `X-Client-Version` with value "1.0.0"
   - Returns the modified request object

2. `fetchWithLogging(url)` - A function that:
   - Makes a GET request to the provided URL
   - Uses the loggingPlugin to add the custom headers
   - Returns the response data

## Test Cases

- The loggingPlugin should add both custom headers to any request object passed to it @test
- Calling `fetchWithLogging('https://api.example.com/data')` should include the X-Request-ID and X-Client-Version headers @test
- The plugin should work with multiple different requests @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
