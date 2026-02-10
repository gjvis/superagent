# Authentication

HTTP authentication methods including Basic, Bearer, and auto authentication, plus SSL/TLS certificate support for Node.js.

## Capabilities

### HTTP Authentication

Set HTTP authentication credentials with support for multiple authentication types.

```javascript { .api }
/**
 * Set HTTP authentication
 * @param user - Username or token
 * @param pass - Password (not used for bearer auth)
 * @param options - Authentication options
 * @returns Request instance for chaining
 */
Request.prototype.auth(user: string, pass: string, options?: object): Request;
```

**Options object:**

```javascript { .api }
interface AuthOptions {
  type?: 'basic' | 'bearer' | 'auto';
}
```

**Usage Examples:**

```javascript
// Basic authentication (default)
request.get('https://api.example.com/users')
  .auth('username', 'password');

// Explicit basic authentication
request.get('https://api.example.com/users')
  .auth('username', 'password', { type: 'basic' });

// Bearer token authentication
request.get('https://api.example.com/users')
  .auth('my-access-token', '', { type: 'bearer' });
// Equivalent to setting header: Authorization: Bearer my-access-token

// Auto authentication (credentials sent only after 401 response)
request.get('https://api.example.com/users')
  .auth('username', 'password', { type: 'auto' });
```

### Bearer Token Authentication

Simplified bearer token authentication.

**Usage Examples:**

```javascript
// Using auth() with bearer type
request.get('https://api.example.com/users')
  .auth('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...', '', { type: 'bearer' });

// Manually setting Authorization header (alternative)
request.get('https://api.example.com/users')
  .set('Authorization', 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...');

// OAuth 2.0 access token
const accessToken = 'your-oauth-access-token';
request.get('https://api.example.com/protected')
  .auth(accessToken, '', { type: 'bearer' });
```

### API Key Authentication

Use custom headers for API key authentication.

**Usage Examples:**

```javascript
// API key in header
request.get('https://api.example.com/users')
  .set('X-API-Key', 'your-api-key-here');

// API key in query parameter
request.get('https://api.example.com/users')
  .query({ api_key: 'your-api-key-here' });

// Multiple authentication headers
request.get('https://api.example.com/users')
  .set('X-API-Key', 'your-api-key')
  .set('X-Client-ID', 'your-client-id');
```

### SSL/TLS Certificates (Node.js)

Configure SSL/TLS certificates for secure HTTPS connections.

```javascript { .api }
/**
 * Set CA certificate(s)
 * @param cert - CA certificate (string, Buffer, or array)
 * @returns Request instance for chaining
 */
Request.prototype.ca(cert: string | Buffer | Array): Request;

/**
 * Set client certificate
 * @param cert - Client certificate (string or Buffer)
 * @returns Request instance for chaining
 */
Request.prototype.cert(cert: string | Buffer): Request;

/**
 * Set private key
 * @param key - Private key (string or Buffer)
 * @returns Request instance for chaining
 */
Request.prototype.key(key: string | Buffer): Request;

/**
 * Set PFX or PKCS12 encoded private key and certificate
 * @param pfx - PFX certificate (string, Buffer, or options object)
 * @returns Request instance for chaining
 */
Request.prototype.pfx(pfx: string | Buffer | object): Request;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Custom CA certificate
const caCert = fs.readFileSync('/path/to/ca-cert.pem');
request.get('https://secure.example.com/api')
  .ca(caCert);

// Multiple CA certificates
const caCerts = [
  fs.readFileSync('/path/to/ca-cert-1.pem'),
  fs.readFileSync('/path/to/ca-cert-2.pem')
];
request.get('https://secure.example.com/api')
  .ca(caCerts);

// Client certificate authentication
const clientCert = fs.readFileSync('/path/to/client-cert.pem');
const clientKey = fs.readFileSync('/path/to/client-key.pem');

request.get('https://secure.example.com/api')
  .cert(clientCert)
  .key(clientKey);

// Using PFX/PKCS12 certificate
const pfxCert = fs.readFileSync('/path/to/certificate.pfx');
request.get('https://secure.example.com/api')
  .pfx(pfxCert);

// PFX with passphrase
request.get('https://secure.example.com/api')
  .pfx({
    pfx: fs.readFileSync('/path/to/certificate.pfx'),
    passphrase: 'secret-passphrase'
  });
```

