# Standardize Structured Logging with Public API Contracts

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging implementations across the codebase. All new logging code and modifications to existing logging code must comply with these rules.

## Context

- The codebase contains multiple logging implementations across CLI utilities, agent packages, and test runners that require consistent public API contracts
- Pattern detected in 3 files (apps/cli/src/utils/logger.ts, packages/agent/src/acp-client.ts, apps/cli/src/utils/run-test.ts) with 90.27% significance indicates a deliberate architectural approach
- Public API contracts for logging ensure consistent observability interfaces across different components and enable standardized log aggregation and analysis
- The facet 'api.public.contracts' suggests these logging implementations expose well-defined interfaces for external consumption or cross-module integration
- Operational concerns require reliable, structured logging to support debugging, monitoring, and incident response across distributed components

## Problem Statement

Without standardized logging contracts, different components implement inconsistent logging interfaces leading to fragmented observability, difficulty in log aggregation, and increased maintenance burden when integrating logging tools or changing log formats across the system.

## Decision

1. MUST_NOT: Logger implementations MUST NOT directly couple to specific logging backends (e.g., console, file, external services) in their public API contracts

## Policy Block

- MUST All logging implementations MUST expose a public API contract with standardized method signatures (e.g., debug, info, warn, error) that can be consumed by other modules
- MUST Logger instances MUST support structured logging with context objects rather than string concatenation to enable machine-readable log analysis
- MUST Public logging contracts MUST define log levels consistently across all implementations (DEBUG, INFO, WARN, ERROR, FATAL)
- SHOULD Logger implementations SHOULD provide factory functions or dependency injection patterns to enable testability and runtime configuration
- SHOULD Logging APIs SHOULD support contextual metadata (request IDs, user IDs, trace IDs) to enable correlation across distributed operations
- MUST_NOT Logger implementations MUST NOT directly couple to specific logging backends (e.g., console, file, external services) in their public API contracts
- MAY Logger contracts MAY provide specialized methods for specific use cases (e.g., audit logging, performance metrics) as long as they maintain interface consistency

In scope:
- All CLI utility logging (apps/cli/src/utils/logger.ts)
- Agent package logging interfaces (packages/agent/src/acp-client.ts)
- Test runner logging (apps/cli/src/utils/run-test.ts)
- Any new logging implementations in application or library code
- Public APIs exposed for logging consumption by external modules

Out of scope:
- Third-party library internal logging mechanisms
- Temporary debug console.log statements during local development (must be removed before commit)
- System-level logging (OS, container runtime) outside application control
- Logging backend implementations (transport layers, formatters) that don't affect public contracts

Exceptions:
- EXC-001: Legacy code in maintenance mode where refactoring risk exceeds observability benefit
- EXC-002: Performance-critical hot paths where structured logging overhead is measured and documented as unacceptable

## Rationale

- Pattern detected across 3 distinct files (CLI utils, agent packages, test runners) with 90.27% significance indicates this is an established architectural pattern rather than coincidental similarity
- Standardized public API contracts enable consistent observability practices across the codebase, reducing cognitive load for developers and enabling centralized log management
- The 'api.public.contracts' facet indicates these logging implementations are designed for cross-module consumption, requiring stable and well-defined interfaces
- Structured logging with consistent contracts supports modern observability tools (log aggregation, distributed tracing, alerting) that depend on predictable log formats and metadata

## Consequences

Positive:
- Consistent logging interfaces across all components reduce learning curve and enable code reuse
- Structured logging with standardized contracts enables powerful log analysis, filtering, and correlation in production environments
- Decoupling public API contracts from backend implementations allows changing log transports without affecting consuming code
- Testability improves through dependency injection and mockable logger interfaces

Negative:
- Initial implementation effort required to standardize existing logging code that doesn't conform to the contract
- Additional abstraction layer may add slight complexity compared to direct console.log usage
- Developers must learn and follow the logging contract rather than using ad-hoc logging approaches
- Structured logging may have minor performance overhead compared to simple string logging in high-throughput scenarios

## Alternatives

