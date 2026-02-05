# Authentication

SuperAgent provides flexible authentication support for various authentication methods through the `.auth()` method. The library supports HTTP Basic authentication, Bearer token authentication, and auto authentication modes, with implementations in both Node.js and browser environments.

## Core Concepts

SuperAgent's authentication system is designed to simplify the process of adding authorization headers to HTTP requests. The `.auth()` method provides a fluent interface that automatically formats credentials according to the authentication type, eliminating the need to manually construct Authorization headers.

The authentication method is available on both individual Request instances and Agent instances, allowing you to set default authentication credentials that persist across multiple requests.

## Capabilities

### Basic Authentication

HTTP Basic authentication encodes credentials as Base64 and sets the Authorization header with the "Basic" scheme.

```javascript { .api }
/**
 * Set HTTP Basic authentication
 * @param user - Username
 * @param pass - Password
 * @param options - Optional configuration object (defaults to { type: 'basic' })
 * @returns Request instance for chaining
 */
auth(user: string, pass: string, options?: { type: 'basic' }): Request;
```

**Examples:**

```javascript
// Basic authentication with username and password
await request
  .get('/api/protected')
  .auth('username', 'password');

// Explicitly specify basic type
await request
  .post('/api/users')
  .auth('admin', 'secret123', { type: 'basic' });

// Basic auth with empty password
await request
  .get('/api/data')
  .auth('apikey');
```

**Implementation Details:**

The `.auth()` method with basic authentication performs the following steps:

1. Concatenates the username and password with a colon separator: `user:pass`
2. Encodes the concatenated string using Base64
3. Sets the Authorization header: `Authorization: Basic <base64-encoded-credentials>`

In Node.js, Base64 encoding uses `Buffer.from(string).toString('base64')`. In browsers, it uses the `btoa()` function when available.

### Bearer Token Authentication

Bearer token authentication is commonly used for OAuth 2.0 and JWT-based authentication. It sets the Authorization header with the "Bearer" scheme.

```javascript { .api }
/**
 * Set Bearer token authentication
 * @param token - Access token or JWT
 * @param options - Configuration object with type: 'bearer'
 * @returns Request instance for chaining
 */
auth(token: string, options: { type: 'bearer' }): Request;
```

**Examples:**

```javascript
// Bearer token authentication
await request
  .get('/api/user/profile')
  .auth('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...', { type: 'bearer' });

// OAuth 2.0 access token
const accessToken = 'ya29.a0AfH6SMBx...';
await request
  .post('/api/data')
  .send({ name: 'example' })
  .auth(accessToken, { type: 'bearer' });

// Using with async/await
const token = await getAuthToken();
const response = await request
  .get('/api/protected-resource')
  .auth(token, { type: 'bearer' });
```

**Implementation Details:**

For bearer token authentication:

1. The token is used as-is without encoding
2. The Authorization header is set: `Authorization: Bearer <token>`
3. The second parameter (pass) is ignored when type is 'bearer'

You can pass the options object as the second parameter, omitting the password parameter entirely:

```javascript
// These are equivalent:
request.get('/api/data').auth(token, '', { type: 'bearer' });
request.get('/api/data').auth(token, { type: 'bearer' });
```

### Auto Authentication

Auto authentication mode stores credentials on the request object for later use, rather than immediately setting the Authorization header. This is primarily used in browser environments where `btoa()` may not be available.

```javascript { .api }
/**
 * Set auto authentication (stores credentials for later use)
 * @param user - Username
 * @param pass - Password (optional)
 * @param options - Configuration object with type: 'auto'
 * @returns Request instance for chaining
 */
auth(user: string, pass?: string, options: { type: 'auto' }): Request;
```

**Examples:**

```javascript
// Auto authentication (browser environment without btoa)
request
  .get('/api/data')
  .auth('username', 'password', { type: 'auto' })
  .end((err, res) => {
    // Credentials stored on request.username and request.password
  });
```

**Implementation Details:**

When using auto authentication:

1. The username is stored in `request.username`
2. The password is stored in `request.password`
3. No Authorization header is set immediately
4. The credentials are available for the underlying HTTP client to use

In browser environments, if `btoa()` is not available, the default authentication type is 'auto'. In Node.js, the default is 'basic'.

