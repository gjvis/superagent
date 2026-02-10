# Access Protected API Endpoint

Build a function that retrieves data from a protected API endpoint that requires authentication.

## Requirements

Create a function `fetchProtectedData(username, password)` that:

- Accepts username and password credentials
- Makes an HTTP GET request to `https://api.example.com/protected/data`
- Includes HTTP Basic authentication credentials in the request
- Returns a promise that resolves with the protected data

## Test Cases

- Calling `fetchProtectedData('admin', 'secret123')` should send credentials and return the protected data @test
- Calling `fetchProtectedData('user@example.com', 'p@ssw0rd!')` should properly encode special characters in credentials @test
- The function should work with any valid username/password combination @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
