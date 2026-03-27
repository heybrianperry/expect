# Adopt Browser-Based Rendering with rrweb for E2E Testing and Demo Recording

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Context

- The project requires capabilities for recording and replaying browser interactions for demonstration and testing purposes
- E2E testing infrastructure needs a reliable way to capture DOM state and user interactions without heavyweight video recording
- Browser-based rendering patterns detected across demo recording scripts, video replay utilities, and locator resolution mechanisms
- The testing.e2e facet indicates this pattern is specifically tied to end-to-end testing workflows where browser rendering fidelity is critical
- Pattern detected in 3 files with 91.67% confidence suggests a consistent architectural approach to browser rendering and interaction capture

## Problem Statement

How should the system handle browser rendering, interaction recording, and DOM state capture for E2E testing and demonstration purposes in a way that is lightweight, reproducible, and maintains high fidelity to actual user interactions?

## Decision

1. MUST: Browser rendering utilities MUST provide locator resolution capabilities to accurately identify and interact with DOM elements during replay

## Policy Block

- MUST E2E testing and demo recording implementations MUST use browser-based rendering with rrweb or equivalent DOM replay technology for capturing user interactions
- MUST Browser rendering utilities MUST provide locator resolution capabilities to accurately identify and interact with DOM elements during replay
- MUST Recording scripts MUST capture sufficient DOM state and mutation information to enable faithful replay of user sessions
- SHOULD Demo recording implementations SHOULD separate recording logic from replay logic to enable independent testing and optimization
- SHOULD Browser rendering modules SHOULD provide utilities for resolving locators across different selector strategies (CSS, XPath, text content)
- MAY Implementations MAY extend rrweb with custom event types or plugins to capture application-specific interactions

In scope:
- E2E test recording and replay infrastructure
- Demo recording scripts and automation
- Browser-based rendering utilities for testing
- DOM locator resolution mechanisms
- User interaction capture and playback systems

Out of scope:
- Unit tests that do not require browser rendering
- Server-side rendering (SSR) implementations
- Static site generation
- Native mobile app rendering
- Video-based screen recording (as opposed to DOM replay)

## Rationale

- rrweb provides lightweight DOM-based recording that is more efficient than video capture while maintaining perfect fidelity for web interactions
- Browser-based rendering with locator resolution enables reliable E2E testing by accurately identifying elements across test runs
- Pattern detected across 3 files (record-demo.ts, rrvideo.ts, resolve-locator.ts) indicates a cohesive architectural approach to browser rendering for testing
- The testing.e2e facet association confirms this pattern is specifically designed for end-to-end testing workflows where browser rendering accuracy is paramount

## Consequences

Positive:
- Lightweight recording mechanism that captures DOM mutations rather than pixel data, reducing storage and bandwidth requirements
- High-fidelity replay of user interactions enables accurate debugging and demonstration of application behavior
- Locator resolution utilities provide robust element identification across different selector strategies
- Separation of recording and replay logic enables independent optimization and testing of each component

Negative:
- Dependency on rrweb or similar libraries introduces external dependency that must be maintained and updated
- DOM-based recording may not capture certain visual effects or animations that depend on timing or external resources
- Locator resolution complexity increases with dynamic or heavily JavaScript-manipulated DOM structures
- Replay fidelity depends on consistent browser environments and may fail if DOM structure changes significantly

## Alternatives

- Use traditional video-based screen recording for demos and testing (rejected)
  Rejected because: Video recording produces large files, lacks interactivity, and cannot provide programmatic access to DOM elements for testing assertions
  When valid: When visual fidelity of animations and effects is more important than file size or interactivity
- Implement custom DOM snapshot and replay system without third-party libraries (rejected)
  Rejected because: Building a custom solution would require significant engineering effort to match the maturity and edge case handling of established libraries like rrweb
  When valid: When project has unique requirements that cannot be met by existing libraries or when avoiding external dependencies is critical
- Use Playwright or Cypress native recording features exclusively (rejected)
  Rejected because: Test framework native features may not provide the same level of portability and replay capabilities outside the testing context (e.g., for demos)
  When valid: When recordings are only needed within the testing framework and do not need to be shared or replayed independently

## Risks

- rrweb library may become unmaintained or introduce breaking changes in future versions
  Mitigation: Monitor library health, maintain version pinning, and evaluate alternative DOM replay libraries periodically
  Owner: Frontend Engineering Team
- Complex or dynamically generated DOM structures may cause locator resolution failures during replay
  Mitigation: Implement fallback locator strategies, add comprehensive error handling, and maintain test data fixtures that cover edge cases
  Owner: QA Engineering Team
- Recording overhead may impact application performance during E2E test execution
  Mitigation: Profile recording performance, implement selective recording for critical paths only, and optimize mutation observer configurations
  Owner: Performance Engineering Team

## Implementation Notes

- Ensure rrweb is properly initialized before any user interactions that need to be captured, with appropriate configuration for mutation observers and event listeners
- Implement locator resolution with multiple fallback strategies (CSS selector, XPath, text content, data attributes) to maximize reliability
- Structure recording scripts to separate concerns: recording logic, storage/serialization, and replay logic should be in distinct modules
- Add comprehensive error handling for replay failures, including logging of DOM state mismatches and locator resolution failures
- Consider implementing a recording sanitization step to remove sensitive data (passwords, tokens) before storing or sharing recordings

## Continuation Context


Verify commands:
- grep -r "rrweb" packages/browser/src/ apps/website/scripts/ --include="*.ts" --include="*.js"
- grep -r "record.*demo\|rrvideo\|resolve-locator" packages/browser/src/ apps/website/scripts/ --include="*.ts"
- npm list rrweb || yarn list --pattern rrweb

Accept when:
- rrweb or equivalent DOM replay library is present in project dependencies
- At least one recording script and one replay utility exist in the codebase
- Locator resolution utilities are implemented with support for multiple selector strategies
- E2E tests or demo scripts successfully use browser-based rendering for interaction capture

## Enforcement

- Verified by: Automated dependency scanning in CI pipeline to verify rrweb presence
- Verified by: Code review checklist requiring browser rendering approach for new E2E tests
- Verified by: Architecture review for any new recording or replay implementations
- Violation handling: Pull requests introducing alternative rendering approaches without justification will be flagged for architecture review
- Violation handling: E2E tests that do not follow the browser-based rendering pattern will fail architecture compliance checks
- Violation handling: Violations will be documented and reviewed in sprint retrospectives to understand if pattern needs refinement
- Exception process: Submit exception request to architecture review board with justification for alternative approach
- Exception process: Document specific requirements that cannot be met by current browser rendering pattern
- Exception process: Approved exceptions must be documented in ADR updates or supplementary decision records