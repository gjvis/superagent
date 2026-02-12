# Session Manager

Build a session-aware HTTP client that maintains cookies across multiple requests, simulating a logged-in user session.

## Requirements

Your implementation should:

1. Create a persistent client that maintains state across requests
2. Make an initial request that sets cookies (e.g., login)
3. Make subsequent requests that automatically include previously received cookies
4. Verify that cookies from the first request are sent with later requests
5. Handle multiple sequential requests with cookie persistence

## Test Cases

- After a login request sets a session cookie, subsequent requests automatically include that cookie @test
- Multiple requests through the same client instance share cookies @test
- Creating a new client instance does not share cookies with the first instance @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