### Single Argument Usage

When only one argument is provided to `.auth()`, the password defaults to an empty string.

```javascript { .api }
/**
 * Set authentication with only username (password defaults to empty string)
 * @param user - Username or credentials string
 * @returns Request instance for chaining
 */
auth(user: string): Request;
```

**Examples:**

```javascript
// Username only (password is empty string)
await request
  .get('/api/data')
  .auth('apikey');

// Equivalent to:
await request
  .get('/api/data')
  .auth('apikey', '');

// Can also use with colon-separated credentials
await request
  .get('/api/users')
  .auth('tobi:learnboost'); // username is "tobi:learnboost", password is ""
```

This pattern is useful when:
- Using API keys that don't require a password
- The authentication system expects credentials in the username field only
- Working with custom authentication schemes that use the username field

### URL-Embedded Credentials

SuperAgent automatically handles credentials embedded in URLs using the standard `user:pass@host` format.

**Examples:**

```javascript
// Credentials in URL (automatically extracted and used)
await request.get('https://username:password@api.example.com/data');

// Node.js URL parsing example
const URL = require('url');
const parsedUrl = URL.parse('https://api.example.com/endpoint');
parsedUrl.auth = 'username:password';
await request.get(URL.format(parsedUrl));
```

**Implementation Details:**

When credentials are present in the URL:
1. SuperAgent automatically extracts the `user:pass` portion
2. Basic authentication is automatically applied
3. The credentials are removed from the URL sent to the server
4. The Authorization header is added with the Basic scheme

## Authentication with Agents

Agents allow you to set default authentication credentials that apply to all requests made through the agent.

```javascript { .api }
/**
 * Agent authentication configuration
 */
interface Agent {
  /**
   * Set default authentication for all requests through this agent
   * @param user - Username or token
   * @param pass - Password (optional)
   * @param options - Auth configuration
   * @returns Agent instance for chaining
   */
  auth(user: string, pass?: string, options?: { type: 'basic' | 'bearer' | 'auto' }): Agent;
}
```

**Examples:**

```javascript
// Create agent with default basic authentication
const agent = request.agent();
agent.auth('username', 'password');

// All requests through this agent include authentication
await agent.get('/api/users'); // Includes auth header
await agent.post('/api/data').send({ value: 123 }); // Includes auth header

// Agent with bearer token
const apiAgent = request.agent();
apiAgent.auth('my-access-token', { type: 'bearer' });

await apiAgent.get('/api/profile');
await apiAgent.get('/api/settings');

// Override agent auth on individual request
await apiAgent
  .get('/api/special')
  .auth('different-token', { type: 'bearer' }); // Uses different token
```

**Implementation Details:**

When authentication is set on an agent:
1. The credentials are stored in the agent's configuration
2. Each request created by the agent automatically inherits the authentication settings
3. Individual requests can override the agent's authentication
4. The agent persists authentication across all requests until changed

## Complete Type Definitions

```javascript { .api }
/**
 * Authentication method signature
 */
interface Request {
  /**
   * Set Authorization header with various authentication types
   *
   * @param user - Username (for basic/auto) or token (for bearer)
   * @param pass - Password (optional, defaults to empty string if omitted)
   * @param options - Authentication options
   * @param options.type - Authentication type: 'basic', 'bearer', or 'auto'
   *   - 'basic': HTTP Basic authentication (default) - encodes user:pass as Base64
   *   - 'bearer': Bearer token authentication - sets token in Authorization header
   *   - 'auto': Stores credentials on request object without setting header immediately
   * @returns Request instance for method chaining
   *
   * @example
   * // Basic authentication
   * request.get('/api').auth('user', 'pass')
   *
   * @example
   * // Bearer token
   * request.get('/api').auth('token123', { type: 'bearer' })
   *
   * @example
   * // Username only (password defaults to '')
   * request.get('/api').auth('apikey')
   */
  auth(user: string, pass?: string, options?: AuthOptions): Request;
}

interface AuthOptions {
  /**
   * Authentication type
   * - 'basic': HTTP Basic authentication (default in Node.js)
   * - 'bearer': Bearer token authentication
   * - 'auto': Automatic - stores credentials without encoding (default in browsers without btoa)
   */
  type?: 'basic' | 'bearer' | 'auto';
}

/**
 * Internal authentication implementation
 * (Implemented in RequestBase prototype)
 */
interface RequestBase {
  /**
   * Internal auth implementation
   * @private
   */
  _auth(
    user: string,
    pass: string,
    options: { type: 'basic' | 'bearer' | 'auto' },
    base64Encoder: (str: string) => string
  ): Request;
}
```

