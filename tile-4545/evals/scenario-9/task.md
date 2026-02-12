# Configured API Client

Build an API client with pre-configured default settings that apply to all requests, avoiding repetitive configuration.

## Requirements

Your implementation should:

1. Create a client with default headers that apply to all requests
2. Set default authentication credentials at the client level
3. Configure default timeout settings for all requests
4. Make multiple requests that all inherit these default settings
5. Allow individual requests to override defaults when needed

## Test Cases

- All requests through a configured client include the default headers without explicitly setting them @test
- Default authentication is applied to all requests made through the client @test
- Individual requests can override default settings while still inheriting other defaults @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
