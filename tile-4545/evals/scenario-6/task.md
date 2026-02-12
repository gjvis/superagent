# Robust API Client

Create an API client with comprehensive error handling that properly processes both successful and failed responses.

## Requirements

Your implementation should:

1. Handle HTTP error responses (4xx, 5xx status codes) gracefully
2. Extract error details from error objects (status code, response body)
3. Implement custom response validation logic that treats certain status codes as success
4. Distinguish between network errors and HTTP errors
5. Provide meaningful error messages based on the error type

## Test Cases

- A 404 response is caught as an error with status code 404 available @test
- Custom validation logic can treat a 404 response as successful using a validation callback @test
- Error objects contain both the status code and response data @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