### Certificate Configuration with Agent (Node.js)

Configure default SSL certificates for all requests using an Agent.

**Usage Examples:**

```javascript
const fs = require('fs');
const request = require('superagent');

// Create agent with SSL certificates
const agent = request.agent({
  ca: fs.readFileSync('/path/to/ca-cert.pem'),
  key: fs.readFileSync('/path/to/client-key.pem'),
  cert: fs.readFileSync('/path/to/client-cert.pem')
});

// All requests from this agent use the configured certificates
agent.get('https://secure.example.com/api/users')
  .then(res => console.log(res.body));

agent.post('https://secure.example.com/api/users')
  .send({ name: 'John' })
  .then(res => console.log(res.body));
```

## Authentication Examples

### JWT Authentication

```javascript
// Login to get JWT token
const loginRes = await request.post('https://api.example.com/auth/login')
  .send({ username: 'john', password: 'secret' });

const token = loginRes.body.token;

// Use JWT for subsequent requests
request.get('https://api.example.com/protected')
  .auth(token, '', { type: 'bearer' })
  .then(res => console.log(res.body));
```

### OAuth 2.0 Flow

```javascript
// After obtaining OAuth access token
const accessToken = 'your-oauth-access-token';

request.get('https://api.example.com/me')
  .auth(accessToken, '', { type: 'bearer' })
  .then(res => console.log('User profile:', res.body));

// Refresh token when needed
const refreshRes = await request.post('https://api.example.com/oauth/token')
  .send({
    grant_type: 'refresh_token',
    refresh_token: 'your-refresh-token',
    client_id: 'your-client-id',
    client_secret: 'your-client-secret'
  });

const newAccessToken = refreshRes.body.access_token;
```

### Digest Authentication

```javascript
// Note: SuperAgent doesn't have built-in digest auth support
// Use 'auto' type for basic/digest challenge-response
request.get('https://api.example.com/protected')
  .auth('username', 'password', { type: 'auto' })
  .then(res => console.log(res.body));

// Or use a plugin like superagent-digest
```

### Session-Based Authentication

```javascript
const request = require('superagent');

// Create agent to persist cookies
const agent = request.agent();

// Login (sets session cookie)
await agent.post('https://api.example.com/login')
  .send({ username: 'john', password: 'secret' });

// Subsequent requests automatically include session cookie
const res = await agent.get('https://api.example.com/profile');
console.log('Profile:', res.body);

// Logout
await agent.post('https://api.example.com/logout');
```

## Security Best Practices

### Avoid Hardcoding Credentials

```javascript
// Bad: hardcoded credentials
request.get('https://api.example.com/users')
  .auth('username', 'hardcoded-password');

// Good: use environment variables
const username = process.env.API_USERNAME;
const password = process.env.API_PASSWORD;

request.get('https://api.example.com/users')
  .auth(username, password);

// Good: use configuration files (not committed to version control)
const config = require('./config.json');
request.get('https://api.example.com/users')
  .auth(config.api.username, config.api.password);
```

### Secure Token Storage

```javascript
// Browser: avoid storing sensitive tokens in localStorage
// Use httpOnly cookies or sessionStorage for temporary tokens

// Node.js: store credentials securely
const keytar = require('keytar'); // Secure credential storage

async function getAuthToken() {
  const token = await keytar.getPassword('myapp', 'api-token');
  return token;
}

const token = await getAuthToken();
request.get('https://api.example.com/users')
  .auth(token, '', { type: 'bearer' });
```

### HTTPS Only

```javascript
// Always use HTTPS for authenticated requests
// Bad: credentials sent over HTTP
request.get('http://api.example.com/users')  // Insecure!
  .auth('username', 'password');

// Good: credentials sent over HTTPS
request.get('https://api.example.com/users')  // Secure
  .auth('username', 'password');
```
