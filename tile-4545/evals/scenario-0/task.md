# HTTP Request Builder

Build a command-line tool that makes HTTP requests to a REST API endpoint. The tool should support different request types and handle responses appropriately.

## Requirements

Your implementation should:

1. Accept a URL and HTTP method as command-line arguments
2. Support GET, POST, PUT, DELETE, and PATCH methods
3. For POST, PUT, and PATCH requests, accept JSON data from a command-line argument
4. Print the response status code and body to stdout
5. Handle both successful responses and error responses gracefully

## Test Cases

- Making a GET request to `https://jsonplaceholder.typicode.com/posts/1` returns status 200 and post data @test
- Making a POST request to `https://jsonplaceholder.typicode.com/posts` with JSON data `{"title":"test","body":"content","userId":1}` returns status 201 @test
- Making a DELETE request to `https://jsonplaceholder.typicode.com/posts/1` returns status 200 @test
- Making a PUT request to `https://jsonplaceholder.typicode.com/posts/1` with JSON data `{"title":"updated","body":"new content","userId":1}` returns status 200 @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
