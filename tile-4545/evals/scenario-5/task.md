# Extensible HTTP Client

Build an HTTP client with custom middleware/plugins that modify requests before they are sent.

## Requirements

Your implementation should:

1. Create plugin functions that modify requests (e.g., add headers, log requests)
2. Apply plugins to individual requests
3. Create reusable plugins that can be shared across multiple requests
4. Demonstrate plugin chaining (applying multiple plugins to one request)
5. Show that plugins receive and can modify the request object

## Test Cases

- A plugin that adds a custom header applies that header to the request @test
- Multiple plugins can be chained on a single request, each modifying it @test
- A plugin function receives the request object as its parameter @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
