# API Search Tool

Create a search tool that queries an API with multiple filter parameters. The tool should construct proper query strings from filter objects.

## Requirements

Your implementation should:

1. Accept a base URL and a filters object containing search criteria
2. Build query strings from the filters object (supporting multiple key-value pairs)
3. Support multiple filter criteria in a single request
4. Allow filters to be added incrementally
5. Return the complete URL with query string and the response data

## Test Cases

- Searching with filters `{userId: 1}` on `https://jsonplaceholder.typicode.com/posts` returns only posts with userId=1 @test
- Searching with filters `{userId: 1, id: 5}` returns only the post matching both criteria @test
- Adding filters incrementally (first `{userId: 1}` then `{id: 5}`) produces the same query string as adding them together @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
