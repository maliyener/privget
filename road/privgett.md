# PRIVGET

> Self-hosted privacy-oriented download broker written in Go.

PRIVGET is a self-hosted download broker designed for users who want to fetch files through infrastructure they control.

The client does not directly connect to the file source. Instead:

```text
CLIENT → PRIVGET BROKER → SOURCE
```

The broker fetches the remote resource and securely streams it back to the client without permanently storing the downloaded payload.

## Project Goals

* [ ] Self-hosted architecture
* [ ] No central PRIVGET service
* [ ] No telemetry
* [ ] No analytics
* [ ] No account requirement
* [ ] No permanent download history by default
* [ ] No permanent payload storage on the broker
* [ ] Secure client-to-broker communication
* [ ] Source-facing connection handled by the broker
* [ ] Streaming downloads
* [ ] Download integrity verification
* [ ] SSRF-resistant source fetching
* [ ] Resource and abuse protection
* [ ] Clean CLI
* [ ] Production-oriented configuration

---

# Phase 0 — Project Foundation

* [ ] Define project structure
* [ ] Define CLI entrypoint
* [ ] Define broker entrypoint
* [ ] Define internal package structure
* [ ] Add configuration model
* [ ] Add environment/config-file support
* [ ] Define application errors
* [ ] Define logging policy
* [ ] Define supported Go version
* [ ] Run `go mod tidy`
* [ ] Verify `go test ./...`

---

# Phase 1 — Basic Broker

## HTTP Server

* [ ] Create HTTP server
* [ ] Add `/health`
* [ ] Add `/download`
* [ ] Add graceful shutdown
* [ ] Add server startup logging
* [ ] Add configurable listen address
* [ ] Add server read timeout
* [ ] Add server write timeout
* [ ] Add server idle timeout

## Basic Request Flow

* [ ] Accept download request
* [ ] Parse requested URL
* [ ] Validate URL syntax
* [ ] Allow only HTTP/HTTPS
* [ ] Fetch remote resource
* [ ] Stream response to client
* [ ] Return appropriate HTTP status codes
* [ ] Handle upstream errors
* [ ] Handle client disconnects

---

# Phase 2 — CLI Client

## CLI

* [ ] Create `privget download <URL>`
* [ ] Add broker address configuration
* [ ] Add authentication configuration
* [ ] Display download progress
* [ ] Display download speed
* [ ] Display downloaded size
* [ ] Save file to disk
* [ ] Determine filename safely
* [ ] Prevent unsafe output paths
* [ ] Handle interrupted downloads cleanly

Example:

```text
privget download https://example.com/file.zip
```

---

# Phase 3 — Secure Client ↔ Broker Communication

## TLS

* [ ] Enable HTTPS
* [ ] Configure server certificate
* [ ] Configure private key
* [ ] Disable insecure TLS versions
* [ ] Configure appropriate TLS settings
* [ ] Validate broker certificate from client
* [ ] Handle TLS connection failures

## Authentication

* [ ] Add broker authentication
* [ ] Do not transmit reusable credentials unnecessarily
* [ ] Support an authentication token
* [ ] Store token outside source code
* [ ] Validate token server-side
* [ ] Reject unauthenticated download requests
* [ ] Return generic authentication errors

## Token Handling

* [ ] Support token rotation
* [ ] Avoid logging authentication tokens
* [ ] Avoid exposing tokens in error messages
* [ ] Keep authentication configuration separate from application configuration

---

# Phase 4 — Secure Source Fetching

This phase protects the broker from being abused as a generic network proxy.

## URL Validation

* [ ] Parse URLs using Go's URL parser
* [ ] Allow only HTTP/HTTPS
* [ ] Reject missing or invalid hosts
* [ ] Reject unsupported ports
* [ ] Normalize host representation
* [ ] Validate redirects
* [ ] Apply the same security checks to redirected URLs

## SSRF Protection

* [ ] Resolve destination hostname
* [ ] Inspect resolved IP addresses
* [ ] Block loopback addresses
* [ ] Block private IPv4 ranges
* [ ] Block link-local addresses
* [ ] Block multicast addresses
* [ ] Block unspecified addresses
* [ ] Block private IPv6 ranges
* [ ] Revalidate redirects
* [ ] Prevent DNS-rebinding bypasses
* [ ] Prevent localhost access
* [ ] Prevent access to internal network services

## Source Request Policy

* [ ] Use a dedicated HTTP client
* [ ] Configure connection timeout
* [ ] Configure response-header timeout
* [ ] Configure idle connection behavior
* [ ] Limit redirects
* [ ] Limit response size
* [ ] Reject unsupported schemes

---

# Phase 5 — Streaming & Resource Protection

The broker must not download the entire file into memory or permanently store it.

## Streaming

* [ ] Stream source response directly to client
* [ ] Avoid loading complete files into RAM
* [ ] Avoid permanent payload storage
* [ ] Use bounded buffers
* [ ] Handle backpressure correctly
* [ ] Handle slow clients
* [ ] Handle upstream disconnects
* [ ] Handle client disconnects

## Resource Limits

* [ ] Maximum download size
* [ ] Maximum concurrent downloads
* [ ] Request timeout
* [ ] Upstream timeout
* [ ] Per-request memory limits where appropriate
* [ ] Rate limiting
* [ ] Reject excessive requests
* [ ] Clean up resources after failed transfers

---

# Phase 6 — Download Integrity

## Hash Verification

