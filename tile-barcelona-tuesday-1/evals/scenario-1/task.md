# API Client with Session Management

Build an authenticated API client that handles session management, automatic retries, and request timeouts.

## Requirements

Your task is to implement an API client class that:

1. **Session Management**: Maintains authentication state across multiple requests without requiring credentials for each call
2. **Automatic Retry**: Handles transient failures by automatically retrying failed requests
3. **Timeout Configuration**: Implements different timeout strategies for different request types
4. **Login Flow**: Authenticates with the API and maintains the session for subsequent requests

## Specifications

### APIClient Class

Create a class `APIClient` that manages authenticated API sessions.

**Constructor**: `new APIClient(baseUrl)`
- `baseUrl`: The base URL for all API requests (e.g., `http://api.example.com`)

**Methods**:

1. `login(credentials)`: Authenticates with the API
   - `credentials`: Object with `username` and `password` fields
   - Returns a Promise that resolves when login succeeds
   - Stores authentication state for subsequent requests
   - Endpoint: `POST /auth/login`

2. `getProfile()`: Retrieves the authenticated user's profile
   - Returns a Promise that resolves with the profile data
   - Must be called after successful login
   - Endpoint: `GET /users/me`

3. `updateProfile(data)`: Updates the user's profile
   - `data`: Object containing profile fields to update
   - Returns a Promise that resolves with the updated profile
   - Must be called after successful login
   - Endpoint: `PUT /users/me`

4. `fetchData(endpoint)`: Makes a GET request to any endpoint
   - `endpoint`: The API endpoint path (e.g., `/data/items`)
   - Returns a Promise that resolves with the response data
   - Must handle transient failures automatically
   - Should use appropriate timeout settings

### Behavioral Requirements

1. **Session Persistence**: After login, all subsequent requests should automatically include authentication state without passing credentials
2. **Retry Logic**: Failed requests should retry up to 3 times on network errors or 5xx server responses
3. **Timeout Configuration**:
   - Login and profile requests: 5 second total timeout
   - Data fetch requests: 10 second response timeout with no overall deadline (to handle large responses)
4. **Error Handling**: All methods should reject with meaningful errors when operations fail after retries

[@generates](./src/api-client.js)

## API Specification

```javascript { #api }
class APIClient {
  constructor(baseUrl);
  login(credentials);
  getProfile();
  updateProfile(data);
  fetchData(endpoint);
}

module.exports = APIClient;
```

## Test Cases

- It creates an API client instance [@test](../test/api-client.test.js)
- It logs in and maintains session for subsequent requests [@test](../test/api-client.test.js)
- It retrieves user profile after authentication [@test](../test/api-client.test.js)
- It updates user profile with new data [@test](../test/api-client.test.js)
- It automatically retries on server errors [@test](../test/api-client.test.js)

## Dependencies { .dependencies }

### superagent { .dependency }

Provides HTTP request functionality with session management, retry logic, and timeout configuration.
