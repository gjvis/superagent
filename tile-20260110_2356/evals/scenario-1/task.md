# Resilient API Client

Build an HTTP client module that makes requests with automatic retry logic, progress tracking, and custom success validation.

## Requirements

### Automatic Retry Logic

Implement retry behavior that automatically retries failed requests:

- Retry network timeout errors
- Retry server errors (HTTP status 500 and above)
- Do NOT retry client errors (HTTP status 400-499)
- Allow a maximum of 2 retry attempts
- Do not retry successful responses (HTTP status 200-299)

### Progress Tracking

Track and report progress for file operations:

- Report upload progress when sending files
- Report download progress when receiving responses
- Provide percentage complete and bytes transferred
- Support callback functions to receive progress updates

### Custom Success Validation

Support custom logic to determine if a response is successful:

- Allow providing custom validation functions
- The validation function receives the response and returns true for success or false for failure
- Default behavior treats HTTP status 200-299 as successful

## Implementation

[@generates](./src/client.js)

## Test Cases

- When a request fails with a network timeout error, it retries up to 2 times before failing [@test](./test/client.test.js)
- When a request returns a 503 status, it retries up to 2 times before failing [@test](./test/client.test.js)
- When a request returns a 404 status, it does NOT retry and fails immediately [@test](./test/client.test.js)
- When uploading a file, progress callbacks receive upload progress data with percent and loaded bytes [@test](./test/client.test.js)
- When custom validation logic is provided, it determines whether the response is treated as success [@test](./test/client.test.js)

## Dependencies { .dependencies }

### superagent { .dependency }

Provides HTTP client functionality with support for retries, progress monitoring, and custom validation.