* [ ] Calculate SHA-256 while streaming
* [ ] Expose final hash to client
* [ ] Support optional expected SHA-256
* [ ] Compare expected and calculated hash
* [ ] Fail download on mismatch
* [ ] Never claim integrity when verification failed

Example:

```text
Expected SHA-256:
abc123...

Downloaded SHA-256:
abc123...

Integrity: VERIFIED
```

---

# Phase 7 — Resume Support

Only implement this if the core streaming path is stable.

## HTTP Range

* [ ] Detect resumable source
* [ ] Support HTTP Range requests
* [ ] Forward valid Range requests
* [ ] Handle `206 Partial Content`
* [ ] Handle `Content-Range`
* [ ] Handle servers that do not support Range
* [ ] Prevent invalid range requests
* [ ] Preserve integrity verification

## Client Resume

* [ ] Detect partial local file
* [ ] Resume when safe
* [ ] Restart when resume is not possible
* [ ] Do not silently corrupt partial files
* [ ] Verify final file integrity

---

# Phase 8 — Privacy-Oriented Logging

The broker should provide operational visibility without becoming a download-history database.

## Logging Rules

* [ ] Never log authentication tokens
* [ ] Never log downloaded file contents
* [ ] Avoid unnecessary query-string logging
* [ ] Avoid persistent download history
* [ ] Avoid storing complete URLs unnecessarily
* [ ] Minimize client IP retention
* [ ] Provide configurable log level
* [ ] Clearly separate security events from operational logs

## Security Events

* [ ] Authentication failure
* [ ] SSRF rejection
* [ ] Rate-limit rejection
* [ ] Invalid request
* [ ] Integrity failure
* [ ] Resource-limit rejection
* [ ] Unexpected upstream failure

---

# Phase 9 — Configuration & Deployment

## Configuration

* [ ] Broker listen address
* [ ] TLS certificate path
* [ ] TLS key path
* [ ] Authentication configuration
* [ ] Maximum download size
* [ ] Maximum concurrent downloads
* [ ] Rate limits
* [ ] Timeout values
* [ ] Logging configuration

## Deployment

* [ ] Provide example configuration
* [ ] Provide systemd service
* [ ] Run under a dedicated user
* [ ] Restrict filesystem permissions
* [ ] Document required firewall ports
* [ ] Document TLS setup
* [ ] Document secure deployment
* [ ] Document update procedure

---

# Phase 10 — CLI Quality

## User Experience

* [ ] Clean command output
* [ ] Download progress
* [ ] Speed display
* [ ] ETA
* [ ] Clear error messages
* [ ] Authentication errors
* [ ] Source errors
* [ ] Integrity errors
* [ ] Resume status

Commands:

```text
privget download <URL>
privget config
privget status
privget version
```

---

# Phase 11 — Security Review

Before calling the product complete:

* [ ] Review all external input
* [ ] Review URL parsing
* [ ] Review redirect handling
* [ ] Review SSRF protection
* [ ] Review authentication
* [ ] Review TLS configuration
* [ ] Review timeout behavior
* [ ] Review resource limits
* [ ] Review temporary files
* [ ] Review filesystem permissions
* [ ] Review logging
* [ ] Review error messages
* [ ] Review concurrent request handling
* [ ] Review client disconnect handling
* [ ] Review upstream disconnect handling
* [ ] Review sensitive data handling
* [ ] Run `go vet ./...`
* [ ] Run `go test ./...`
* [ ] Run `govulncheck ./...`

---

# Phase 12 — Documentation

* [ ] Write README
* [ ] Explain architecture
* [ ] Explain privacy model
* [ ] Explain threat model
* [ ] Explain what PRIVGET does not protect against
* [ ] Installation instructions
* [ ] Configuration instructions
* [ ] CLI usage
* [ ] Self-hosting instructions
* [ ] TLS setup
* [ ] Security considerations
* [ ] Limitations
* [ ] Example download flow

Architecture:

```text
                  TLS
CLIENT ───────────────────► BROKER
                              │
                              │ HTTPS
                              ▼
                           SOURCE

CLIENT ◄──── streamed file ─── BROKER
```

---

# Definition of Done

PRIVGET v1 is complete when:

* [ ] A user can deploy the broker on their own VPS
* [ ] A client can authenticate to the broker
* [ ] A client can request a remote HTTP/HTTPS file
* [ ] The broker fetches the file
* [ ] The source connection originates from the broker
* [ ] The broker streams the file to the client
* [ ] The broker does not permanently store the payload
* [ ] SSRF protections are active
* [ ] Resource limits are active
* [ ] TLS is properly configured
* [ ] SHA-256 integrity verification works
* [ ] Errors and disconnects are handled safely
* [ ] Logs do not unnecessarily expose sensitive information
* [ ] The application can be deployed as a system service
* [ ] Security review is complete
* [ ] Documentation is complete

---

# Explicitly Out of Scope for v1

These are intentionally NOT part of the first release:

* [ ] Multi-hop relay networks
* [ ] Multiple broker federation
* [ ] Custom encryption algorithms
* [ ] Custom cryptographic protocols
* [ ] Complex distributed routing
* [ ] Anonymous overlay networking
* [ ] P2P networking
* [ ] Tor integration
* [ ] Browser extension
* [ ] Web dashboard
* [ ] Database-backed download history
* [ ] Cloud-managed control plane
* [ ] Multi-tenant SaaS architecture
* [ ] Parallel multi-node downloads
* [ ] Complex distributed consensus
* [ ] Merkle-tree transfer protocol

The goal is not to build a new Internet.

The goal is to build a small, self-hosted, security-focused download broker that solves one problem properly.
