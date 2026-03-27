# Standardize Public API Error Handling and Event Contracts

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public-facing API modules, external integration points, and client-facing SDK components. It applies to error handling, event contracts, and constant definitions exposed to external consumers.

## Context

- The codebase contains multiple public-facing API surfaces including browser SDKs, CLI tools, and MCP (Model Context Protocol) integrations that require consistent contract definitions
- External consumers depend on stable error codes, event schemas, and constant definitions across package boundaries (browser, cookies, iOS, CLI)
- Pattern detected across 6 files with 89.17% confidence, indicating a systematic approach to defining public API contracts through dedicated modules for errors, constants, and events
- The facet 'api.public.contracts' suggests explicit contract definition files that serve as the interface boundary between internal implementation and external consumers
- Cross-platform support (browser, iOS, CLI) necessitates consistent error handling and event communication patterns that work across different runtime environments

## Problem Statement

Without standardized contract definitions for public APIs, external consumers face inconsistent error handling, unpredictable event schemas, and fragile integrations. The lack of centralized constant definitions and error taxonomies leads to breaking changes, poor developer experience, and increased support burden as consumers must handle platform-specific variations.

## Decision

1. SHOULD: Public API contracts SHOULD be versioned and changes to error codes, constants, or event schemas SHOULD follow semantic versioning principles

## Policy Block

