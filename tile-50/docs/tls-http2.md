# TLS/SSL and HTTP/2 (Node.js)

Configure TLS certificates and enable HTTP/2 protocol. Available in Node.js only.

## Capabilities

### TLS/SSL Certificate Configuration

Configure client certificates and CA certificates for secure connections.

```javascript { .api }
/**
 * Set CA certificate (Certificate Authority)
 * @param cert - Certificate as string or Buffer
 * @returns Request for chaining
 */
request.get(url).ca(cert: string | Buffer): Request;

/**
 * Set client private key
 * @param cert - Private key as string or Buffer
 * @returns Request for chaining
 */
request.get(url).key(cert: string | Buffer): Request;

/**
 * Set client certificate
 * @param cert - Certificate as string or Buffer
 * @returns Request for chaining
 */
request.get(url).cert(cert: string | Buffer): Request;

/**
 * Set PFX or PKCS12 certificate
 * @param cert - Certificate as string/Buffer or {pfx: Buffer, passphrase: string}
 * @returns Request for chaining
 */
request.get(url).pfx(cert: string | Buffer | {pfx: Buffer, passphrase: string}): Request;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Set CA certificate
request
  .get('https://secure-api.example.com/data')
  .ca(fs.readFileSync('ca-cert.pem'))
  .then(res => console.log(res.body));

// Client certificate authentication
request
  .get('https://api.example.com/secure')
  .key(fs.readFileSync('client-key.pem'))
  .cert(fs.readFileSync('client-cert.pem'))
  .then(res => console.log(res.body));

// Complete TLS configuration
request
  .get('https://api.example.com/data')
  .ca(fs.readFileSync('ca-cert.pem'))
  .key(fs.readFileSync('client-key.pem'))
  .cert(fs.readFileSync('client-cert.pem'))
  .then(res => console.log(res.body));

// PFX certificate
request
  .get('https://api.example.com/data')
  .pfx(fs.readFileSync('certificate.pfx'))
  .then(res => console.log(res.body));

// PFX with passphrase
request
  .get('https://api.example.com/data')
  .pfx({
    pfx: fs.readFileSync('certificate.pfx'),
    passphrase: 'secret-password'
  })
  .then(res => console.log(res.body));

// Multiple CA certificates
const ca = [
  fs.readFileSync('ca1.pem'),
  fs.readFileSync('ca2.pem'),
  fs.readFileSync('ca3.pem')
];

request
  .get('https://api.example.com/data')
  .ca(ca)
  .then(res => console.log(res.body));
```

### HTTP/2 Protocol

Enable HTTP/2 for improved performance.

```javascript { .api }
/**
 * Enable HTTP/2 protocol (Node.js only)
 * Requires server support for HTTP/2
 * @param enable - Enable flag (default: true)
 * @returns Request for chaining
 */
request.get(url).http2(enable?: boolean): Request;
```

**Usage Examples:**

```javascript
// Enable HTTP/2
request
  .get('https://http2-server.com/api/data')
  .http2()
  .then(res => console.log(res.body));

// Explicit enable
request
  .get('https://http2-server.com/api/data')
  .http2(true)
  .then(res => console.log(res.body));

// Disable HTTP/2 (use HTTP/1.1)
request
  .get('https://http2-server.com/api/data')
  .http2(false)
  .then(res => console.log(res.body));

// HTTP/2 with TLS
request
  .get('https://http2-server.com/api/data')
  .http2()
  .ca(fs.readFileSync('ca-cert.pem'))
  .then(res => console.log(res.body));
```

## TLS Configuration with Agent

Set TLS configuration as defaults for all requests from an agent.

```javascript
const fs = require('fs');

// Create agent with TLS configuration
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem')
});

// All requests use these certificates
await agent.get('https://api.example.com/users');
await agent.get('https://api.example.com/posts');
await agent.post('https://api.example.com/data').send({ value: 123 });
```

## Common TLS Patterns

### Self-Signed Certificates

```javascript
const fs = require('fs');

// Accept self-signed certificate
request
  .get('https://localhost:8443/api/data')
  .ca(fs.readFileSync('self-signed-ca.pem'))
  .then(res => console.log(res.body))
  .catch(err => console.error('TLS error:', err));
```

### Mutual TLS (mTLS)

```javascript
const fs = require('fs');

// Client and server authenticate each other
request
  .get('https://api.example.com/secure')
  .ca(fs.readFileSync('server-ca.pem'))     // Verify server certificate
  .key(fs.readFileSync('client-key.pem'))   // Client private key
  .cert(fs.readFileSync('client-cert.pem')) // Client certificate
  .then(res => console.log('Authenticated:', res.body))
  .catch(err => console.error('Auth failed:', err));
```

### Multiple Environments

