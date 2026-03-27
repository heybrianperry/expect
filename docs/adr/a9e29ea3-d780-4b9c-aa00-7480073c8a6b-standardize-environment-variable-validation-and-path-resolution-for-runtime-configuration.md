# Standardize Environment Variable Validation and Path Resolution for Runtime Configuration

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime configuration and environment management code across the codebase.

## Context

- The codebase operates across multiple runtime environments including CLI tools, browser automation, iOS simulators, and cookie management utilities, each requiring platform-specific configuration paths and environment variables
- Configuration sources include environment variables, file system paths, user home directories, and platform-specific application data locations that must be resolved consistently
- Security concerns around input validation are critical when processing user-provided paths, environment variables, and external configuration to prevent path traversal, injection attacks, and unauthorized file access
- Pattern detected across 7 files with 90.47% confidence, indicating a consistent architectural approach to environment configuration and validation
- The facet 'security.input_validation' suggests this pattern specifically addresses secure handling of external configuration inputs

## Problem Statement

Without standardized validation and resolution of environment variables and configuration paths, the system is vulnerable to security issues including path traversal attacks, injection vulnerabilities, and inconsistent behavior across different runtime environments. Additionally, lack of consistent error handling for missing or invalid configuration can lead to runtime failures that are difficult to diagnose and debug.

## Decision

1. MUST: Configuration resolution logic MUST provide clear error messages indicating which environment variable or configuration parameter is missing or invalid

## Policy Block

- MUST All environment variables used for configuration MUST be validated before use, checking for null/undefined values and appropriate data types
- MUST File system paths derived from environment variables or user input MUST be normalized and validated to prevent path traversal attacks
- MUST Configuration resolution logic MUST provide clear error messages indicating which environment variable or configuration parameter is missing or invalid
- SHOULD Platform-specific path resolution (e.g., HOME directory, application data directories) SHOULD use dedicated utility functions rather than inline string concatenation
- SHOULD Default values for optional configuration parameters SHOULD be explicitly documented and consistently applied across the codebase
- MUST_NOT Configuration code MUST NOT trust user-provided paths without validation and sanitization
- MAY Configuration modules MAY cache resolved and validated configuration values to avoid repeated validation overhead

In scope:
- All environment variable access for runtime configuration
- File system path resolution from environment variables or user input
- Platform-specific configuration directory resolution (HOME, AppData, Library, etc.)
- Configuration validation in CLI tools, browser automation, and utility packages
- Cookie storage path resolution and browser profile path handling

Out of scope:
- Build-time configuration and compile-time constants
- Hard-coded internal paths that are not derived from external input
- Test fixtures and mock configuration in test suites
- Third-party library configuration that is managed by the library itself

Exceptions:
- EXC-001: Development and testing environments where security validation may be relaxed for debugging purposes

## Rationale

- The pattern appears consistently across 7 files spanning CLI utilities, browser automation, iOS simulator integration, and cookie management, indicating a proven architectural approach to configuration management
- The security.input_validation facet classification demonstrates that this pattern specifically addresses security concerns around external input handling, which is critical for preventing common vulnerabilities
- Standardizing environment variable validation and path resolution reduces code duplication and ensures consistent security posture across all runtime environments
- Clear error messages for configuration issues significantly improve developer experience and reduce time spent debugging environment-related failures

## Consequences

Positive:
- Improved security posture by preventing path traversal, injection attacks, and unauthorized file access through validated configuration inputs
- Consistent error handling and messaging across the codebase makes configuration issues easier to diagnose and resolve
- Reduced code duplication through shared validation and path resolution utilities
- Better cross-platform compatibility by standardizing platform-specific path resolution logic

Negative:
- Additional validation overhead may introduce minor performance impact during configuration initialization
- Stricter validation rules may break existing code that relies on lenient configuration handling
- Developers must learn and follow standardized configuration patterns rather than ad-hoc approaches
- Initial implementation effort required to refactor existing configuration code to meet validation standards

## Alternatives

- Use a third-party configuration management library (e.g., dotenv, config, convict) with built-in validation (rejected)
  Rejected because: Third-party libraries add dependencies and may not provide the specific platform-specific path resolution logic required for browser automation and iOS simulator integration. Custom validation provides more control over error messages and security policies.
  When valid: Could be reconsidered if the codebase grows significantly and requires more sophisticated configuration features like schema validation, type coercion, and hierarchical configuration merging
- Implement configuration validation only at application entry points rather than throughout the codebase (rejected)
  Rejected because: Validation only at entry points creates a false sense of security and doesn't protect against configuration issues introduced by internal code paths or dynamic configuration changes at runtime
  When valid: May be appropriate for simple CLI tools with minimal configuration requirements and no dynamic configuration changes
- Use TypeScript strict null checks and type guards for configuration validation without explicit validation functions (deferred)
  Rejected because: While TypeScript provides compile-time type safety, it doesn't prevent runtime issues with invalid environment variable values, malformed paths, or security vulnerabilities. However, TypeScript type guards can complement explicit validation.
  When valid: Should be used in conjunction with explicit validation to provide both compile-time and runtime safety

## Risks

- Overly strict validation rules may reject valid configuration in edge cases or unusual deployment environments
  Mitigation: Implement comprehensive test coverage for configuration validation across different platforms and environments. Provide clear documentation on validation rules and exception processes.
  Owner: Engineering team
- Performance degradation if validation is performed repeatedly for the same configuration values
  Mitigation: Implement caching for validated configuration values and ensure validation occurs only once during initialization or when configuration changes
  Owner: Engineering team
- Inconsistent application of validation standards across different modules and packages
  Mitigation: Create shared validation utility functions in a common package, enforce through code review, and add linting rules to detect non-compliant configuration access patterns
  Owner: Engineering team and code reviewers

## Implementation Notes

- Create a shared configuration utility module that exports validation functions for common patterns (path validation, environment variable access, platform-specific directory resolution)
- Use path.normalize() and path.resolve() for all file system paths, and validate that resolved paths don't escape expected boundaries
- Implement a consistent error handling pattern that includes the configuration parameter name, expected format, and actual value (sanitized to avoid leaking sensitive information)
- Document all environment variables used by the application in a central location (e.g., README or configuration documentation) with their purpose, expected format, and default values

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.js' | grep -v 'validate\|check\|sanitize' | wc -l
- grep -r 'path\.join.*process\.env' --include='*.ts' --include='*.js' | grep -v 'path\.normalize\|path\.resolve' | wc -l
- npm test -- --grep 'configuration.*validation'

Accept when:
- All direct access to process.env includes validation or uses a validated configuration wrapper
- All file system paths constructed from environment variables or user input are normalized and validated
- Configuration validation tests pass with 100% coverage for all environment variables and path resolution logic

## Enforcement

- Verified by: Automated code review checks for direct process.env access without validation
- Verified by: CI pipeline runs configuration validation tests on every pull request
- Verified by: Security-focused code review checklist includes verification of configuration validation patterns
- Violation handling: Pull requests with unvalidated environment variable access are blocked until validation is added
- Violation handling: Security team is notified of violations in production code for immediate remediation
- Violation handling: Violations are tracked in security audit logs and reviewed quarterly
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead reviews security implications and approves or rejects with documented reasoning
- Exception process: Approved exceptions are documented in code comments with expiration date and must be reviewed annually