- Use console.log/console.error directly throughout the codebase without standardized contracts (rejected)
  Rejected because: Direct console usage provides no structure, makes testing difficult, prevents log level filtering, and creates tight coupling to console output that cannot be redirected or formatted
  When valid: Only acceptable for temporary local debugging that will not be committed
- Adopt a third-party logging library (Winston, Pino, Bunyan) directly without abstraction layer (rejected)
  Rejected because: Direct dependency on specific logging library creates vendor lock-in and makes it difficult to change logging backends or support multiple environments with different requirements
  When valid: Could be reconsidered if a single logging library becomes an industry standard with guaranteed long-term support
- Implement separate logging solutions per component without shared contracts (rejected)
  Rejected because: Fragmented logging approaches create inconsistent observability, duplicate implementation effort, and make centralized log management impossible
  When valid: Not valid for this codebase given the detected pattern of shared logging contracts

## Risks

- Existing code may not comply with the standardized logging contract, requiring significant refactoring effort
  Mitigation: Implement gradual migration strategy with automated detection of non-compliant logging patterns. Create adapter wrappers for legacy code during transition period.
  Owner: Engineering team with observability working group oversight
- Performance overhead from structured logging may impact high-throughput operations
  Mitigation: Benchmark logging performance in critical paths. Provide async logging options and log level filtering to minimize overhead. Document exception process for performance-critical code.
  Owner: Performance engineering team
- Developers may bypass the logging contract for convenience, leading to inconsistent adoption
  Mitigation: Implement automated linting rules to detect non-compliant logging patterns. Provide clear documentation and examples. Include logging contract compliance in code review checklist.
  Owner: Engineering team and code review process

## Implementation Notes

- Create a shared logging interface/type definition that all logger implementations must satisfy, exporting it from a common package
- Provide factory functions or builder patterns for creating logger instances with appropriate context (component name, module path, etc.)
- Document the standard log levels and when to use each (DEBUG for verbose diagnostics, INFO for significant events, WARN for recoverable issues, ERROR for failures)
- Include examples in documentation showing how to use structured logging with context objects: logger.info('User action', { userId, action, timestamp })
- Consider providing utility functions for common logging patterns (request/response logging, error serialization, performance timing)

## Continuation Context


Verify commands:
- grep -r 'console\.log\|console\.error' apps/ packages/ --exclude-dir=node_modules --exclude-dir=dist | grep -v '// TODO: replace with logger' || echo 'No direct console usage found'
- find apps/ packages/ -name '*logger*.ts' -o -name '*logging*.ts' | xargs grep -l 'export.*interface.*Logger\|export.*type.*Logger' | wc -l
- npm run lint -- --rule 'no-console: error' 2>&1 | grep -q 'no-console' && echo 'Linting enforces no direct console usage' || echo 'Warning: no-console rule not configured'

Accept when:
- All logger implementations export a public interface or type definition with standardized method signatures (debug, info, warn, error)
- Grep search for direct console.log/console.error usage returns zero results in production code (excluding explicitly marked temporary debug statements)
- At least 90% of logging calls use structured format with context objects rather than string concatenation
- Linting rules successfully detect and flag non-compliant logging patterns in CI pipeline

## Enforcement

- Verified by: Automated linting rules in CI pipeline checking for direct console usage and enforcing structured logging patterns
- Verified by: Code review checklist includes verification of logging contract compliance
- Verified by: Static analysis tools scan for logger interface conformance
- Verified by: Periodic audits of logging implementations to ensure continued compliance with public API contracts
- Violation handling: CI pipeline fails on detection of direct console.log/console.error usage in production code
- Violation handling: Code review blocks merge requests that introduce non-compliant logging patterns
- Violation handling: Automated comments on pull requests flag potential logging contract violations with links to documentation
- Violation handling: Technical debt tickets created for legacy code that doesn't comply, prioritized based on component criticality
- Exception process: Developer submits exception request with justification (performance data, legacy code risk assessment, etc.)
- Exception process: Engineering lead or architecture review board evaluates request against documented exception criteria
- Exception process: Approved exceptions must be documented in code with comments explaining rationale and linking to approval
- Exception process: Exception registry maintained with periodic review to identify opportunities for remediation