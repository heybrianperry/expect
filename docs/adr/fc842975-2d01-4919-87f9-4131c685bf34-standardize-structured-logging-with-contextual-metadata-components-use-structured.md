# Standardize Structured Logging with Contextual Metadata: Components Use Structured

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging implementations across the codebase. All components that emit log messages must comply with these standards.

## Context

- The codebase contains multiple components (CLI utilities, agent packages, command handlers) that require consistent logging for debugging, monitoring, and operational visibility
- Evidence from 4 files shows a pattern of structured logging implementation with contextual metadata across different layers of the application stack
- Operational teams need standardized log formats to effectively troubleshoot issues, trace requests, and monitor system health across distributed components
- The pattern detected (signature c02a37427e28e82472d6ffabec40e650) indicates a deliberate architectural choice for logging infrastructure with 90.50% confidence across CLI tools, agent clients, and command handlers

## Problem Statement

Without a standardized logging approach, different components may emit logs in inconsistent formats, making it difficult to correlate events across the system, aggregate logs effectively, and maintain operational visibility. Ad-hoc logging practices lead to missing context, inconsistent severity levels, and reduced debuggability in production environments.

## Decision

1. MUST: All components MUST use a structured logging framework that supports contextual metadata and consistent formatting

## Policy Block

- MUST All components MUST use a structured logging framework that supports contextual metadata and consistent formatting

In scope:
- All TypeScript/JavaScript modules in apps/cli/src/**
- All agent packages in packages/agent/src/**
- Command handlers and utility functions that perform I/O operations
- Client libraries that interact with external services
- Test runners and development utilities

Out of scope:
- Third-party library internal logging (unless configurable)
- Build scripts and tooling configuration files
- Static content and documentation files
- Test fixtures and mock data

Exceptions:
- EXC-001: Legacy code in maintenance mode that is scheduled for deprecation within 6 months
- EXC-002: Performance-critical hot paths where structured logging overhead is measurably prohibitive

## Rationale

- Pattern detected across 4 critical files (logger.ts, run-test.ts, acp-client.ts, init.ts) with 90.50% significance indicates this is an established architectural pattern
- Structured logging with contextual metadata enables effective log aggregation, searching, and correlation in modern observability platforms
- Consistent logging standards reduce cognitive load for developers and operators who need to understand system behavior across multiple components
- Standardized severity levels and error context improve incident response time and reduce mean time to resolution (MTTR)

## Consequences

Positive:
- Improved operational visibility through consistent log formats that can be easily parsed and aggregated
- Faster debugging and troubleshooting with rich contextual information in error logs
- Better distributed tracing capabilities through correlation identifiers across component boundaries
- Reduced onboarding time for new developers who can rely on consistent logging patterns

Negative:
- Initial implementation overhead to standardize logging across existing components
- Slight performance overhead from structured logging framework compared to simple console.log statements
- Potential for log volume increase if not properly managed with appropriate severity levels
- Dependency on specific logging library creates coupling and potential migration costs

## Alternatives

- Use simple console.log/console.error statements without structured logging framework (rejected)
  Rejected because: Unstructured logs are difficult to parse, aggregate, and search in production environments. Lacks contextual metadata and consistent formatting needed for operational visibility.
  When valid: Only appropriate for throwaway scripts or local development debugging
- Implement custom logging abstraction layer without external dependencies (rejected)
  Rejected because: Reinventing logging infrastructure diverts engineering resources from core features and results in less mature, less tested logging capabilities compared to established libraries.
  When valid: May be considered if existing logging libraries cannot meet specific regulatory or performance requirements
- Allow each component to choose its own logging approach (rejected)
  Rejected because: Inconsistent logging approaches across components make it impossible to correlate events, aggregate logs effectively, or maintain operational visibility across the system.
  When valid: Never valid in a production system requiring operational support

## Risks

- Excessive logging volume may impact application performance or increase infrastructure costs
  Mitigation: Implement log level configuration per environment (debug in dev, info/warn/error in production). Use sampling for high-frequency events. Monitor log volume metrics.
  Owner: Engineering team with DevOps oversight
- Sensitive information may be inadvertently logged, creating security or compliance issues
  Mitigation: Implement automated scanning for common sensitive patterns (API keys, tokens, PII). Provide redaction utilities and clear guidelines. Include security review in code review process.
  Owner: Security team with engineering implementation
- Logging framework dependency may become unmaintained or require migration
  Mitigation: Choose widely-adopted logging libraries with active communities. Abstract logging interface to enable future migration. Document migration strategy.
  Owner: Architecture team

## Implementation Notes

- Create a centralized logger utility module (e.g., utils/logger.ts) that configures and exports logger instances with consistent settings
- Define standard log metadata fields (timestamp, severity, component, correlationId, userId) that should be included where applicable
- Configure log levels via environment variables to enable different verbosity in development vs production environments
- Integrate logging with existing error handling patterns to ensure exceptions are properly logged with context before propagation
- Document logging best practices in developer guidelines with examples of appropriate log levels and context inclusion

## Continuation Context


Verify commands:
- grep -r "console\.log\|console\.error" apps/cli/src packages/agent/src --include="*.ts" --exclude="*.test.ts" | wc -l
- grep -r "logger\.info\|logger\.error\|logger\.warn\|logger\.debug" apps/cli/src packages/agent/src --include="*.ts" | wc -l
- find apps/cli/src packages/agent/src -name "logger.ts" -o -name "*logger*.ts" | head -5

Accept when:
- Structured logging framework usage (logger.info/error/warn/debug) significantly exceeds direct console usage in production code
- Logger utility modules exist and are imported by components in the detected pattern files (apps/cli/src/utils/logger.ts, etc.)
- Code review checklist includes verification of appropriate log levels and context inclusion for new logging statements

## Enforcement

- Verified by: Automated linting rules that flag direct console.log usage in production code paths
- Verified by: Code review process with specific checklist items for logging standards compliance
- Verified by: Static analysis tools that verify logger imports and usage patterns
- Violation handling: Linter warnings for console.log usage must be addressed before merge approval
- Violation handling: Code review feedback requires addition of appropriate context and severity levels
- Violation handling: Repeated violations trigger team discussion and additional training on logging standards
- Exception process: Developer documents exception rationale in pull request description
- Exception process: Engineering lead or architect reviews and approves exception with justification
- Exception process: Exception is documented in code with TODO comment linking to tracking issue for future remediation