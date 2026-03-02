# Security Policy

## Supported Versions

| Version | Supported |
| ------- | --------- |
| 1.x.x   | Yes       |
| 0.x.x   | No        |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it responsibly.

**Do NOT open a public GitHub issue for security vulnerabilities.**

Instead, please email the maintainer directly or use GitHub's private vulnerability reporting feature:

1. Go to the repository's **Security** tab
2. Click **Report a vulnerability**
3. Provide a detailed description of the issue

### What to Include

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial assessment**: Within 1 week
- **Fix or mitigation**: Dependent on severity, typically within 2 weeks for critical issues

## Security Considerations

This project runs agent-generated code in a sandboxed environment. Key security measures:

### Sandbox Isolation

- **QuickJS WASM sandbox**: All untrusted code runs in an isolated WASM environment with memory (64 MB) and CPU (30s) limits
- **No host access**: Sandbox code cannot access `process`, `require`, `fs`, `net`, or any Node.js APIs
- **No eval in host**: The host Node.js process never executes `eval()` or `new Function()` with untrusted input
- **Fresh contexts**: Each execution creates a new sandbox context; no state persists between invocations

### Input Validation

- **Method validation**: Sandbox code can only invoke allowed FMG methods (`get`, `set`, `add`, `update`, `delete`, `exec`, `clone`, `move`, `replace`)
- **Parameter validation**: All parameters from sandbox code must be arrays with required `url` fields before being forwarded to FortiManager
- **Code size limit**: Code inputs exceeding 100 KB are rejected before execution
- **Input validation**: All environment variables and API inputs validated with Zod schemas

### Resource Limits

- **API call cap**: Maximum 50 API calls per sandbox execution prevents runaway loops
- **Log accumulation cap**: Console output capped at 1 MB / 1,000 entries to prevent host memory exhaustion
- **Response truncation**: Results exceeding 100 KB are truncated with guidance on narrowing the query
- **HTTP timeout**: 30-second timeout on all FortiManager API calls prevents indefinite hangs

### Network Security

- **TLS verification**: Enabled by default for FortiManager connections
- **Token-based auth**: API tokens via `Authorization: Bearer` header; no passwords stored
- **Response shape validation**: JSON-RPC response bodies are validated before processing, preventing crashes from malformed responses
- **SSL fallback hardened**: Throws an error instead of silently degrading when SSL bypass is not available

### Transport Security

- **Graceful shutdown**: Both stdio and HTTP transports handle SIGINT/SIGTERM for clean shutdown with signal deduplication
- **HTTP error boundary**: All HTTP handler exceptions are caught and return proper 500 responses
- **Startup health check**: FortiManager connectivity validated at boot (non-fatal warning)
