# Redirect-Aware HTTP Client

Create a client that handles HTTP redirects with tracking capabilities.

## Requirements

Implement a function `fetchWithRedirectTracking(url, maxRedirects)` that:

1. Makes a GET request to the provided URL
2. Configures the maximum number of redirects to follow
3. Follows redirects automatically up to the specified limit
4. Returns both the final response data and a list of all redirect URLs encountered
5. Handles the case where redirect limit is exceeded

Example usage:
```javascript
const result = await fetchWithRedirectTracking('https://short.url/abc123', 5)
// Returns: {
//   data: { ...final response... },
//   redirects: ['https://medium.url/xyz', 'https://final.url/destination'],
//   redirectCount: 2
// }
```

Also implement `fetchWithoutRedirects(url)` that disables automatic redirect following.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: URL with 2 redirects, maxRedirects set to 5
Expected: Follows all redirects and returns final content with redirect list

@test Input: URL with 3 redirects, maxRedirects set to 1
Expected: Stops after 1 redirect and handles error appropriately

@test Input: Direct URL with maxRedirects set to 0
Expected: Fetches content without following any redirects
