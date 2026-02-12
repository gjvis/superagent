# Resilient API Client

Create an API client that handles transient failures gracefully by automatically retrying failed requests.

## Requirements

Your implementation should:

1. Configure requests to retry automatically on failure
2. Set the number of retry attempts
3. Handle both network errors and 5xx server errors with retries
4. Implement custom retry logic that only retries specific error conditions
5. Track and report the number of retry attempts made

## Test Cases

- A request configured with 3 retries attempts the request up to 4 times total (1 original + 3 retries) before failing @test
- A request that encounters a 503 error automatically retries @test
- Custom retry logic can selectively retry based on error type (e.g., retry on 503 but not on 404) @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
