# HTTPS and TLS

Configure SSL/TLS certificates for secure HTTPS requests in Node.js environments.

## Capabilities

### Certificate Authority

Set custom CA certificates for HTTPS requests.

```javascript { .api }
/**
 * Set certificate authority
 * @param {Buffer|string|Array} cert - CA certificate(s)
 * @returns {Request} Request instance for chaining
 */
Request.prototype.ca = function(cert);

/**
 * Set default CA certificate on agent
 * @param {Buffer|string|Array} cert - CA certificate(s)
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.ca = function(cert);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Single CA certificate
const caCert = fs.readFileSync('ca-cert.pem');

request
  .get('https://api.example.com/data')
  .ca(caCert)
  .end(callback);

// Multiple CA certificates
const caCerts = [
  fs.readFileSync('ca-cert1.pem'),
  fs.readFileSync('ca-cert2.pem'),
  fs.readFileSync('ca-cert3.pem')
];

request
  .get('https://api.example.com/data')
  .ca(caCerts)
  .end(callback);

// CA certificate as string
const caCertString = `-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAKL0UG+mRKmzMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
...
-----END CERTIFICATE-----`;

request
  .get('https://api.example.com/data')
  .ca(caCertString)
  .end(callback);

// Set CA on agent (applies to all requests)
const agent = request.agent();
agent.ca(caCert);

agent.get('https://api.example.com/users').end(callback);
agent.post('https://api.example.com/data').send({}).end(callback);
```

### Client Certificate

Set client certificate for mutual TLS authentication.

```javascript { .api }
/**
 * Set client certificate
 * @param {Buffer|string} cert - Client certificate
 * @returns {Request} Request instance for chaining
 */
Request.prototype.cert = function(cert);

/**
 * Set default client certificate on agent
 * @param {Buffer|string} cert - Client certificate
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.cert = function(cert);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Client certificate as Buffer
const clientCert = fs.readFileSync('client-cert.pem');

request
  .get('https://api.example.com/protected')
  .cert(clientCert)
  .end(callback);

// Client certificate as string
const clientCertString = fs.readFileSync('client-cert.pem', 'utf8');

request
  .get('https://api.example.com/protected')
  .cert(clientCertString)
  .end(callback);

// Set on agent
const agent = request.agent();
agent.cert(clientCert);

agent.get('https://api.example.com/protected').end(callback);
```

### Private Key

Set private key for client certificate.

```javascript { .api }
/**
 * Set client private key
 * @param {Buffer|string} key - Private key
 * @returns {Request} Request instance for chaining
 */
Request.prototype.key = function(key);

/**
 * Set default client private key on agent
 * @param {Buffer|string} key - Private key
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.key = function(key);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Private key as Buffer
const privateKey = fs.readFileSync('client-key.pem');

request
  .get('https://api.example.com/protected')
  .key(privateKey)
  .end(callback);

// Private key as string
const privateKeyString = fs.readFileSync('client-key.pem', 'utf8');

request
  .get('https://api.example.com/protected')
  .key(privateKeyString)
  .end(callback);

// Set on agent
const agent = request.agent();
agent.key(privateKey);

agent.get('https://api.example.com/protected').end(callback);
```

### PFX/PKCS12 Certificate

Set PFX or PKCS12 certificate bundle containing both certificate and private key.

```javascript { .api }
/**
 * Set PFX/PKCS12 certificate
 * @param {Buffer|string|object} pfx - PFX certificate or options
 * @param {Buffer|string} [pfx.pfx] - PFX certificate data
 * @param {string} [pfx.passphrase] - Passphrase for encrypted PFX
 * @returns {Request} Request instance for chaining
 */
Request.prototype.pfx = function(pfx);

/**
 * Set default PFX certificate on agent
 * @param {Buffer|string|object} pfx - PFX certificate or options
 * @returns {Agent} Agent instance for chaining
 */
Agent.prototype.pfx = function(pfx);
```

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// PFX certificate as Buffer
const pfxCert = fs.readFileSync('client-cert.pfx');

request
  .get('https://api.example.com/protected')
  .pfx(pfxCert)
  .end(callback);

// PFX with passphrase
const pfxCert = fs.readFileSync('client-cert.pfx');

request
  .get('https://api.example.com/protected')
  .pfx({
    pfx: pfxCert,
    passphrase: 'secret-password'
  })
  .end(callback);

// PFX as file path (Node.js will read it)
request
  .get('https://api.example.com/protected')
  .pfx('client-cert.pfx')
  .end(callback);

// Set on agent
const agent = request.agent();
agent.pfx({
  pfx: pfxCert,
  passphrase: 'secret-password'
});

