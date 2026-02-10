# Maintain Session Across Multiple Requests

Build a function that performs authenticated operations by maintaining session cookies across multiple API calls.

## Requirements

Create a function `loginAndFetchData(username, password)` that:

- Creates a persistent HTTP client that saves cookies
- First POSTs credentials to `https://api.example.com/login` with `{username, password}`
- The server responds with a Set-Cookie header containing session information
- Then makes a GET request to `https://api.example.com/profile` using the same client
- The second request should automatically include the session cookie
- Returns a promise that resolves with the profile data

## Test Cases

- Calling `loginAndFetchData('user', 'pass')` should login and fetch profile using persisted cookies @test
- The session cookie from the login response should be automatically sent with the profile request @test
- Both requests should use the same client instance @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