- MUST All public-facing API modules MUST define their error contracts in dedicated error definition files (e.g., errors.ts) that export typed error classes or error code constants
- MUST Public API constants that form part of the external contract MUST be defined in dedicated constants files (e.g., constants.ts) separate from implementation logic
- MUST Event schemas for public APIs MUST be defined in dedicated event definition files (e.g., viewer-events.ts) with explicit type definitions for all event payloads
- MUST Error definitions MUST include machine-readable error codes or types that remain stable across versions to enable programmatic error handling by consumers
- SHOULD Public API contracts SHOULD be versioned and changes to error codes, constants, or event schemas SHOULD follow semantic versioning principles
- SHOULD Contract definition files SHOULD be co-located with their respective package modules (e.g., packages/browser/src/*/errors.ts) to maintain clear ownership
- MUST_NOT Public API contract definitions MUST NOT contain implementation logic or business rules; they MUST only define the interface contract
- MAY Packages MAY provide utility functions for extracting or transforming contract artifacts (e.g., extract-close-artifacts.ts) as long as the core contracts remain stable

In scope:
- All packages exposing public APIs to external consumers (browser SDKs, CLI tools, public npm packages)
- Error handling contracts for cross-package boundaries
- Event schemas for MCP and viewer integrations
- Constant definitions used in public API signatures or responses
- Platform-specific API implementations (iOS, browser) that share common contract patterns

Out of scope:
- Internal-only modules and utilities not exposed to external consumers
- Private implementation details within a package
- Development and testing utilities
- Internal event buses or messaging systems not exposed as public APIs

Exceptions:
- EXC-001: A package is explicitly marked as internal-only and will never be published or consumed externally
- EXC-002: Rapid prototyping or experimental features marked as alpha/unstable with explicit warnings

## Rationale

- Pattern detected across 6 files with 89.17% confidence indicates this is an established architectural practice worth codifying
- Dedicated contract files (errors.ts, constants.ts, events.ts) provide a clear API surface that can be reviewed, versioned, and documented independently of implementation
- Cross-platform consistency (browser, iOS, CLI) is critical for developer experience and reduces integration friction for consumers using multiple packages
- Separating contracts from implementation enables better backward compatibility management and makes breaking changes more visible during code review

## Consequences

Positive:
- External consumers gain predictable, stable error handling and event contracts that reduce integration bugs
- Contract files serve as living documentation of the public API surface, improving discoverability
- Easier to maintain backward compatibility by isolating contract changes from implementation changes
- Cross-platform consistency improves developer experience for consumers using multiple packages
- Automated tooling can validate contract stability and detect breaking changes in CI/CD pipelines

Negative:
- Additional files and structure overhead for maintaining separate contract definitions
- Potential duplication if similar contracts exist across multiple packages without shared abstractions
- Requires discipline to keep contracts minimal and avoid leaking implementation details
- May slow down rapid iteration if contract changes require additional review processes

## Alternatives

- Inline error definitions and constants within implementation files without dedicated contract modules (rejected)
  Rejected because: Makes it difficult to identify the public API surface, increases risk of accidental breaking changes, and provides no clear boundary for external consumers to reference
  When valid: Only appropriate for internal-only modules with no external consumers
- Centralize all error codes and constants in a single shared package used by all modules (rejected)
  Rejected because: Creates tight coupling between packages and makes it difficult to version contracts independently; however, common base error types could be shared
  When valid: Valid for truly cross-cutting error types like network errors or authentication errors that span all packages
- Generate contract definitions from OpenAPI/JSON Schema specifications (deferred)
  Rejected because: Not rejected but deferred; could complement this approach for REST APIs but doesn't address SDK-level contracts, events, and platform-specific errors
  When valid: Should be considered for HTTP-based public APIs as an additional layer of contract definition

## Risks

- Contract drift where implementation diverges from documented contracts without detection
  Mitigation: Implement automated contract testing that validates actual API behavior against contract definitions; use TypeScript strict mode to enforce type contracts
  Owner: Engineering team + QA
- Proliferation of similar but inconsistent contract patterns across packages
  Mitigation: Establish shared contract base types in a common package; conduct periodic architecture reviews to identify and consolidate patterns
  Owner: Architecture team
- Breaking changes introduced unintentionally when modifying contract files
  Mitigation: Implement breaking change detection in CI using tools like API Extractor or custom linting; require architecture review for contract modifications
  Owner: DevOps + Architecture team

## Implementation Notes

- Create or identify contract files following naming conventions: errors.ts for error definitions, constants.ts for public constants, *-events.ts for event schemas
- Use TypeScript interfaces and type exports to define contracts; leverage discriminated unions for error types and event types
- Document each error code, constant, and event type with JSDoc comments explaining when it's used and what consumers should expect
- Consider using const enums or as const assertions for error codes and constants to enable tree-shaking and type safety
- Add contract files to package.json exports map to make them explicitly importable by consumers
- Implement contract validation tests that ensure error codes are unique and event schemas match runtime payloads

## Continuation Context


Verify commands:
- find . -path '*/src/*/errors.ts' -o -path '*/src/*/constants.ts' -o -path '*/src/*-events.ts' | grep -E '(packages|apps)' | wc -l
- grep -r 'export.*Error' --include='errors.ts' packages/ apps/ | head -5
- grep -r 'export const' --include='constants.ts' packages/ apps/ | head -5

Accept when:
- Each public-facing package contains at least one contract definition file (errors.ts, constants.ts, or events.ts) in its source tree
- Contract files export only type definitions, interfaces, constants, or error classes without implementation logic
- Error definitions include stable identifiers (error codes or discriminated type fields) that enable programmatic handling
- Contract files are referenced in package documentation or API reference materials

## Enforcement

- Verified by: Automated linting rules that detect contract violations (e.g., implementation logic in contract files)
- Verified by: Code review checklist requiring architecture review for changes to contract files
- Verified by: CI pipeline checks for breaking changes using API surface comparison tools
- Verified by: Periodic architecture audits reviewing contract consistency across packages
- Violation handling: CI build fails if contract files contain implementation logic (detected via linting rules)
- Violation handling: Pull requests modifying contract files are automatically tagged for architecture review
- Violation handling: Breaking changes to contracts require explicit version bump and changelog entry
- Violation handling: Violations identified in audits are tracked as technical debt items with remediation plans
- Exception process: Request exception via architecture review meeting with justification for deviation
- Exception process: Document exception in ADR or architecture decision log with expiration date
- Exception process: Exceptions for experimental features require @alpha/@experimental annotations and sunset timeline
- Exception process: Annual review of all active exceptions to determine if they should be regularized or removed