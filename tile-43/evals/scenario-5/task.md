# Upload User Avatar Image

Build a function that uploads an image file to update a user's avatar.

## Requirements

Create a function `uploadAvatar(userId, filePath)` that:

- Accepts a user ID and a file path to an image
- Makes an HTTP POST request to `https://api.example.com/users/{userId}/avatar`
- Uploads the file using multipart/form-data encoding
- The file should be uploaded with the form field name "avatar"
- Returns a promise that resolves with the updated user object

## Test Cases

- Calling `uploadAvatar(123, '/tmp/photo.jpg')` should upload photo.jpg to the avatar field for user 123 @test
- Calling `uploadAvatar(456, './images/profile.png')` should handle different file paths and formats @test
- The request should automatically use multipart/form-data encoding @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