agent.get('https://api.example.com/protected').end(callback);
```

### Mutual TLS (mTLS)

Configure mutual TLS authentication with client certificates.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Complete mTLS configuration
const caCert = fs.readFileSync('ca-cert.pem');
const clientCert = fs.readFileSync('client-cert.pem');
const clientKey = fs.readFileSync('client-key.pem');

request
  .get('https://api.example.com/protected')
  .ca(caCert)
  .cert(clientCert)
  .key(clientKey)
  .end((err, res) => {
    if (err) {
      console.error('mTLS authentication failed:', err);
    } else {
      console.log('Authenticated:', res.body);
    }
  });

// mTLS with PFX
const caCert = fs.readFileSync('ca-cert.pem');
const pfxCert = fs.readFileSync('client-cert.pfx');

request
  .get('https://api.example.com/protected')
  .ca(caCert)
  .pfx({
    pfx: pfxCert,
    passphrase: 'secret'
  })
  .end(callback);

// mTLS agent configuration
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem')
});

// All requests through agent use mTLS
agent.get('https://api.example.com/users').end(callback);
agent.post('https://api.example.com/data').send({}).end(callback);
agent.put('https://api.example.com/resource/123').send({}).end(callback);

// Factory function for mTLS agent
function createMTLSAgent(certPath, keyPath, caPath) {
  return request.agent({
    ca: fs.readFileSync(caPath),
    cert: fs.readFileSync(certPath),
    key: fs.readFileSync(keyPath)
  });
}

const mtlsAgent = createMTLSAgent(
  'client-cert.pem',
  'client-key.pem',
  'ca-cert.pem'
);

await mtlsAgent.get('https://api.example.com/data');
```

### Self-Signed Certificates

Handle self-signed certificates in development environments.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Accept self-signed certificate with CA
const caCert = fs.readFileSync('self-signed-ca.pem');

request
  .get('https://localhost:3000/api/data')
  .ca(caCert)
  .end(callback);

// DANGER: Disable certificate validation (development only)
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

request
  .get('https://localhost:3000/api/data')
  .end(callback);

// Better approach: Use custom agent with rejectUnauthorized
const https = require('https');

const agent = new https.Agent({
  rejectUnauthorized: false
});

request
  .get('https://localhost:3000/api/data')
  .agent(agent)
  .end(callback);

// Production-safe: Only disable for specific hosts
function createDevAgent(host) {
  const https = require('https');

  return new https.Agent({
    rejectUnauthorized: process.env.NODE_ENV !== 'production',
    servername: host
  });
}

request
  .get('https://localhost:3000/api/data')
  .agent(createDevAgent('localhost'))
  .end(callback);
```

### Agent Configuration

Configure TLS settings when creating an agent.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');

// Create agent with TLS configuration
const agent = request.agent({
  ca: fs.readFileSync('ca-cert.pem'),
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem')
});

// Use agent for all requests
await agent.get('https://api.example.com/users');
await agent.post('https://api.example.com/data').send({});

// Agent with PFX
const agent = request.agent({
  pfx: fs.readFileSync('client-cert.pfx'),
  passphrase: 'secret-password'
});

// Agent with multiple CA certificates
const agent = request.agent({
  ca: [
    fs.readFileSync('ca-cert1.pem'),
    fs.readFileSync('ca-cert2.pem')
  ]
});

// Configure TLS settings after creation
const agent = request.agent();

agent
  .ca(fs.readFileSync('ca-cert.pem'))
  .cert(fs.readFileSync('client-cert.pem'))
  .key(fs.readFileSync('client-key.pem'));

// All requests inherit TLS configuration
await agent.get('https://api.example.com/data');
```

### Common TLS Scenarios

Real-world examples of TLS configuration.

**Usage Examples:**

```javascript
const request = require('superagent');
const fs = require('fs');
const path = require('path');

// 1. Corporate Internal API with Custom CA
function createCorporateAPIClient(certDir) {
  const agent = request.agent({
    ca: fs.readFileSync(path.join(certDir, 'corporate-ca.pem'))
  });

  return {
    async getUsers() {
      const res = await agent.get('https://internal-api.company.com/users');
      return res.body;
    },

    async getData() {
      const res = await agent.get('https://internal-api.company.com/data');
      return res.body;
    }
  };
}

const api = createCorporateAPIClient('./certs');
const users = await api.getUsers();

// 2. IoT Device Authentication
function createIoTClient(deviceId, certDir) {
  const agent = request.agent({
    ca: fs.readFileSync(path.join(certDir, 'iot-ca.pem')),
    cert: fs.readFileSync(path.join(certDir, `device-${deviceId}.pem`)),
    key: fs.readFileSync(path.join(certDir, `device-${deviceId}-key.pem`))
  });

  return {
    async sendTelemetry(data) {
      const res = await agent
        .post('https://iot.example.com/telemetry')
        .send(data);
      return res.body;
    },

    async getConfig() {
      const res = await agent.get('https://iot.example.com/config');
      return res.body;
    }
  };
}

const device = createIoTClient('device-001', './device-certs');
await device.sendTelemetry({ temperature: 25, humidity: 60 });

// 3. Microservices with Mutual TLS
class ServiceClient {
  constructor(serviceName, certDir) {
    this.serviceName = serviceName;
    this.agent = request.agent({
      ca: fs.readFileSync(path.join(certDir, 'service-ca.pem')),
      cert: fs.readFileSync(path.join(certDir, `${serviceName}-cert.pem`)),
      key: fs.readFileSync(path.join(certDir, `${serviceName}-key.pem`))
    });
  }

  async call(method, endpoint, data) {
    const req = this.agent[method.toLowerCase()](
      `https://${this.serviceName}.internal${endpoint}`
    );

    if (data) {
      req.send(data);
    }

    const res = await req;
    return res.body;
  }
}

