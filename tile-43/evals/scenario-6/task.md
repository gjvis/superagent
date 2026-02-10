# Fetch Data from Unreliable API

Build a function that fetches data from an API that occasionally fails due to transient errors.

## Requirements

Create a function `fetchWithRetry(url)` that:

- Accepts a URL string
- Makes an HTTP GET request to the provided URL
- Automatically retries the request up to 3 times if it fails
- Only retries on transient errors (network issues, 5xx server errors)
- Returns a promise that resolves with the response data

## Test Cases

- Calling `fetchWithRetry('https://api.example.com/data')` should retry up to 3 times on server errors @test
- The function should eventually succeed if the server recovers within the retry limit @test
- The function should handle permanent failures after exhausting retries @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
