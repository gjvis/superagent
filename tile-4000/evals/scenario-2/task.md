# Protected API Client

Create a client that accesses protected API endpoints using authentication.

## Requirements

Implement two functions:

1. `fetchWithBasicAuth(url, username, password)` - Makes a GET request with HTTP Basic Authentication
2. `fetchWithBearerToken(url, token)` - Makes a GET request with Bearer token authentication

Both functions should:
- Set the appropriate authentication credentials
- Return the response data
- Handle authentication errors appropriately

Example usage:
```javascript
// Basic auth
await fetchWithBasicAuth('https://api.secure.com/data', 'user123', 'pass456')

// Bearer token
await fetchWithBearerToken('https://api.secure.com/profile', 'eyJhbGc...')
```

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Basic auth with username "admin" and password "secret"
Expected: Request includes proper Authorization header with Base64 encoded credentials

@test Input: Bearer token "abc123token"
Expected: Request includes Authorization header with "Bearer abc123token"

@test Input: Invalid credentials
Expected: Handles 401 Unauthorized response appropriately
