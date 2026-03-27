<rule_activation id="76feadcc-bd1e-4ddd-a3a1-d6a40b3496fb" title="Standardize External Client Libraries for Third-Party Service Integration" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files in the codebase that integrate with external third-party services.
</rule_activation>

### Rules

- **R-EXT-CLIENT-001** MUST: External service integrations MUST be encapsulated in dedicated client library modules with clear boundaries.
- **R-EXT-CLIENT-002** MUST: Client libraries MUST provide typed interfaces that abstract the underlying service API details from consuming code.
- **R-EXT-CLIENT-003** MUST: Connection management and authentication logic MUST be centralized within the client library implementation.
- **R-EXT-CLIENT-004** SHOULD: Client libraries SHOULD implement consistent error handling patterns including retry logic and timeout management.
- **R-EXT-CLIENT-005** SHOULD: External client modules SHOULD be placed in dedicated directories or packages that clearly identify them as external integration points.
- **R-EXT-CLIENT-006** SHOULD: Client libraries SHOULD provide configuration options for connection parameters, timeouts, and service endpoints.
- **R-EXT-CLIENT-007** MAY: Client libraries MAY implement caching or connection pooling when appropriate for the service characteristics.

### Scope

**In scope:**
- All integrations with external HTTP/REST APIs
- WebDriver and browser automation service clients
- Remote state management and synchronization services
- Third-party SaaS platform integrations
- External database or storage service clients

**Out of scope:**
- Internal service-to-service communication within the same deployment
- Standard library HTTP utilities for one-off requests
- Native browser APIs or DOM manipulation
- File system or local resource access

**Exceptions:**
- EXC-001: Proof-of-concept or experimental integrations with services not yet approved for production use
- EXC-002: Simple webhook receivers or one-time data import scripts with no ongoing maintenance requirements

### Verify

```bash
# Check for client library implementations with proper naming
grep -r "class.*Client" --include="*-client.ts" packages/ apps/

# Count external client modules
find . -type f -name "*-client.ts" -o -name "*-client.js" | wc -l

# Detect direct HTTP calls outside of client libraries
grep -r "import.*axios\|fetch" --include="*.ts" packages/ apps/ | grep -v "client.ts" | wc -l
```

**Accept when:**
- All external service integrations are encapsulated in dedicated client modules with clear naming conventions
- Client libraries provide typed interfaces and do not expose raw HTTP client details to consumers
- Direct HTTP calls to external services outside of client libraries are limited to exceptional cases with documented justification
- New external service integrations follow the established client library pattern without requiring architectural review

<enforcement>
Claude Code MUST NOT skip or defer verification. All external service integrations MUST be reviewed to ensure compliance with client library patterns. Code review MUST block merge if external service calls bypass client library pattern without documented exception.
</enforcement>