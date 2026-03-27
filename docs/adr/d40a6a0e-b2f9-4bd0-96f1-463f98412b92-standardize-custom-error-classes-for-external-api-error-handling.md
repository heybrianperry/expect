# Standardize Custom Error Classes for External API Error Handling

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple packages and applications that expose public/external APIs to consumers, including CLI tools, browser integrations, and cookie management utilities
- External API consumers require clear, structured error information to handle failures gracefully and implement appropriate retry logic or user feedback mechanisms
- Standard JavaScript Error objects lack the semantic richness needed to distinguish between different failure modes (validation errors, network errors, permission errors, etc.)
- A consistent error handling pattern across all public APIs improves developer experience, reduces integration friction, and enables better error monitoring and debugging
- The pattern was detected across 4 files with 90% confidence, indicating widespread adoption of custom error classes in external-facing components

## Problem Statement

External API consumers need a standardized, predictable way to handle errors that provides sufficient context for debugging, appropriate error categorization for programmatic handling, and consistent structure across all public/external API surfaces. Without custom error classes, APIs would return generic Error objects that lack semantic meaning and make it difficult for consumers to distinguish between recoverable and non-recoverable failures.

## Decision

1. MUST: Custom error classes MUST be exported from dedicated error modules (e.g., errors.ts) to ensure they are accessible to API consumers

## Policy Block

- MUST All public/external API modules MUST define custom error classes that extend the base Error class for domain-specific error conditions
- MUST Custom error classes MUST be exported from dedicated error modules (e.g., errors.ts) to ensure they are accessible to API consumers
- MUST Custom error classes MUST set the error name property to a descriptive, unique identifier that clearly indicates the error type
- SHOULD Custom error classes SHOULD include additional context properties beyond the standard message field (e.g., error codes, validation details, HTTP status codes)
- SHOULD Error modules SHOULD be co-located with the API implementation in the same package to maintain cohesion
- MUST Public API functions MUST throw custom error instances rather than generic Error objects or string literals
- MAY Custom error classes MAY implement additional methods for error serialization, logging, or transformation to support observability requirements

In scope:
- All packages exposing public/external APIs (packages/cookies, packages/browser, etc.)
- CLI applications with external-facing commands and utilities (apps/cli)
- SDK modules and libraries intended for third-party consumption
- API endpoints and integration points that cross package boundaries

Out of scope:
- Internal utility functions not exposed in public API surfaces
- Private implementation details within a single module
- Test utilities and mock implementations
- Temporary or experimental features not yet stabilized for external use

## Rationale

- The pattern signature (eff7ce064ab2e3f219859e6512c7e811) was detected across 4 files in external-facing components with 90% confidence and 90% significance, indicating this is an established architectural pattern
- Custom error classes provide type safety and enable consumers to use instanceof checks for error handling, improving code reliability and reducing runtime errors
- Dedicated error modules create a clear contract between API providers and consumers, making it explicit what error conditions can occur and how to handle them
- This pattern aligns with industry best practices for API design and is consistent with error handling patterns in major JavaScript/TypeScript libraries and frameworks

## Consequences

Positive:
- API consumers can implement precise error handling logic using instanceof checks and access to structured error properties
- Error messages and types are self-documenting, reducing the need for extensive error handling documentation
- Monitoring and observability tools can categorize and aggregate errors more effectively based on error class names
- Type safety in TypeScript projects is improved, with compile-time checking of error handling code

Negative:
- Increases the initial development overhead for new APIs, requiring definition of error classes before implementation
- Creates maintenance burden as error classes must be kept in sync with API changes and versioned appropriately
- May lead to error class proliferation if not carefully managed, with too many granular error types
- Requires API consumers to import and understand multiple error types rather than handling a single Error type

## Alternatives

- Use standard Error objects with error codes in the message or as a property (rejected)
  Rejected because: Error codes as strings or numbers lack type safety and require consumers to parse or check magic values, leading to brittle error handling code. Custom classes provide compile-time safety and clearer semantics.
  When valid: May be acceptable for internal APIs with limited error conditions or legacy codebases where introducing custom classes would be disruptive
- Return error objects as values (Result/Either pattern) instead of throwing exceptions (rejected)
  Rejected because: While the Result pattern has benefits, it represents a fundamental shift in API design that would be inconsistent with existing JavaScript/TypeScript ecosystem conventions and would require all consumers to adopt the pattern.
  When valid: Appropriate for functional programming contexts or when designing new APIs from scratch with explicit error handling requirements
- Use a single custom error class with an error type discriminator property (rejected)
  Rejected because: A single error class with type discriminators loses the benefits of instanceof checks and type narrowing in TypeScript, making error handling more verbose and error-prone.
  When valid: May be suitable for simple APIs with only 2-3 error conditions where the overhead of multiple classes is not justified

## Risks

- Error class proliferation leads to an unwieldy number of error types that are difficult for consumers to understand and handle comprehensively
  Mitigation: Establish guidelines for error granularity, grouping related errors into base classes with subclasses only when consumers need to handle them differently. Document common error handling patterns.
  Owner: API Design Team
- Breaking changes to error classes (renaming, restructuring) can break consumer code that relies on instanceof checks or specific error properties
  Mitigation: Treat error classes as part of the public API contract, version them appropriately, and provide deprecation warnings before removing or changing error types. Use semantic versioning for packages.
  Owner: Engineering Team
- Inconsistent error class implementations across different packages lead to fragmented developer experience and confusion
  Mitigation: Create shared base error classes or utilities in a common package, establish error class naming conventions, and include error handling examples in API documentation.
  Owner: Platform Team

## Implementation Notes

- Create a base custom error class that extends Error and properly sets the prototype chain for instanceof checks to work correctly across different JavaScript environments
- Use TypeScript's class syntax with proper type annotations for error properties to enable type checking in consumer code
- Export all custom error classes from a dedicated errors.ts module at the package root or alongside the main API entry point
- Include JSDoc comments on error classes documenting when they are thrown and what properties they contain
- Consider creating error factory functions for complex error construction scenarios to encapsulate error creation logic

## Continuation Context


Verify commands:
- grep -r "class.*Error extends Error" packages/*/src/errors.ts apps/*/src/**/errors.ts
- grep -r "export class.*Error" packages/*/src/errors.ts apps/*/src/**/errors.ts
- find packages apps -name 'errors.ts' -type f | xargs grep -l "extends Error"

Accept when:
- All public/external API packages contain an errors.ts module that exports at least one custom error class
- Custom error classes properly extend Error and set the name property to a descriptive identifier
- Public API functions throw custom error instances rather than generic Error objects in documented error conditions

## Enforcement

- Verified by: Automated code review checks using AST analysis to detect throw statements with generic Error objects in public API modules
- Verified by: Manual code review during pull request approval focusing on error handling patterns in new or modified public APIs
- Verified by: Static analysis tools configured to flag missing error exports in packages with public API surfaces
- Violation handling: Pull requests introducing new public APIs without custom error classes are flagged for revision during code review
- Violation handling: Existing violations are tracked as technical debt items and prioritized for refactoring based on API usage and stability
- Violation handling: Documentation is updated to reflect the error handling standard and examples are provided for common patterns
- Exception process: Exceptions may be granted for simple utility functions with single, obvious error conditions where custom errors add no value
- Exception process: Exception requests must be documented in the API module with a comment explaining why the standard does not apply
- Exception process: Exceptions are reviewed quarterly to determine if they should be converted to follow the standard as the API evolves