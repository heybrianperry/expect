# Standardize Browser Runtime and Cookie Management as Public API Surface

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all browser automation packages and public API modules that expose runtime interfaces or cookie management capabilities to external consumers.

## Context

- The codebase contains multiple packages (browser, cookies) that expose runtime interfaces and configuration APIs for browser automation and cookie management
- These packages provide public-facing APIs that external consumers depend on for browser automation workflows, including CDP (Chrome DevTools Protocol) integration and cross-browser cookie handling
- The pattern appears across 4 files with high significance (89.58%), indicating a consistent architectural approach to exposing browser automation capabilities as stable public interfaces
- Browser runtime and cookie management require careful API design to balance flexibility for diverse use cases while maintaining stability and backward compatibility
- The packages integrate with multiple browser engines (Chromium, CDP) and need to abstract implementation details while providing predictable, well-documented interfaces

## Problem Statement

External consumers of browser automation libraries need stable, well-defined public APIs for runtime management and cookie operations, but without clear architectural boundaries between public interfaces and internal implementation details, the API surface can become unstable, difficult to version, and prone to breaking changes that disrupt downstream integrations.

## Decision

1. SHOULD: Configuration objects for browser features SHOULD use TypeScript interfaces to provide compile-time type safety for API consumers

## Policy Block

- MUST Browser runtime packages MUST export a clearly defined public API surface through dedicated index.ts entry points that serve as the contract for external consumers
- MUST Cookie management APIs MUST provide browser-agnostic configuration interfaces that abstract underlying protocol implementations (CDP, native APIs)
- MUST Public API modules MUST maintain semantic versioning and document breaking changes when modifying exported interfaces
- SHOULD Browser-specific implementation details (CDP clients, Chromium-specific logic) SHOULD be encapsulated in separate modules and not directly exposed in public APIs
- SHOULD Configuration objects for browser features SHOULD use TypeScript interfaces to provide compile-time type safety for API consumers
- MUST_NOT Public API packages MUST NOT expose internal protocol-specific types or implementation classes that could create tight coupling with specific browser engines
- MAY Packages MAY provide advanced configuration options for power users while maintaining sensible defaults for common use cases

