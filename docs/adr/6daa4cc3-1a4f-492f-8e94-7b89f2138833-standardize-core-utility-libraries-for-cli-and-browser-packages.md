# Standardize Core Utility Libraries for CLI and Browser Packages

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple CLI applications and browser packages that require common utility functions for formatting, data manipulation, and system interaction
- Pattern detected across 12 files with 87.73% confidence, indicating consistent architectural approach to library organization
- Core utilities are distributed across apps/cli/src/utils/, apps/cli/src/stores/, and packages/ directories, suggesting a modular library structure
- Files include specialized utilities for syntax highlighting, time formatting, image processing, clipboard operations, and state management
- The pattern shows integration with external libraries (query-client, webdriver-client) and custom utility modules for cross-cutting concerns

## Problem Statement

Without standardized core utility libraries, teams may duplicate functionality across packages, create inconsistent APIs, or introduce unnecessary dependencies. This leads to maintenance overhead, larger bundle sizes, and difficulty in ensuring consistent behavior across the application ecosystem.

## Decision

1. MUST: All utility functions MUST be organized into dedicated modules within a utils/ directory structure

## Policy Block

- MUST All utility functions MUST be organized into dedicated modules within a utils/ directory structure
- MUST Shared utilities used across multiple packages MUST be extracted into the packages/ directory as reusable modules
- MUST Each utility module MUST have a single, well-defined responsibility (e.g., formatting, clipboard operations, image processing)
- SHOULD State management utilities SHOULD be organized in a separate stores/ directory when using state management patterns
- SHOULD Utility modules SHOULD export pure functions without side effects where possible to improve testability
- MUST_NOT Utility modules MUST NOT contain business logic specific to individual features; they must remain generic and reusable
- MAY Application-specific utilities MAY remain in the application's utils/ directory if they are not needed by other packages

In scope:
- All utility functions in apps/cli/src/utils/
- All state management stores in apps/cli/src/stores/
- All shared libraries in packages/browser/src/ and packages/cookies/src/
- Client wrappers for external services (query-client, webdriver-client)
- Cross-cutting concerns like formatting, clipboard operations, and display management

Out of scope:
- Feature-specific business logic components
- UI components and presentation layer code
- API route handlers and backend services
- Configuration files and environment-specific settings
- Third-party library code not wrapped by internal utilities

## Rationale

- The detected pattern shows consistent organization of utilities across 12 files with high confidence (87.73%), indicating this is an established architectural practice
- Modular utility libraries reduce code duplication and provide a single source of truth for common operations like time formatting, syntax highlighting, and clipboard management
- Separating utilities into packages/ enables code reuse across multiple applications while keeping application-specific utilities isolated in their respective apps
- This pattern aligns with separation of concerns principles and makes the codebase more maintainable and testable

## Consequences

Positive:
- Reduced code duplication across applications and packages through shared utility modules
- Improved maintainability with clear separation between generic utilities and application-specific code
- Enhanced testability through isolated, single-responsibility utility functions
- Easier onboarding for new developers with consistent utility organization patterns
- Smaller bundle sizes when utilities are properly tree-shaken and not duplicated

Negative:
- Additional overhead in determining whether a utility should be app-specific or shared in packages/
- Potential for over-abstraction if utilities are made too generic to accommodate all use cases
- Requires discipline to maintain the separation and prevent utility modules from accumulating unrelated functions
- May increase initial development time as developers need to locate or create appropriate utility modules

## Alternatives

- Inline all utility logic directly in feature components without extraction (rejected)
  Rejected because: Leads to significant code duplication, makes testing difficult, and violates DRY principles. The detected pattern shows clear preference for extracted utilities.
  When valid: Only for truly one-off operations that will never be reused
- Use a single monolithic utils.ts file for all utility functions (rejected)
  Rejected because: Creates a maintenance bottleneck, makes tree-shaking ineffective, and violates single responsibility principle. Evidence shows utilities are organized by function.
  When valid: Never recommended for codebases of this scale
- Rely exclusively on external npm packages for all utility needs (rejected)
  Rejected because: Increases dependency footprint, reduces control over implementation details, and may not provide domain-specific utilities needed. Pattern shows custom utilities alongside external libraries.
  When valid: For well-established, battle-tested utilities like date formatting libraries

## Risks

- Utility modules may become dumping grounds for miscellaneous functions without clear organization
  Mitigation: Enforce naming conventions and code review guidelines that require utilities to have clear, single responsibilities. Regularly audit utility modules for cohesion.
  Owner: Engineering team leads
- Circular dependencies may emerge between utility modules and application code
  Mitigation: Implement linting rules to detect circular dependencies. Ensure utilities depend only on other utilities or external libraries, never on application code.
  Owner: Engineering team
- Shared utilities in packages/ may introduce breaking changes affecting multiple applications
  Mitigation: Apply semantic versioning to shared packages, maintain comprehensive test coverage, and use dependency version pinning in consuming applications.
  Owner: Package maintainers

## Implementation Notes

- Create a utils/ directory in each application for app-specific utilities and a packages/ directory for shared utilities
- Name utility files descriptively based on their function (e.g., format-elapsed-time.ts, copy-to-clipboard.ts, highlighter.ts)
- For state management utilities, use a separate stores/ directory with clear naming conventions (e.g., use-project-preferences.ts, use-preferences.ts)
- When extracting a utility to packages/, ensure it has comprehensive tests and documentation before other applications depend on it
- Use barrel exports (index.ts) in utility directories to provide clean import paths for consumers

## Continuation Context


Verify commands:
- find apps/cli/src/utils -type f -name '*.ts' | wc -l | grep -E '[0-9]+'
- grep -r 'export.*function' apps/cli/src/utils/ packages/*/src/ | grep -v 'node_modules'
- find packages/*/src -type f -name '*.ts' | head -5

Accept when:
- Utility functions are organized in dedicated utils/ directories within applications
- Shared utilities exist in packages/ directory and are imported by multiple applications
- Each utility module has a clear, single responsibility reflected in its filename
- No business logic specific to individual features exists in utility modules

## Enforcement

- Verified by: Code review process checking for proper utility organization and single responsibility
- Verified by: Automated linting rules detecting circular dependencies and improper imports
- Verified by: Architecture decision review for new utility modules added to packages/
- Verified by: Periodic audits of utility directories to ensure cohesion and prevent bloat
- Violation handling: Code review feedback requesting reorganization of misplaced utilities
- Violation handling: CI pipeline failures for circular dependency violations
- Violation handling: Refactoring tickets created for utilities that have grown beyond single responsibility
- Violation handling: Documentation updates required for any exceptions to the standard pattern
- Exception process: Document the exception rationale in code comments or ADR amendments
- Exception process: Obtain approval from tech lead or architecture review board for structural exceptions
- Exception process: Create a tracking ticket to revisit the exception and evaluate if it can be resolved
- Exception process: Update this ADR with documented exceptions if they become permanent patterns