const userService = new ServiceClient('user-service', './service-certs');
const users = await userService.call('get', '/users');

// 4. Multi-Environment Configuration
class APIClient {
  constructor(environment) {
    const certDir = './certs/' + environment;

    const config = {
      production: {
        baseUrl: 'https://api.example.com',
        ca: fs.readFileSync(path.join(certDir, 'ca.pem')),
        cert: fs.readFileSync(path.join(certDir, 'client.pem')),
        key: fs.readFileSync(path.join(certDir, 'client-key.pem'))
      },
      staging: {
        baseUrl: 'https://staging-api.example.com',
        ca: fs.readFileSync(path.join(certDir, 'ca.pem')),
        cert: fs.readFileSync(path.join(certDir, 'client.pem')),
        key: fs.readFileSync(path.join(certDir, 'client-key.pem'))
      },
      development: {
        baseUrl: 'https://localhost:3000',
        // Self-signed cert for development
        ca: fs.readFileSync(path.join(certDir, 'self-signed-ca.pem'))
      }
    };

    const envConfig = config[environment];

    this.agent = request.agent({
      ca: envConfig.ca,
      cert: envConfig.cert,
      key: envConfig.key
    });

    this.baseUrl = envConfig.baseUrl;
  }

  async get(path) {
    const res = await this.agent.get(this.baseUrl + path);
    return res.body;
  }

  async post(path, data) {
    const res = await this.agent.post(this.baseUrl + path).send(data);
    return res.body;
  }
}

const prodAPI = new APIClient('production');
const stagingAPI = new APIClient('staging');
const devAPI = new APIClient('development');

// 5. Certificate Rotation Support
class CertificateManager {
  constructor(certPaths) {
    this.certPaths = certPaths;
    this.agent = null;
    this.loadCertificates();
  }

  loadCertificates() {
    this.agent = request.agent({
      ca: fs.readFileSync(this.certPaths.ca),
      cert: fs.readFileSync(this.certPaths.cert),
      key: fs.readFileSync(this.certPaths.key)
    });

    console.log('Certificates loaded');
  }

  reloadCertificates() {
    console.log('Reloading certificates...');
    this.loadCertificates();
  }

  async request(method, url, data) {
    try {
      const req = this.agent[method.toLowerCase()](url);

      if (data) {
        req.send(data);
      }

      const res = await req;
      return res.body;

    } catch (err) {
      if (err.code === 'EPROTO' || err.message.includes('certificate')) {
        console.log('Certificate error, attempting reload...');
        this.reloadCertificates();

        // Retry request with new certificates
        return this.request(method, url, data);
      }

      throw err;
    }
  }
}

const certManager = new CertificateManager({
  ca: './certs/ca.pem',
  cert: './certs/client.pem',
  key: './certs/client-key.pem'
});

// Reload certificates periodically
setInterval(() => {
  certManager.reloadCertificates();
}, 24 * 60 * 60 * 1000); // Daily

await certManager.request('get', 'https://api.example.com/data');
```

### Important Notes

- **Node.js Only**: TLS certificate configuration is only available in Node.js, not in browsers.
- **File Formats**: Certificates can be provided as Buffer or string in PEM format. PFX/PKCS12 format is also supported.
- **Private Key Security**: Never commit private keys to version control. Use environment variables or secure key management systems.
- **Certificate Chains**: For CA certificates, you can provide an array of certificates to support certificate chains.
- **mTLS**: For mutual TLS, you need both CA certificate (to validate server) and client certificate + key (to authenticate client).
- **Self-Signed Certs**: In production, avoid disabling certificate validation. In development, use CA certificate or custom agent.
- **Agent Reuse**: When using TLS, create an agent and reuse it for multiple requests to avoid loading certificates repeatedly.
- **PFX Passphrase**: If PFX file is encrypted, provide passphrase in options object.
- **Certificate Expiration**: Monitor certificate expiration dates and implement rotation strategies.
