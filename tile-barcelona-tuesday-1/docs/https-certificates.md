# HTTPS and Certificates

Configure SSL/TLS certificates, custom HTTPS agents, and certificate validation for secure connections in SuperAgent. These features are **Node.js only** and provide comprehensive control over HTTPS request security, including support for self-signed certificates, mutual TLS authentication, and custom connection pooling.

## CA Certificates

Set a Certificate Authority (CA) certificate to validate the server's certificate. This is essential when connecting to servers with self-signed certificates or custom certificate authorities.

```javascript { .api }
/**
 * Set the certificate authority option for HTTPS requests
 * @param cert - CA certificate as Buffer or Array of Buffers
 * @returns Request instance for chaining
 */
ca(cert: Buffer | Buffer[]): Request;
```

### Usage

```javascript
import request from 'superagent';
import fs from 'fs';

// Load CA certificate from file
const caCert = fs.readFileSync('./ca-cert.pem');

// Use CA certificate for request
const res = await request
  .get('https://secure.example.com/api/data')
  .ca(caCert);

// Multiple CA certificates
const caCerts = [
  fs.readFileSync('./ca-cert-1.pem'),
  fs.readFileSync('./ca-cert-2.pem')
];

await request
  .get('https://secure.example.com/api/data')
  .ca(caCerts);
```

### Common Use Cases

**Self-Signed Certificates**: When connecting to a server with a self-signed certificate, you must provide the CA certificate to avoid certificate validation errors:

```javascript
// Self-signed certificate example
const caCert = fs.readFileSync('./self-signed-ca.pem');

try {
  const res = await request
    .get('https://localhost:8443/api/data')
    .ca(caCert);
  console.log(res.body);
} catch (err) {
  console.error('Certificate validation failed:', err.message);
}
```

**Internal Certificate Authorities**: For corporate or internal networks with custom CAs:

```javascript
// Internal CA certificate
const internalCA = fs.readFileSync('/etc/ssl/certs/internal-ca.pem');

await request
  .get('https://internal.company.com/api')
  .ca(internalCA);
```

**Development Environments**: Working with development servers that use self-signed certificates:

```javascript
// Development environment
const devCert = fs.readFileSync('./dev-ca-cert.pem');

const agent = require('superagent').agent();
agent.ca(devCert); // Set CA for all requests from this agent

await agent.get('https://dev.local:3000/api/users');
await agent.post('https://dev.local:3000/api/users').send({ name: 'Alice' });
```

## Client Certificates

Configure client certificates for mutual TLS (mTLS) authentication. This establishes two-way authentication where both the client and server verify each other's identity.

### Client Key

```javascript { .api }
/**
 * Set the client certificate key for HTTPS requests
 * @param cert - Private key as Buffer or string
 * @returns Request instance for chaining
 */
key(cert: Buffer | string): Request;
```

### Client Certificate

```javascript { .api }
/**
 * Set the client certificate for HTTPS requests
 * @param cert - Certificate as Buffer or string
 * @returns Request instance for chaining
 */
cert(cert: Buffer | string): Request;
```

### Usage

```javascript
import request from 'superagent';
import fs from 'fs';

// Load client certificate and key
const clientCert = fs.readFileSync('./client-cert.pem');
const clientKey = fs.readFileSync('./client-key.pem');

// Use client certificate for mutual TLS
const res = await request
  .get('https://secure.example.com/api/data')
  .cert(clientCert)
  .key(clientKey);

// With CA certificate for self-signed server cert
const caCert = fs.readFileSync('./ca-cert.pem');

await request
  .get('https://secure.example.com/api/data')
  .ca(caCert)
  .cert(clientCert)
  .key(clientKey);
```

### Mutual TLS Authentication

Mutual TLS provides strong authentication by requiring both parties to present certificates:

```javascript
// Complete mTLS setup
const caCert = fs.readFileSync('./ca-cert.pem');
const clientCert = fs.readFileSync('./client-cert.pem');
const clientKey = fs.readFileSync('./client-key.pem');

try {
  const res = await request
    .post('https://api.secure-service.com/transactions')
    .ca(caCert)
    .cert(clientCert)
    .key(clientKey)
    .send({ amount: 100, currency: 'USD' });

  console.log('Transaction completed:', res.body);
} catch (err) {
  if (err.code === 'UNABLE_TO_VERIFY_LEAF_SIGNATURE') {
    console.error('Client certificate validation failed');
  } else {
    console.error('Request failed:', err.message);
  }
}
```

### Using with Agent

For persistent connections with client certificates:

