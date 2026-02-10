# Create New Blog Post

Build a function that creates a new blog post by sending data to a REST API.

## Requirements

Create a function `createBlogPost(postData)` that:

- Accepts an object containing post data (title, content, author, tags)
- Makes an HTTP POST request to `https://api.example.com/posts`
- Sends the post data as JSON in the request body
- Returns a promise that resolves with the created post including the assigned ID

## Test Cases

- Calling `createBlogPost({title: 'Hello', content: 'World', author: 'John', tags: ['intro']})` should POST the data as JSON and return the created post @test
- The Content-Type header should be automatically set to application/json @test
- Calling `createBlogPost({title: 'Test', content: 'Content'})` with partial data should still send valid JSON @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
