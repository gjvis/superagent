# Fetch User Profile Data

Build a function that retrieves user profile information from a REST API endpoint.

## Requirements

Create a function `fetchUserProfile(userId)` that:

- Accepts a user ID as a parameter
- Makes an HTTP GET request to `https://api.example.com/users/{userId}`
- Returns a promise that resolves with the parsed response body
- The response will be in JSON format containing user data

## Test Cases

- Calling `fetchUserProfile(123)` should make a GET request to `https://api.example.com/users/123` and return the parsed user object @test
- Calling `fetchUserProfile('abc-def')` should make a GET request to `https://api.example.com/users/abc-def` and return the user data @test
- The function should handle the promise-based response correctly @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