```javascript
const { agent } = require('superagent');

// Create agent with client certificates
const secureAgent = agent({
  ca: fs.readFileSync('./ca-cert.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem')
});

// All requests from this agent use the certificates
await secureAgent.get('https://secure.example.com/api/users');
await secureAgent.post('https://secure.example.com/api/users')
  .send({ name: 'Bob' });
```

## PFX/PKCS12 Certificates

Use PFX (PKCS#12) format certificates, which bundle the private key, certificate, and CA certificates into a single encrypted file.

```javascript { .api }
/**
 * Set the key, certificate, and CA certs of the client in PFX or PKCS12 format
 * @param cert - PFX certificate as Buffer or string, or object with pfx and passphrase
 * @returns Request instance for chaining
 */
pfx(cert: Buffer | string | { pfx: Buffer | string; passphrase: string }): Request;
```

### Usage

```javascript
import request from 'superagent';
import fs from 'fs';

// Load PFX certificate
const pfxCert = fs.readFileSync('./client-cert.pfx');

// Use PFX certificate without passphrase
await request
  .get('https://secure.example.com/api/data')
  .pfx(pfxCert);

// Use PFX certificate with passphrase
await request
  .get('https://secure.example.com/api/data')
  .pfx({
    pfx: pfxCert,
    passphrase: 'secret-password'
  });
```

### Converting and Using PFX Certificates

PFX files are commonly used in Windows environments and can be created from PEM files:

```javascript
// Using PFX from different sources
const pfxBuffer = fs.readFileSync('./certificate.pfx');

// Without password
await request
  .post('https://api.example.com/secure-endpoint')
  .pfx(pfxBuffer)
  .send({ data: 'secure data' });

// With password protection
const securePfx = {
  pfx: fs.readFileSync('./secure-cert.pfx'),
  passphrase: process.env.CERT_PASSWORD
};

await request
  .get('https://api.example.com/protected-resource')
  .pfx(securePfx);
```

### Using with Agent

Configure an agent to use PFX certificates for all requests:

```javascript
const { agent } = require('superagent');

// Create agent with PFX certificate
const pfxAgent = agent({
  pfx: fs.readFileSync('./certificate.pfx')
});

// Or with passphrase (note: pass pfx and passphrase separately to agent)
const secureAgent = agent({
  pfx: {
    pfx: fs.readFileSync('./certificate.pfx'),
    passphrase: 'password'
  }
});

// All requests use the PFX certificate
await secureAgent.get('https://secure.example.com/api/data');
await secureAgent.post('https://secure.example.com/api/submit')
  .send({ value: 42 });
```

## Custom HTTP Agent

Set a custom `http.Agent` or `https.Agent` for fine-grained control over connection pooling, keep-alive settings, and socket management. This is different from SuperAgent's `agent()` function, which creates a SuperAgent Agent for cookie management.

```javascript { .api }
/**
 * Gets/sets the Agent to use for this HTTP request
 * The default (if this function is not called) is to opt out of connection pooling (agent: false)
 * @param agent - Node.js http.Agent or https.Agent instance
 * @returns http.Agent when called without arguments, Request for chaining when setting
 */
agent(agent: http.Agent | https.Agent): Request;
agent(): http.Agent | https.Agent;
```

### Usage

```javascript
import request from 'superagent';
import https from 'https';
import fs from 'fs';

// Create custom HTTPS agent with specific settings
const httpsAgent = new https.Agent({
  keepAlive: true,
  keepAliveMsecs: 1000,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 60000,
  ca: fs.readFileSync('./ca-cert.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem')
});

// Use custom agent
await request
  .get('https://api.example.com/data')
  .agent(httpsAgent);

// Get current agent
const currentAgent = request
  .get('https://api.example.com/data')
  .agent();
```

### Connection Pooling

Control connection reuse and pooling behavior:

```javascript
import https from 'https';

// Agent with connection pooling enabled
const pooledAgent = new https.Agent({
  keepAlive: true,
  keepAliveMsecs: 3000,
  maxSockets: 100,        // Max sockets per host
  maxFreeSockets: 10,     // Max idle sockets per host
  maxTotalSockets: 200    // Max sockets across all hosts
});

// Reuse agent across multiple requests
await request
  .get('https://api.example.com/users')
  .agent(pooledAgent);

await request
  .get('https://api.example.com/posts')
  .agent(pooledAgent);
```

### Advanced HTTPS Configuration

Combine custom agent with certificate options for complete control:

```javascript
import https from 'https';
import fs from 'fs';

// Advanced HTTPS agent configuration
const secureAgent = new https.Agent({
  // Connection pooling
  keepAlive: true,
  maxSockets: 50,

  // Certificate configuration
  ca: fs.readFileSync('./ca-bundle.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem'),

  // Security options
  rejectUnauthorized: true,  // Enforce certificate validation
  minVersion: 'TLSv1.2',     // Minimum TLS version
  maxVersion: 'TLSv1.3',     // Maximum TLS version

  // Socket options
  timeout: 30000,
  scheduling: 'lifo'         // Last-in-first-out socket reuse
});

await request
  .post('https://secure-api.example.com/transactions')
  .agent(secureAgent)
  .send({ amount: 500 });
```

### Disabling Certificate Validation (Development Only)

For development and testing purposes only, you can disable certificate validation. **Never use this in production:**

```javascript
import https from 'https';

// WARNING: Only for development/testing!
const unsafeAgent = new https.Agent({
  rejectUnauthorized: false
});

await request
  .get('https://localhost:3000/api/test')
  .agent(unsafeAgent);
```

### HTTP/2 Agent Support

When using HTTP/2, you may need to configure agents differently:

```javascript
// For HTTP/2, certificate options are typically passed directly
// rather than through an agent
await request
  .get('https://http2.example.com/api/data')
  .http2()
  .ca(caCert)
  .cert(clientCert)
  .key(clientKey);
```

## Complete HTTPS Configuration Examples

### Self-Signed Certificate Server

Connect to a server with a self-signed certificate:

```javascript
import request from 'superagent';
import fs from 'fs';

const caCert = fs.readFileSync('./self-signed-ca.pem');

async function fetchFromSelfSigned() {
  try {
    const res = await request
      .get('https://localhost:8443/api/data')
      .ca(caCert);

    console.log('Success:', res.body);
  } catch (err) {
    if (err.code === 'DEPTH_ZERO_SELF_SIGNED_CERT') {
      console.error('Self-signed certificate not trusted');
    } else {
      console.error('Request failed:', err.message);
    }
  }
}
```

### Mutual TLS with Agent

Set up an agent for multiple requests with mutual TLS:

```javascript
import { agent } from 'superagent';
import fs from 'fs';

// Create secure agent with all certificates
const secureAgent = agent({
  ca: fs.readFileSync('./ca-cert.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem')
});

// Additional configuration
secureAgent
  .set('Authorization', 'Bearer token123')
  .timeout(30000)
  .retry(2);

// All requests use the certificates and defaults
async function secureFetch() {
  const users = await secureAgent.get('https://api.example.com/users');
  const posts = await secureAgent.get('https://api.example.com/posts');

  await secureAgent
    .post('https://api.example.com/users')
    .send({ name: 'Alice' });

  return { users, posts };
}
```

### Enterprise Configuration

Complete setup for enterprise environments with custom CA and connection pooling:

```javascript
import request from 'superagent';
import https from 'https';
import fs from 'fs';

// Load enterprise certificates
const enterpriseCerts = {
  ca: fs.readFileSync('/etc/ssl/certs/enterprise-ca.pem'),
  cert: fs.readFileSync('/etc/ssl/certs/client-cert.pem'),
  key: fs.readFileSync('/etc/ssl/private/client-key.pem')
};

// Create optimized HTTPS agent
const enterpriseAgent = new https.Agent({
  ...enterpriseCerts,
  keepAlive: true,
  keepAliveMsecs: 5000,
  maxSockets: 100,
  maxFreeSockets: 20,
  timeout: 60000,
  rejectUnauthorized: true,
  minVersion: 'TLSv1.2'
});

// Make requests
async function enterpriseRequest() {
  try {
    const res = await request
      .post('https://internal-api.company.com/data')
      .agent(enterpriseAgent)
      .set('X-API-Key', process.env.API_KEY)
      .timeout({ response: 30000, deadline: 60000 })
      .retry(3)
      .send({ query: 'data' });

    return res.body;
  } catch (err) {
    console.error('Enterprise request failed:', err.message);
    throw err;
  }
}
```

### PFX Certificate with Environment Variables

Secure configuration using environment variables:

```javascript
import request from 'superagent';
import fs from 'fs';

// Load PFX from secure location
const pfxPath = process.env.CLIENT_CERT_PATH || './cert.pfx';
const pfxPassword = process.env.CLIENT_CERT_PASSWORD;

const pfxConfig = {
  pfx: fs.readFileSync(pfxPath),
  passphrase: pfxPassword
};

async function securePfxRequest(endpoint, data) {
  try {
    return await request
      .post(`https://secure-api.example.com${endpoint}`)
      .pfx(pfxConfig)
      .send(data);
  } catch (err) {
    if (err.code === 'ERR_OSSL_PEM_NO_START_LINE') {
      console.error('Invalid PFX certificate format');
    } else if (err.code === 'ERR_OSSL_BAD_DECRYPT') {
      console.error('Incorrect PFX passphrase');
    } else {
      console.error('Request failed:', err.message);
    }
    throw err;
  }
}
```

### Testing with Self-Signed Certificates

Utility function for development/testing:

```javascript
import request from 'superagent';
import https from 'https';
import fs from 'fs';

