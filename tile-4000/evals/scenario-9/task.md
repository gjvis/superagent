# Secure mTLS Client

Create an HTTPS client that supports mutual TLS authentication with client certificates.

## Requirements

Implement a function `secureFetch(url, certConfig)` that:

1. Makes a GET request to an HTTPS endpoint
2. Configures custom Certificate Authority for server verification
3. Provides client certificate for mutual TLS authentication
4. Provides the corresponding private key
5. Handles PFX/PKCS12 format certificates with optional passphrase

Example usage:
```javascript
// PEM format (separate cert and key)
await secureFetch('https://secure-api.example.com/data', {
  ca: fs.readFileSync('./ca-cert.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem')
})

// PFX format (bundled cert and key)
await secureFetch('https://secure-api.example.com/data', {
  pfx: fs.readFileSync('./client-cert.pfx'),
  passphrase: 'secret123'
})
```

Return the response data on success.

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: Request with custom CA certificate
Expected: Validates server certificate against custom CA

@test Input: Request with client cert and key for mTLS
Expected: Provides client certificate for authentication

@test Input: Request with PFX bundle and passphrase
Expected: Successfully authenticates using encrypted PFX certificate
