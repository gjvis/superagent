# HTTP Client Utility

Create a command-line utility that fetches data from multiple REST API endpoints and displays the results.

## Requirements

Implement a Node.js script that:

1. Fetches user data from a GET endpoint at `https://api.example.com/users/123`
2. Creates a new post by sending JSON data to a POST endpoint at `https://api.example.com/posts`
3. Updates an existing resource using a PUT request to `https://api.example.com/posts/456`
4. Deletes a resource using a DELETE request to `https://api.example.com/posts/789`
5. Prints the response status and body for each request

The script should handle all requests asynchronously and use modern JavaScript patterns.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: GET request to /users/123
Expected: Successfully fetches and displays user data

@test Input: POST request with JSON body {title: "Test", content: "Hello"}
Expected: Creates new resource and returns 201 status

@test Input: PUT request to /posts/456 with updated data
Expected: Updates resource and returns 200 status

@test Input: DELETE request to /posts/789
Expected: Deletes resource and returns 204 or 200 status