function createTestAgent(useSelfSigned = true) {
  if (useSelfSigned && process.env.NODE_ENV === 'development') {
    // Load development CA
    const devCA = fs.readFileSync('./dev-ca.pem');

    return new https.Agent({
      ca: devCA,
      rejectUnauthorized: true  // Still validate against our CA
    });
  } else if (process.env.NODE_ENV === 'development') {
    // Unsafe mode - only for local testing
    console.warn('WARNING: Certificate validation disabled');
    return new https.Agent({
      rejectUnauthorized: false
    });
  }

  // Production mode - use system certificates
  return undefined; // Use default agent
}

async function apiRequest(endpoint, options = {}) {
  const testAgent = createTestAgent();

  const req = request
    .get(`https://localhost:3000${endpoint}`)
    .timeout(5000);

  if (testAgent) {
    req.agent(testAgent);
  }

  return await req;
}
```

## Error Handling

Common certificate-related errors and how to handle them:

```javascript
import request from 'superagent';

async function handleCertErrors() {
  try {
    await request.get('https://secure.example.com/api/data');
  } catch (err) {
    // Certificate validation errors
    if (err.code === 'UNABLE_TO_VERIFY_LEAF_SIGNATURE') {
      console.error('Server certificate could not be verified');
      console.error('Provide CA certificate using .ca() method');
    }
    else if (err.code === 'DEPTH_ZERO_SELF_SIGNED_CERT') {
      console.error('Server uses self-signed certificate');
      console.error('Provide CA certificate using .ca() method');
    }
    else if (err.code === 'CERT_HAS_EXPIRED') {
      console.error('Server certificate has expired');
    }
    else if (err.code === 'CERT_NOT_YET_VALID') {
      console.error('Server certificate is not yet valid');
    }
    // Client certificate errors
    else if (err.code === 'ERR_OSSL_PEM_NO_START_LINE') {
      console.error('Invalid certificate format');
      console.error('Ensure certificate is in PEM or PFX format');
    }
    else if (err.code === 'ERR_OSSL_BAD_DECRYPT') {
      console.error('Invalid PFX passphrase');
    }
    // Connection errors
    else if (err.code === 'ECONNREFUSED') {
      console.error('Connection refused - server may be down');
    }
    else if (err.code === 'ETIMEDOUT') {
      console.error('Connection timed out');
    }
    else {
      console.error('Unexpected error:', err.message);
    }
  }
}
```

## Best Practices

### Security Recommendations

1. **Always validate certificates in production**: Set `rejectUnauthorized: true` on custom agents
2. **Store certificates securely**: Use environment variables or secure vaults, never commit certificates to version control
3. **Rotate certificates regularly**: Implement certificate rotation policies
4. **Use strong TLS versions**: Set `minVersion: 'TLSv1.2'` or higher
5. **Handle certificate errors gracefully**: Provide clear error messages for certificate issues

### Performance Optimization

1. **Reuse agents**: Create a single agent instance and reuse it across requests
2. **Enable keep-alive**: Use `keepAlive: true` for connection pooling
3. **Configure socket limits**: Set appropriate `maxSockets` based on your needs
4. **Cache certificate files**: Load certificates once at startup, not per request

### Development vs Production

```javascript
import https from 'https';
import fs from 'fs';

function createAgent() {
  const isProduction = process.env.NODE_ENV === 'production';

  if (isProduction) {
    // Production: strict security
    return new https.Agent({
      ca: fs.readFileSync(process.env.CA_CERT_PATH),
      cert: fs.readFileSync(process.env.CLIENT_CERT_PATH),
      key: fs.readFileSync(process.env.CLIENT_KEY_PATH),
      rejectUnauthorized: true,
      minVersion: 'TLSv1.2',
      keepAlive: true,
      maxSockets: 100
    });
  } else {
    // Development: allow self-signed certificates
    return new https.Agent({
      ca: fs.readFileSync('./dev-ca.pem'),
      rejectUnauthorized: true,  // Still validate against dev CA
      keepAlive: true
    });
  }
}
```

## Platform Notes

All HTTPS and certificate configuration features documented here are **Node.js only** and are not available in browser environments. Browsers handle HTTPS and certificate validation automatically through their own certificate stores and security policies.

For browser environments, HTTPS security is managed by the browser and cannot be configured programmatically for security reasons.