```javascript
const fs = require('fs');

function createSecureRequest(url, environment) {
  const certs = {
    development: {
      ca: fs.readFileSync('dev-ca.pem'),
      key: fs.readFileSync('dev-key.pem'),
      cert: fs.readFileSync('dev-cert.pem')
    },
    production: {
      ca: fs.readFileSync('prod-ca.pem'),
      key: fs.readFileSync('prod-key.pem'),
      cert: fs.readFileSync('prod-cert.pem')
    }
  };

  const config = certs[environment];

  return request
    .get(url)
    .ca(config.ca)
    .key(config.key)
    .cert(config.cert);
}

// Usage
const devData = await createSecureRequest('/api/data', 'development');
const prodData = await createSecureRequest('/api/data', 'production');
```

### Certificate Validation

```javascript
const fs = require('fs');

// Strict certificate validation
request
  .get('https://api.example.com/data')
  .ca(fs.readFileSync('ca-bundle.pem'))
  .then(res => console.log('Valid certificate'))
  .catch(err => {
    if (err.code === 'UNABLE_TO_VERIFY_LEAF_SIGNATURE') {
      console.error('Certificate validation failed');
    } else if (err.code === 'CERT_HAS_EXPIRED') {
      console.error('Certificate expired');
    } else {
      console.error('TLS error:', err.code);
    }
  });
```

## HTTP/2 Benefits

HTTP/2 provides several performance improvements:

- **Multiplexing**: Multiple requests over single connection
- **Header Compression**: Reduced overhead
- **Server Push**: Server can send resources proactively
- **Binary Protocol**: More efficient parsing

```javascript
// HTTP/2 performance example
async function fetchMultiple() {
  // Multiple requests over single HTTP/2 connection
  const promises = [];
  for (let i = 0; i < 10; i++) {
    promises.push(
      request
        .get(`https://http2-server.com/api/data/${i}`)
        .http2()
    );
  }

  // All requests use same connection (multiplexing)
  const results = await Promise.all(promises);
  return results.map(r => r.body);
}
```

## HTTP/2 with Agent

```javascript
const agent = request.agent();

// Enable HTTP/2 for all requests
agent.http2();

// All requests use HTTP/2
await agent.get('https://http2-server.com/api/users');
await agent.get('https://http2-server.com/api/posts');
```

## TLS Configuration Reference

### Certificate Formats

SuperAgent accepts certificates in various formats:

- **PEM**: Text format with -----BEGIN CERTIFICATE----- markers
- **DER**: Binary format
- **PFX/PKCS12**: Password-protected bundle with key and certificate

```javascript
// PEM format (most common)
request
  .get(url)
  .cert(fs.readFileSync('cert.pem', 'utf8'));

// DER format
request
  .get(url)
  .cert(fs.readFileSync('cert.der'));

// PFX format
request
  .get(url)
  .pfx(fs.readFileSync('cert.pfx'));
```

### Certificate Loading

```javascript
const fs = require('fs');

// Load from file
const ca = fs.readFileSync('ca.pem');

// Load from string
const caString = `
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAKL...
-----END CERTIFICATE-----
`;

// Load multiple certificates
const caBundle = [
  fs.readFileSync('ca1.pem'),
  fs.readFileSync('ca2.pem'),
  fs.readFileSync('ca3.pem')
];

request
  .get(url)
  .ca(caBundle);
```

## Error Handling

```javascript
request
  .get('https://secure-api.example.com/data')
  .ca(fs.readFileSync('ca.pem'))
  .key(fs.readFileSync('key.pem'))
  .cert(fs.readFileSync('cert.pem'))
  .catch(err => {
    // Common TLS errors
    if (err.code === 'UNABLE_TO_VERIFY_LEAF_SIGNATURE') {
      console.error('Invalid certificate chain');
    } else if (err.code === 'CERT_HAS_EXPIRED') {
      console.error('Certificate expired');
    } else if (err.code === 'DEPTH_ZERO_SELF_SIGNED_CERT') {
      console.error('Self-signed certificate');
    } else if (err.code === 'UNABLE_TO_GET_ISSUER_CERT') {
      console.error('Cannot verify certificate issuer');
    } else {
      console.error('TLS error:', err.code, err.message);
    }
  });
```

## Best Practices

### Secure Certificate Storage

```javascript
// Don't commit certificates to repository
// Load from environment or secure storage

const ca = process.env.CA_CERT || fs.readFileSync(process.env.CA_CERT_PATH);
const key = process.env.CLIENT_KEY || fs.readFileSync(process.env.CLIENT_KEY_PATH);
const cert = process.env.CLIENT_CERT || fs.readFileSync(process.env.CLIENT_CERT_PATH);

request
  .get(url)
  .ca(ca)
  .key(key)
  .cert(cert);
```

### Certificate Validation

```javascript
// Always validate certificates in production
// Only disable validation for development/testing

if (process.env.NODE_ENV === 'development') {
  // Development: accept self-signed
  process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';
} else {
  // Production: strict validation
  request
    .get(url)
    .ca(fs.readFileSync('trusted-ca.pem'));
}
```