## Platform Differences

### Node.js Implementation

In Node.js (`lib/node/index.js`):

```javascript
// Default authentication type is 'basic'
// Uses Buffer for Base64 encoding
Request.prototype.auth = function(user, pass, options) {
  if (1 === arguments.length) pass = '';
  if (typeof pass === 'object' && pass !== null) {
    options = pass;
    pass = '';
  }
  if (!options) {
    options = { type: 'basic' };
  }

  const encoder = string => new Buffer.from(string).toString('base64');

  return this._auth(user, pass, options, encoder);
};
```

**Node.js Features:**
- Uses `Buffer.from()` for Base64 encoding
- Default type is always 'basic'
- Full support for all authentication types
- Works with HTTPS client certificates

### Browser Implementation

In browsers (`lib/client.js`):

```javascript
// Default authentication type is 'auto' if btoa is not available
// Uses btoa() for Base64 encoding when available
Request.prototype.auth = function(user, pass, options) {
  if (1 === arguments.length) pass = '';
  if (typeof pass === 'object' && pass !== null) {
    options = pass;
    pass = '';
  }
  if (!options) {
    options = {
      type: 'function' === typeof btoa ? 'basic' : 'auto',
    };
  }

  const encoder = string => {
    if ('function' === typeof btoa) {
      return btoa(string);
    }
    throw new Error('Cannot use basic auth, btoa is not a function');
  };

  return this._auth(user, pass, options, encoder);
};
```

**Browser Features:**
- Uses `btoa()` for Base64 encoding
- Default type is 'auto' if `btoa()` is not available (legacy browsers)
- Throws error if basic auth is attempted without `btoa()` support
- Bearer token authentication works in all browsers

## Common Patterns

### API Key Authentication

Many APIs use API keys passed in the username field with an empty password:

```javascript
// API key as username
await request
  .get('https://api.service.com/data')
  .auth('sk_live_abc123xyz789');

// API key with empty password (explicit)
await request
  .get('https://api.service.com/data')
  .auth('sk_live_abc123xyz789', '');
```

### JWT Authentication

JSON Web Tokens are typically used with Bearer authentication:

```javascript
// JWT token authentication
const jwt = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c';

await request
  .get('/api/protected')
  .auth(jwt, { type: 'bearer' });
```

### OAuth 2.0 Flow

Combining authentication with OAuth 2.0 access tokens:

```javascript
// Get access token
const tokenResponse = await request
  .post('https://oauth.example.com/token')
  .auth('client_id', 'client_secret') // Basic auth for token request
  .send({
    grant_type: 'client_credentials',
    scope: 'read write'
  });

const accessToken = tokenResponse.body.access_token;

// Use access token for API requests
await request
  .get('https://api.example.com/data')
  .auth(accessToken, { type: 'bearer' });
```

### Persistent Authentication with Agent

Create a reusable agent with authentication:

```javascript
// Create authenticated agent
const authenticatedAgent = request.agent();

// Login and save session
await authenticatedAgent
  .post('/api/login')
  .auth('user@example.com', 'password')
  .send({ remember: true });

// Agent now has session cookies + can set default auth
authenticatedAgent.auth('session-token', { type: 'bearer' });

// All subsequent requests include both cookies and auth
await authenticatedAgent.get('/api/profile');
await authenticatedAgent.get('/api/settings');
await authenticatedAgent.post('/api/data').send({ value: 123 });
```

### Conditional Authentication

Apply authentication based on environment or conditions:

```javascript
const isDevelopment = process.env.NODE_ENV === 'development';
const apiRequest = request.get('/api/data');

if (isDevelopment) {
  // Use test credentials in development
  apiRequest.auth('test-user', 'test-pass');
} else {
  // Use token in production
  const token = process.env.API_TOKEN;
  apiRequest.auth(token, { type: 'bearer' });
}

const response = await apiRequest;
```