In scope:
- All packages under packages/browser/* that export runtime interfaces
- All packages under packages/cookies/* that provide cookie management APIs
- Public-facing TypeScript interfaces and configuration objects
- Entry point modules (index.ts) that define the external API contract
- Browser automation APIs consumed by external applications or test frameworks

Out of scope:
- Internal utility functions not exported through package entry points
- Private implementation details of protocol handlers (CDP, WebDriver)
- Development-only tools and test utilities
- Experimental or unstable APIs marked with appropriate warnings
- Third-party browser driver implementations

Exceptions:
- EXC-001: Advanced users need direct access to CDP client for protocol-level operations not covered by the public API
- EXC-002: Browser-specific optimizations require exposing engine-specific configuration for performance-critical scenarios

## Rationale

- The pattern detected across 4 files with 89.58% confidence indicates a deliberate architectural decision to structure browser automation capabilities as public APIs with clear boundaries
- Separating runtime interfaces (packages/browser/src/runtime/index.ts) from implementation details (cdp-client.ts, chromium.ts) enables independent evolution of internal implementations without breaking external consumers
- Browser-agnostic configuration patterns (browser-config.ts) allow the codebase to support multiple browser engines while presenting a unified API surface to consumers
- This architecture supports the common use case of browser automation libraries that need to balance power-user flexibility with stability for production integrations

## Consequences

Positive:
- External consumers gain stable, predictable APIs that can be versioned and evolved without frequent breaking changes
- Clear separation between public interfaces and internal implementation enables refactoring and optimization of browser protocol handlers without affecting API consumers
- TypeScript interfaces provide compile-time safety and excellent IDE support for developers integrating the browser automation libraries
- Browser-agnostic abstractions allow adding support for new browser engines without changing the public API contract

Negative:
- Additional abstraction layers may introduce slight performance overhead compared to direct protocol access
- Maintaining backward compatibility constraints can slow down internal refactoring and modernization efforts
- Power users may find the abstracted APIs limiting for advanced use cases requiring direct protocol manipulation
- Documentation burden increases as both public APIs and their underlying implementations need to be maintained

## Alternatives

- Expose CDP and browser protocol clients directly as the primary API without abstraction layers (rejected)
  Rejected because: Creates tight coupling to specific browser implementations, makes it difficult to support multiple browsers, and exposes consumers to protocol-level breaking changes
  When valid: For internal tools or single-browser scenarios where maximum control and minimal abstraction are required
- Use a plugin architecture where all browser-specific functionality is loaded dynamically at runtime (rejected)
  Rejected because: Adds complexity to the API surface, makes type safety harder to achieve, and increases runtime overhead for common operations
  When valid: For highly extensible frameworks where supporting unknown browser engines at build time is a core requirement
- Provide both high-level abstractions and low-level protocol access through separate package exports (accepted)
  When valid: This is the current approach, allowing common use cases through stable APIs while providing escape hatches for advanced users

## Risks

- Public API abstractions may not cover all use cases, forcing consumers to work around limitations or fork the codebase
  Mitigation: Provide extension points and document advanced usage patterns; gather feedback from early adopters to identify missing capabilities before stabilizing APIs
  Owner: API design team and browser automation maintainers
- Browser protocol changes (CDP updates, WebDriver spec changes) may require breaking changes to public APIs
  Mitigation: Design APIs to be protocol-agnostic where possible; use adapter patterns to isolate protocol-specific logic; maintain compatibility layers during transitions
  Owner: Engineering team
- Performance-sensitive applications may be impacted by abstraction overhead in cookie operations or runtime management
  Mitigation: Profile critical paths and optimize hot code paths; provide direct access options for performance-critical scenarios with appropriate documentation
  Owner: Performance engineering team

## Implementation Notes

- Use index.ts files as the single source of truth for public exports; all external imports should go through these entry points
- Apply TypeScript's 'export type' for type-only exports to minimize runtime bundle size while maintaining type safety
- Document all public interfaces with JSDoc comments including @public, @param, @returns, and usage examples
- Implement integration tests that consume the public API as an external package would, ensuring the API contract remains stable
- Consider using API Extractor or similar tools to generate API reports and detect unintentional breaking changes during CI

## Continuation Context


Verify commands:
- grep -r "export.*from.*runtime" packages/browser/src/index.ts packages/browser/src/runtime/index.ts
- grep -r "export.*Config" packages/cookies/src/browser-config.ts packages/cookies/src/index.ts
- find packages/browser packages/cookies -name 'index.ts' -exec grep -L 'export' {} \; | wc -l | grep -q '^0$'

Accept when:
- All browser and cookie packages have index.ts entry points that explicitly export public APIs
- Implementation files (cdp-client.ts, chromium.ts) are not directly imported by external consumers and are only accessed through public interfaces
- TypeScript compilation succeeds with strict mode enabled and all public APIs have complete type definitions
- API documentation exists for all exported interfaces and breaking changes are tracked in CHANGELOG

## Enforcement

- Verified by: Automated CI checks using API Extractor or TypeScript API Guardian to detect breaking changes
- Verified by: Code review process verifying that new exports through index.ts are intentional and documented
- Verified by: Integration tests that import packages as external consumers would, catching unintended API surface changes
- Violation handling: CI pipeline fails if breaking changes are detected without corresponding major version bump
- Violation handling: Pull requests that modify public API surface require architecture review approval
- Violation handling: Violations discovered post-merge trigger immediate review and either revert or expedited patch release with migration guide
- Exception process: Request exception through architecture review board with justification for API change
- Exception process: Document exception in ADR amendments with rationale and impact assessment
- Exception process: For approved exceptions, create migration guide and deprecation timeline before implementing breaking change