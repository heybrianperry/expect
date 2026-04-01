# Standardize Structured Logging with Dedicated Logger Utilities: Logger Utilities Imported

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging implementations across the codebase. All components that emit log messages MUST follow the structured logging patterns defined herein.

## Context

- The codebase has evolved to include multiple components (CLI, agent, packages) that require consistent logging for debugging and operational visibility
- Evidence from 4 files shows a pattern of dedicated logger utilities being used across apps/cli and packages/agent, indicating an architectural decision to centralize logging behavior
- Structured logging enables better observability in production environments, facilitating log aggregation, filtering, and analysis across distributed components
- The pattern appears in critical paths including CLI commands (init), test runners, and agent communication (acp-client), suggesting logging is a cross-cutting concern requiring standardization

## Problem Statement

Without a standardized logging approach, different components may implement inconsistent logging mechanisms, leading to fragmented observability, difficulty in debugging across module boundaries, and challenges in implementing system-wide logging policies such as log levels, formatting, and output destinations.

## Decision

1. MUST: Logger utilities MUST be imported from centralized locations (e.g., utils/logger.ts) to ensure consistent behavior across the application

## Policy Block

- MUST Logger utilities MUST be imported from centralized locations (e.g., utils/logger.ts) to ensure consistent behavior across the application

In scope:
- All application code in apps/cli
- All agent code in packages/agent
- Utility modules that perform I/O or external communication
- Command implementations and test runners
- Error handling and exception logging

Out of scope:
- Third-party library internal logging
- Development-only debug statements that are removed before commit
- Test assertion output from testing frameworks
- Build system and tooling output

Exceptions:
- EXC-001: Direct console output is required for user-facing CLI output that is not diagnostic logging (e.g., command results, formatted reports)
- EXC-002: Emergency debugging in production requires temporary console.log statements

## Rationale

- The detection of this pattern across 4 files with 90.50% confidence indicates a deliberate architectural choice to centralize logging behavior, suggesting this is an established practice worth codifying
- Structured logging through dedicated utilities enables system-wide changes to logging behavior (format, destination, filtering) without modifying individual call sites throughout the codebase
- Consistent logging patterns improve developer productivity by providing predictable debugging information and reducing cognitive load when working across different modules
- Centralized logger utilities provide a natural extension point for future observability enhancements such as distributed tracing, metrics collection, or integration with external monitoring systems

## Consequences

Positive:
- Consistent log format and structure across all components improves debuggability and operational visibility
- Centralized logger utilities enable system-wide logging policy changes without widespread code modifications
- Structured logging facilitates integration with log aggregation and analysis tools in production environments
- Clear separation between application logging and user-facing output improves code clarity and maintainability

Negative:
- Additional abstraction layer adds minor complexity compared to direct console calls
- Developers must learn and remember to import logger utilities rather than using built-in console methods
- Potential performance overhead from structured logging in high-frequency code paths (though typically negligible)
- Requires ongoing enforcement through code review to prevent regression to direct console usage

## Alternatives

- Use console.log/console.error directly throughout the codebase without abstraction (rejected)
  Rejected because: Direct console usage provides no centralized control over log formatting, filtering, or output destinations, making it difficult to implement consistent logging policies or integrate with external monitoring systems
  When valid: Only appropriate for simple scripts or prototypes with no production deployment requirements
- Adopt a third-party logging framework (Winston, Pino, Bunyan) as the standard (deferred)
  Rejected because: While third-party frameworks offer rich features, the current custom logger utilities may provide sufficient functionality for current needs; this decision can be revisited if requirements grow
  When valid: Should be reconsidered if requirements emerge for advanced features like log rotation, multiple transports, or complex filtering rules
- Implement separate logger utilities per package with no shared interface (rejected)
  Rejected because: Fragmented logging implementations would prevent consistent observability across the system and create maintenance burden when implementing cross-cutting logging changes
  When valid: Only if packages are truly independent with no shared deployment or operational context

## Risks

- Developers may bypass logger utilities and use console methods directly, especially when under time pressure or unfamiliar with the pattern
  Mitigation: Implement linting rules to detect direct console usage in application code; provide clear documentation and onboarding materials; enforce through code review
  Owner: Engineering team / DevOps
- Logger utility implementation bugs could affect logging across the entire system, potentially hiding critical errors
  Mitigation: Maintain comprehensive tests for logger utilities; implement fallback mechanisms that revert to console output if logger fails; monitor for logging failures in production
  Owner: Platform team
- Performance degradation in high-throughput scenarios if logging is not optimized or if excessive logging is added
  Mitigation: Implement log level filtering to disable verbose logging in production; profile logging overhead in performance-critical paths; provide guidance on appropriate logging frequency
  Owner: Engineering team

## Implementation Notes

- Create or maintain centralized logger utility modules (e.g., utils/logger.ts) that export standard logging functions (debug, info, warn, error)
- Ensure logger utilities support configuration through environment variables (e.g., LOG_LEVEL) to control verbosity in different deployment contexts
- Document the logger API and usage patterns in developer documentation, including examples of appropriate log levels for different scenarios
- Consider implementing structured logging with JSON output for production environments to facilitate parsing by log aggregation tools
- Provide migration guidance for converting existing console.log statements to use the standardized logger utilities

## Continuation Context


Verify commands:
- grep -r "console\.log\|console\.error\|console\.warn" apps/cli/src packages/agent/src --include="*.ts" --exclude="*.test.ts" | grep -v "// user output" || echo "No direct console usage found"
- grep -r "import.*logger" apps/cli/src packages/agent/src --include="*.ts" | wc -l
- test -f apps/cli/src/utils/logger.ts && echo "Logger utility exists" || echo "Logger utility missing"

Accept when:
- Direct console.log/console.error calls in application code (excluding test files) are either absent or explicitly marked as user output with comments
- Logger utility modules exist in expected locations (e.g., apps/cli/src/utils/logger.ts) and are imported by multiple components
- At least 80% of logging statements in the codebase use the standardized logger utilities rather than direct console methods

## Enforcement

- Verified by: Code review process checks for direct console usage in application code
- Verified by: ESLint rules configured to warn or error on console.log/console.error usage in source files
- Verified by: Automated grep-based checks in CI pipeline to detect non-compliant logging patterns
- Violation handling: CI warnings for direct console usage require justification or correction before merge
- Violation handling: Code review feedback requests conversion to logger utilities for new code
- Violation handling: Existing violations may be grandfathered but should be addressed during refactoring of affected modules
- Exception process: Developer identifies legitimate need for exception (e.g., user-facing output) and adds explanatory comment
- Exception process: Code reviewer approves exception during pull request review
- Exception process: Exception is documented inline with comment explaining why direct console usage is appropriate
- Exception process: Exceptions are tracked and reviewed periodically to ensure they remain valid