### Retry with Re-authentication

Handle token expiration by re-authenticating:

```javascript
async function fetchWithAuth(url) {
  let token = getStoredToken();

  try {
    return await request
      .get(url)
      .auth(token, { type: 'bearer' });
  } catch (err) {
    if (err.status === 401) {
      // Token expired, refresh it
      token = await refreshAuthToken();

      // Retry with new token
      return await request
        .get(url)
        .auth(token, { type: 'bearer' });
    }
    throw err;
  }
}
```

## Security Considerations

### HTTPS Required

Always use HTTPS when sending authentication credentials to prevent interception:

```javascript
// Good - HTTPS protects credentials
await request
  .get('https://api.example.com/data')
  .auth('username', 'password');

// Bad - HTTP sends credentials in clear text
// await request
//   .get('http://api.example.com/data')
//   .auth('username', 'password');
```

### Credential Storage

Never hardcode credentials in source code. Use environment variables or secure storage:

```javascript
// Good - credentials from environment
await request
  .get('/api/data')
  .auth(process.env.API_USER, process.env.API_PASSWORD);

// Bad - hardcoded credentials
// await request
//   .get('/api/data')
//   .auth('hardcoded-user', 'hardcoded-pass');
```

### Token Management

Handle tokens securely and refresh them before expiration:

```javascript
class AuthManager {
  constructor() {
    this.token = null;
    this.tokenExpiry = null;
  }

  async getValidToken() {
    // Check if token is still valid
    if (this.token && this.tokenExpiry > Date.now()) {
      return this.token;
    }

    // Refresh token if needed
    const response = await request
      .post('/oauth/token')
      .auth(CLIENT_ID, CLIENT_SECRET)
      .send({ grant_type: 'client_credentials' });

    this.token = response.body.access_token;
    this.tokenExpiry = Date.now() + (response.body.expires_in * 1000);

    return this.token;
  }

  async makeAuthenticatedRequest(url) {
    const token = await this.getValidToken();
    return request
      .get(url)
      .auth(token, { type: 'bearer' });
  }
}
```

### Sensitive Data in Logs

Be careful not to log authentication headers or credentials:

```javascript
// The auth() method only sets headers, they won't be in logs by default
const response = await request
  .get('/api/data')
  .auth(username, password);

// But don't log the full request object which may contain headers
// console.log(request.toJSON()); // May expose credentials
```

## Error Handling

Handle authentication errors appropriately:

```javascript
try {
  const response = await request
    .get('/api/protected')
    .auth('username', 'password');

  console.log(response.body);
} catch (err) {
  if (err.status === 401) {
    console.error('Authentication failed: Invalid credentials');
  } else if (err.status === 403) {
    console.error('Access forbidden: Insufficient permissions');
  } else {
    console.error('Request failed:', err.message);
  }
}
```

Common authentication-related status codes:
- `401 Unauthorized` - Invalid or missing credentials
- `403 Forbidden` - Valid credentials but insufficient permissions
- `407 Proxy Authentication Required` - Proxy authentication needed

## Related Methods

Authentication is often used in combination with other SuperAgent features:

```javascript
// Authentication + custom headers
await request
  .post('/api/data')
  .auth('token', { type: 'bearer' })
  .set('X-API-Version', '2.0')
  .set('X-Client-ID', 'mobile-app')
  .send({ data: 'value' });

// Authentication + timeout + retry
await request
  .get('/api/data')
  .auth('username', 'password')
  .timeout({ response: 5000, deadline: 10000 })
  .retry(2);

// Authentication + query parameters
await request
  .get('/api/search')
  .auth('api-key')
  .query({ q: 'nodejs', limit: 10 });
```

## Summary

SuperAgent's `.auth()` method provides a clean, fluent interface for handling various authentication schemes:

- **Basic Authentication**: Username and password encoded as Base64
- **Bearer Token**: OAuth 2.0 and JWT token support
- **Auto Authentication**: Flexible credential storage for special cases
- **Agent Support**: Persistent authentication across requests
- **Cross-Platform**: Works in both Node.js and browser environments

The method automatically handles encoding, header formatting, and platform differences, making authentication simple and secure across different environments and authentication types.
