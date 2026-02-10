# Fetch Multiple Resources Sequentially

Build an async function that fetches multiple related resources in sequence.

## Requirements

Create an async function `fetchUserWithPosts(userId)` that:

- Accepts a user ID
- First fetches user data from `https://api.example.com/users/{userId}`
- Then fetches that user's posts from `https://api.example.com/users/{userId}/posts`
- Returns an object containing both the user data and their posts array
- Uses modern async/await syntax for cleaner code

## Test Cases

- Calling `await fetchUserWithPosts(5)` should fetch user 5 and their posts, returning `{user: {...}, posts: [...]}` @test
- The function should properly await each request before making the next one @test
- The function should handle both successful responses correctly @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
