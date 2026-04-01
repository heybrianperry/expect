# Adopt Browser-Based Rendering with Locator Resolution for UI Components: Browser Rendering Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The browser package implements a specialized rendering architecture that requires dynamic resolution of UI component locators at runtime
- Three core modules (resolve-locator.ts, browser.ts, rrvideo.ts) demonstrate a consistent pattern of browser-based rendering with locator resolution mechanisms
- The pattern appears in public API contracts (api.public.contracts facet), indicating this is an intentional architectural boundary exposed to consumers
- The rendering model needs to support video replay (rrvideo) and general browser interactions, requiring a unified locator resolution strategy

## Problem Statement

Frontend applications require a consistent and reliable method to locate, resolve, and render UI components across different contexts (standard browser interactions and video replay scenarios). Without a standardized rendering model with locator resolution, component identification becomes fragile, leading to brittle tests, inconsistent user interactions, and maintenance challenges across the browser package.

## Decision

1. SHOULD: Browser rendering modules SHOULD expose locator resolution as part of their public API contracts for external consumers

## Policy Block

- SHOULD Browser rendering modules SHOULD expose locator resolution as part of their public API contracts for external consumers

In scope:
- All modules within the packages/browser/src directory
- Public API contracts exposed by the browser package
- Standard browser interaction rendering
- Video replay rendering (rrvideo module)
- Utility functions for locator resolution

Out of scope:
- Server-side rendering implementations
- Non-browser UI frameworks (native mobile, desktop applications)
- Static HTML generation without dynamic locator resolution
- Third-party browser automation tools with their own locator strategies

## Rationale

- The pattern was detected across 3 files with 91.23% confidence, indicating a deliberate architectural choice rather than coincidental code organization
- Centralizing locator resolution in a dedicated utility module promotes code reuse and reduces the risk of inconsistent component identification across different rendering contexts
- Exposing locator resolution through public API contracts enables external consumers to integrate with the browser package's rendering model predictably
- Supporting both standard browser interactions and video replay scenarios with a unified approach simplifies maintenance and ensures behavioral consistency

## Consequences

Positive:
- Consistent component locator resolution across all browser rendering contexts reduces bugs and improves reliability
- Centralized locator resolution logic in utility modules makes the codebase more maintainable and easier to test
- Public API contracts for locator resolution enable external tools and frameworks to integrate seamlessly with the browser package
- Unified rendering model between standard browser operations and video replay simplifies mental model for developers

Negative:
- Tight coupling between browser rendering and locator resolution utilities may make it harder to swap out locator strategies in the future
- Additional abstraction layer for locator resolution adds complexity compared to direct DOM queries
- Public API contracts create long-term maintenance obligations and limit flexibility to refactor internal implementations
- Performance overhead from locator resolution abstraction may impact rendering speed in high-frequency interaction scenarios

## Alternatives

- Use native browser APIs (querySelector, getElementById) directly in each rendering context without abstraction (rejected)
  Rejected because: Direct DOM queries lead to duplicated locator logic across modules, making it difficult to maintain consistency between standard browser rendering and video replay scenarios
  When valid: For simple single-context applications that do not require video replay or complex component resolution
- Adopt a third-party locator resolution library (e.g., Playwright locators, Selenium WebDriver strategies) (rejected)
  Rejected because: Third-party libraries may not support the specific requirements of video replay rendering and would introduce external dependencies with potential version conflicts
  When valid: When the browser package is exclusively used for standard browser automation without video replay requirements
- Implement separate locator resolution strategies for each rendering context (browser vs. video replay) (rejected)
  Rejected because: Separate strategies would lead to behavioral inconsistencies and increased maintenance burden when locator logic needs to be updated
  When valid: When rendering contexts have fundamentally incompatible locator requirements that cannot be unified

## Risks

- Changes to the locator resolution utility could break multiple rendering contexts simultaneously due to tight coupling
  Mitigation: Implement comprehensive integration tests covering all rendering contexts (browser.ts, rrvideo.ts) that verify locator resolution behavior. Use semantic versioning and deprecation warnings for public API changes.
  Owner: Frontend Engineering Team
- Performance degradation in high-frequency rendering scenarios due to abstraction overhead in locator resolution
  Mitigation: Profile locator resolution performance in realistic scenarios. Implement caching strategies for frequently resolved locators. Consider lazy evaluation for expensive resolution operations.
  Owner: Performance Engineering Team
- Public API contracts for locator resolution may limit future architectural flexibility if requirements change significantly
  Mitigation: Design the public API with extensibility in mind using plugin or strategy patterns. Document clear versioning and migration paths. Maintain internal flexibility behind stable public interfaces.
  Owner: API Design Team

## Implementation Notes

- Ensure all new browser rendering features import and use the resolve-locator utility module rather than implementing custom locator logic
- When extending the rrvideo module, verify that locator resolution behavior matches the standard browser.ts implementation through shared test cases
- Document the public API contracts for locator resolution with clear examples showing how external consumers should integrate with the browser package
- Consider implementing a locator resolution strategy interface to allow future extensibility while maintaining the current centralized architecture

## Continuation Context


Verify commands:
- grep -r 'import.*resolve-locator' packages/browser/src/ | wc -l
- grep -r 'querySelector\|getElementById' packages/browser/src/*.ts | grep -v resolve-locator.ts | wc -l
- test -f packages/browser/src/utils/resolve-locator.ts && test -f packages/browser/src/browser.ts && test -f packages/browser/src/rrvideo.ts

Accept when:
- The resolve-locator utility module is imported and used by both browser.ts and rrvideo.ts modules
- Direct DOM query methods (querySelector, getElementById) are minimized outside the resolve-locator utility, with fewer than 3 occurrences in core rendering modules
- All three core files (resolve-locator.ts, browser.ts, rrvideo.ts) exist and maintain the locator resolution pattern

## Enforcement

- Verified by: Automated static analysis in CI pipeline checking for imports of resolve-locator utility in browser rendering modules
- Verified by: Code review checklist requiring verification that new rendering features use centralized locator resolution
- Verified by: Integration tests validating consistent locator resolution behavior across browser.ts and rrvideo.ts contexts
- Violation handling: CI pipeline fails if direct DOM queries are added to core rendering modules without using resolve-locator utility
- Violation handling: Pull requests introducing new rendering logic without proper locator resolution are flagged for architectural review
- Violation handling: Quarterly audits identify and refactor any locator resolution code that has diverged from the standard pattern
- Exception process: Exceptions must be requested through an architecture review board with justification for why centralized locator resolution cannot be used
- Exception process: Approved exceptions must be documented in code comments with ADR reference and expiration date for re-evaluation
- Exception process: Exception requests require demonstration that the alternative approach has been prototyped and provides measurable benefits