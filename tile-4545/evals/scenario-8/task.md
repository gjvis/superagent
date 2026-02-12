# Secure API Client

Build a secure HTTP client that uses custom TLS certificates for mutual authentication with a server.

## Requirements

Your implementation should:

1. Configure a custom Certificate Authority (CA) certificate for server verification
2. Set client certificate and private key for mutual TLS authentication
3. Make HTTPS requests using the configured certificates
4. Support loading certificates from file paths or buffers
5. Handle certificate-related errors appropriately

## Test Cases

- A request configured with a custom CA certificate uses that certificate for server verification @test
- A request with both client certificate and key performs mutual TLS authentication @test
- Certificate configuration can use file paths or Buffer objects @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

An HTTP client library that provides a fluent API for making HTTP requests in both browser and Node.js environments.
