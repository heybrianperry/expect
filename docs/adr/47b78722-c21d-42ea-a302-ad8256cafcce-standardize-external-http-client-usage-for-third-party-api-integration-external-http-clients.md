# Standardize External HTTP Client Usage for Third-Party API Integration: External Http Clients

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires integration with multiple external third-party services and APIs across different modules (CLI utilities, update checking hooks, and WebDriver clients)
- Pattern detected across 3 files with 86.10% confidence indicates a consistent approach to external HTTP client usage for API communication
- The facet 'boundaries.external_clients' suggests architectural boundaries are established between internal code and external service dependencies
- Multiple application layers (CLI tools, browser automation packages) need reliable, consistent mechanisms for making outbound HTTP requests to external services
- The pattern emerges from practical needs: push state management, version update checks, and WebDriver protocol communication all require HTTP client capabilities

## Problem Statement

Without a standardized approach to external HTTP client usage, the codebase risks inconsistent error handling, varying timeout behaviors, incompatible authentication patterns, and duplicated client configuration logic across different modules that interact with third-party APIs.

## Decision

1. MUST: External HTTP clients MUST implement proper timeout configuration to prevent indefinite blocking on external service calls

## Policy Block

- MUST External HTTP clients MUST implement proper timeout configuration to prevent indefinite blocking on external service calls

In scope:
- HTTP/HTTPS requests to external third-party APIs and services
- WebDriver protocol communication with external browser automation services
- Version check requests to package registries or update servers
- State synchronization with external backend services
- Any outbound network communication crossing application boundaries

Out of scope:
- Internal service-to-service communication within the same deployment
- Database client connections
- File system I/O operations
- WebSocket or streaming connections (unless HTTP-based)
- GraphQL clients (covered by separate ADR if applicable)

Exceptions:
- EXC-001: Legacy code in maintenance mode where refactoring risk exceeds benefit
- EXC-002: Third-party SDK provides its own HTTP client that cannot be replaced

## Rationale

- The pattern appears consistently across 3 distinct files spanning different application layers (CLI, hooks, browser packages), indicating an established architectural practice
- The 86.10% confidence score suggests this is a deliberate, well-established pattern rather than accidental code duplication
- Standardizing external HTTP client usage reduces cognitive load for developers and ensures consistent behavior across all external API integrations
- The facet 'boundaries.external_clients' indicates intentional architectural separation between internal logic and external dependencies, improving testability and maintainability

## Consequences

Positive:
- Consistent error handling and timeout behavior across all external API integrations
- Easier testing through mockable HTTP client interfaces and dependency injection
- Simplified maintenance when HTTP client library needs upgrading or replacing
- Clear architectural boundaries between application logic and external service dependencies
- Reduced code duplication and configuration inconsistencies

Negative:
- Additional abstraction layer may add slight complexity for simple HTTP requests
- Team must learn and follow the standardized client patterns rather than using ad-hoc solutions
- Potential performance overhead if abstraction layer is not optimized
- Migration effort required for existing code using non-standard HTTP clients

## Alternatives

- Allow each module to choose its own HTTP client library based on specific needs (rejected)
  Rejected because: Creates inconsistent error handling, timeout behaviors, and increases maintenance burden with multiple client libraries to manage
  When valid: Only valid for isolated prototypes or proof-of-concept code not intended for production
- Use native fetch API or built-in HTTP modules without abstraction (rejected)
  Rejected because: Lacks standardized retry logic, timeout handling, and makes testing more difficult without a mockable abstraction layer
  When valid: Acceptable for simple one-off scripts or tools with no reliability requirements
- Implement a custom HTTP client wrapper around a standard library (accepted)
  When valid: This is the recommended approach, providing standardization while allowing customization for specific needs

## Risks

- Chosen HTTP client library may have security vulnerabilities or become unmaintained
  Mitigation: Regularly audit dependencies, monitor security advisories, and maintain abstraction layer to enable library replacement if needed
  Owner: Engineering team and security team
- Abstraction layer may not support all features needed by future external API integrations
  Mitigation: Design abstraction with extensibility in mind, allow escape hatches for advanced use cases, and review patterns quarterly
  Owner: Architecture team
- Developers may bypass standardized clients for perceived convenience or lack of awareness
  Mitigation: Enforce through code review, linting rules, and clear documentation with examples
  Owner: Engineering team leads

## Implementation Notes

- Create a shared HTTP client module in a common package that can be imported by CLI, browser, and other packages
- Configure default timeouts (e.g., 30 seconds for connection, 60 seconds for response) that can be overridden per-request
- Implement typed request/response interfaces to improve type safety across external API boundaries
- Provide clear examples in documentation showing how to use the standardized client for common scenarios (GET, POST, authentication, error handling)
- Consider using dependency injection to make HTTP clients easily mockable in unit tests

## Continuation Context


Verify commands:
- grep -r "import.*axios\|import.*fetch\|import.*http" apps/ packages/ | grep -v "node_modules" | grep -E "\.(ts|js)$"
- find apps/ packages/ -name "*client*.ts" -o -name "*http*.ts" | xargs grep -l "export.*class.*Client"
- npm list --depth=0 | grep -E "axios|node-fetch|got|superagent"

Accept when:
- All external HTTP API calls use the standardized client library or approved abstraction
- HTTP client usage is isolated in dedicated client modules with clear boundaries
- No direct usage of multiple competing HTTP libraries (axios, node-fetch, got) scattered across the codebase
- Code review checklist includes verification of standardized HTTP client usage for new external API integrations

## Enforcement

- Verified by: Code review process checks for standardized HTTP client usage
- Verified by: ESLint rules restrict direct imports of HTTP libraries outside approved client modules
- Verified by: CI pipeline runs grep-based verification commands to detect non-standard HTTP client usage
- Verified by: Architecture review for new external service integrations
- Violation handling: Pull requests with non-standard HTTP client usage are blocked until refactored
- Violation handling: Existing violations are tracked in technical debt backlog with prioritization
- Violation handling: Violations in critical paths are addressed immediately before release
- Violation handling: Repeated violations trigger team training sessions on standardized patterns
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead reviews with architecture team if needed
- Exception process: Approved exceptions are documented in code with ADR reference and expiration date
- Exception process: Exceptions are reviewed quarterly and must be renewed or resolved