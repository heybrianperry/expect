# Standardize Environment Variable Access Through Centralized Configuration Constants: Environment Variables Read

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The CLI application requires access to environment-specific configuration values across multiple modules and execution contexts
- Direct access to process.env throughout the codebase creates scattered configuration logic that is difficult to test, validate, and maintain
- Build-time configuration (vite.config.ts) and runtime configuration (constants.ts, audit utilities) need consistent patterns for accessing environment variables
- Type safety and validation of environment variables is critical for preventing runtime errors in production deployments
- The pattern was detected across 3 files with 89.47% confidence, indicating a deliberate architectural choice for configuration management

## Problem Statement

Applications that access environment variables directly throughout the codebase suffer from poor testability, lack of type safety, inconsistent validation, and difficulty tracking which configuration values are actually used. This creates maintenance burden and increases the risk of runtime failures due to missing or invalid environment variables.

## Decision

1. MUST: Environment variables MUST be read and validated at application initialization time, not lazily throughout execution

## Policy Block

- MUST Environment variables MUST be read and validated at application initialization time, not lazily throughout execution

In scope:
- All CLI application modules requiring configuration values
- Build tool configuration files (vite.config.ts, webpack.config.js, etc.)
- Runtime utilities and services that depend on environment-specific behavior
- Test fixtures and mocks that need to override configuration values

Out of scope:
- Third-party libraries that manage their own configuration
- Development-only scripts that explicitly document direct env access
- Infrastructure deployment scripts outside the application codebase

Exceptions:
- EXC-001: Build tool configuration files (vite.config.ts) that must access environment variables to configure the build process
- EXC-002: Bootstrap/initialization code that sets up the configuration system itself

## Rationale

- Centralized configuration provides a single source of truth for all environment-dependent values, making it easier to understand what configuration the application requires
- Type-safe configuration exports enable compile-time checking and IDE autocomplete, reducing runtime errors and improving developer experience
- Early validation of required environment variables enables fail-fast behavior, preventing partial application startup with missing configuration
- The pattern detected across constants.ts, vite.config.ts, and audit.ts demonstrates consistent application of this approach across build-time and runtime contexts

## Consequences

Positive:
- Improved testability through centralized mocking and stubbing of configuration values
- Enhanced type safety with explicit TypeScript types for all configuration values
- Better documentation of required environment variables in a single location
- Easier validation and transformation of raw environment strings into typed values
- Reduced risk of typos in environment variable names through constant references

Negative:
- Additional indirection when accessing configuration values (import from constants vs direct process.env)
- Potential for circular dependencies if configuration module imports other application modules
- Requires discipline to maintain the pattern and prevent direct environment access creep
- May require refactoring existing code that directly accesses process.env

## Alternatives

- Direct process.env access throughout the codebase (rejected)
  Rejected because: Creates scattered configuration logic, poor testability, no type safety, and makes it difficult to track which environment variables are actually required
  When valid: Only appropriate for trivial scripts or prototypes with minimal configuration needs
- Runtime configuration service with lazy loading (rejected)
  Rejected because: Lazy loading delays validation and can cause runtime failures deep in execution; adds complexity without significant benefit for CLI applications
  When valid: May be appropriate for long-running services where configuration can change at runtime
- Configuration schema validation library (e.g., zod, joi) (deferred)
  Rejected because: Not rejected, but not detected in current pattern; could be complementary to centralized constants approach
  When valid: Recommended for complex configuration with nested objects, unions, or sophisticated validation rules

## Risks

- Developers may bypass the centralized configuration and directly access process.env, degrading the pattern over time
  Mitigation: Implement linting rules to detect direct process.env access; enforce through code review and CI checks
  Owner: Engineering team
- Circular dependency issues if configuration module needs to import other application modules
  Mitigation: Keep configuration module as a leaf dependency with no imports from application code; only import from external libraries
  Owner: Engineering team
- Build-time vs runtime configuration divergence if patterns are not consistently applied
  Mitigation: Document and enforce consistent patterns for both build-time (vite.config.ts) and runtime (constants.ts) configuration access
  Owner: Engineering team

## Implementation Notes

- Create a constants.ts or config.ts module at the application root that exports all configuration values as typed constants
- Use TypeScript const assertions and explicit types to ensure type safety (e.g., const API_URL: string = process.env.API_URL || '')
- Implement validation logic at module initialization to check for required variables and throw descriptive errors if missing
- For build tools like Vite, document environment variable access in comments and consider extracting to a separate config section
- Provide a .env.example file documenting all required and optional environment variables with descriptions

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' --include='*.ts' --include='*.js' --exclude='*constants.ts' --exclude='*config.ts' --exclude='vite.config.ts' apps/cli/src/
- grep -r 'import\.meta\.env\.' --include='*.ts' --exclude='vite.config.ts' apps/cli/src/
- test -f apps/cli/src/constants.ts && echo 'Configuration module exists' || echo 'Missing constants.ts'

Accept when:
- No direct process.env or import.meta.env access found in application code outside of designated configuration modules
- All configuration values are exported from a centralized constants or config module with explicit types
- Required environment variables are validated at application startup with clear error messages

## Enforcement

- Verified by: ESLint rule to detect direct process.env access outside configuration modules
- Verified by: Code review checklist requiring configuration changes to go through centralized module
- Verified by: CI pipeline grep checks for process.env patterns in non-configuration files
- Violation handling: CI build fails if direct environment access is detected outside approved files
- Violation handling: Pull requests with violations are blocked until refactored to use centralized configuration
- Violation handling: Existing violations are tracked as technical debt items for remediation
- Exception process: Developer documents the specific reason why centralized configuration cannot be used
- Exception process: Architecture review approves the exception with documented justification
- Exception process: Exception is added to ESLint ignore list with inline comment explaining the rationale
- Exception process: Exception is reviewed quarterly to determine if it can be eliminated