# Standardize Utility Module Exports with Single-Purpose Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all library and utility module development across the codebase.

## Context

- The codebase contains numerous utility modules and library components that provide reusable functionality across CLI applications, browser packages, and cookie management systems
- Analysis of 11 files reveals a consistent pattern of single-purpose utility modules with focused exports, including query clients, formatters, highlighters, and specialized helpers
- The pattern demonstrates high consistency (87.53% confidence) across different application domains including CLI tools, browser automation, and state management
- Current architecture favors small, composable modules over large monolithic utility libraries, enabling better tree-shaking and dependency management
- The pattern spans multiple package boundaries (apps/cli, packages/browser, packages/cookies) indicating an organization-wide architectural preference

## Problem Statement

Without standardized guidelines for utility module design and export patterns, teams may create inconsistent library structures leading to poor discoverability, difficult maintenance, bloated bundle sizes, and unclear API contracts. The codebase needs explicit rules to maintain the observed pattern of focused, single-purpose utility modules with clear public interfaces.

## Decision

1. SHOULD: Library modules providing client interfaces (e.g., query-client.ts, webdriver-client.ts) SHOULD export factory functions or class constructors with clear initialization patterns

## Policy Block

- MUST Utility modules MUST export a single primary function or a cohesive set of related functions serving one specific purpose
- MUST Module file names MUST clearly describe their single responsibility using kebab-case (e.g., format-elapsed-time.ts, copy-to-clipboard.ts)
- MUST Public API exports MUST be explicitly defined using named exports; default exports SHOULD be avoided for utility functions
- SHOULD Utility modules SHOULD be organized in a dedicated utils/ or similar directory structure to maintain clear separation from business logic
- SHOULD Each utility module SHOULD have minimal external dependencies to maximize reusability and minimize coupling
- MUST_NOT Utility modules MUST NOT contain side effects at module initialization; all effects must be contained within exported functions
- SHOULD Library modules providing client interfaces (e.g., query-client.ts, webdriver-client.ts) SHOULD export factory functions or class constructors with clear initialization patterns
- MAY Utility modules MAY export TypeScript types and interfaces alongside functions when they form part of the public API contract

In scope:
- All utility functions in apps/cli/src/utils/
- All library modules in packages/*/src/ directories
- Store modules and state management utilities
- Client wrappers and API abstraction layers
- Formatting, parsing, and transformation utilities

Out of scope:
- React components and UI elements
- Application entry points and main files
- Configuration files and build scripts
- Test utilities and fixtures (may follow different patterns)
- Third-party library re-exports

Exceptions:
- EXC-001: A utility module requires multiple related exports that are always used together (e.g., a parser with its associated types and validators)
- EXC-002: Legacy modules being incrementally refactored

## Rationale

- Pattern detected across 11 files with 87.53% confidence indicates this is an established and successful architectural practice within the organization
- Single-purpose modules improve code discoverability, as developers can quickly locate functionality by scanning file names that match their intent
- Focused exports enable better tree-shaking in modern bundlers, reducing final bundle sizes by eliminating unused code paths
- Clear module boundaries reduce cognitive load during code review and maintenance, as each file has a well-defined scope and responsibility

## Consequences

Positive:
- Improved code discoverability through self-documenting file names and clear module purposes
- Better bundle optimization and tree-shaking capabilities leading to smaller production builds
- Reduced cognitive complexity during development and code review
- Enhanced testability through isolated, focused units of functionality
- Easier refactoring and maintenance due to clear module boundaries

Negative:
- Increased number of files in the codebase, potentially making initial navigation more complex for new developers
- Risk of over-fragmentation if taken to extremes, creating excessive indirection
- May require additional effort to identify related utilities that could be used together
- Import statements may become more verbose with many individual module imports

## Alternatives

- Monolithic utility library with all helpers in a single file or barrel export (rejected)
  Rejected because: Creates tight coupling, prevents effective tree-shaking, and makes it difficult to understand dependencies. The detected pattern explicitly avoids this approach.
  When valid: Only for very small projects with fewer than 5 utility functions
- Domain-grouped utility modules (e.g., string-utils.ts, array-utils.ts) with multiple related functions (rejected)
  Rejected because: While better than monolithic, still creates larger modules than necessary. The detected pattern shows preference for even finer granularity (e.g., format-elapsed-time.ts rather than time-utils.ts).
  When valid: When 3-5 functions are always used together and share significant implementation code
- Class-based utility modules with static methods (rejected)
  Rejected because: Adds unnecessary ceremony and complexity. The detected pattern favors simple function exports over class-based designs for utilities.
  When valid: When utilities require shared state or complex initialization logic

## Risks

- Over-fragmentation leading to excessive file proliferation and difficulty finding related functionality
  Mitigation: Establish clear naming conventions and directory structure. Use index files sparingly to group truly related utilities. Document common utility patterns in team wiki.
  Owner: Engineering team leads
- Inconsistent application of the pattern across different teams or packages
  Mitigation: Implement automated linting rules to enforce module structure. Include pattern examples in onboarding documentation. Conduct regular architecture reviews.
  Owner: Platform architecture team
- Difficulty refactoring when utilities need to be combined or split
  Mitigation: Maintain comprehensive test coverage for all utility modules. Use automated refactoring tools. Document module dependencies clearly.
  Owner: Individual development teams

## Implementation Notes

- When creating a new utility, ask: 'Does this do exactly one thing?' If the answer is no, consider splitting into multiple modules
- Use descriptive, verb-based names for utility files that clearly indicate their purpose (e.g., build-image-sequence.ts, clear-ink-display.ts)
- Place utility modules in a utils/ directory at the appropriate package level; avoid deeply nested utility directories
- Export TypeScript types alongside functions when they are part of the public API contract, but keep type definitions focused
- For client libraries (query-client.ts, webdriver-client.ts), prefer factory functions or clear initialization patterns over complex constructors

## Continuation Context


Verify commands:
- find apps/cli/src/utils -name '*.ts' -exec sh -c 'grep -L "export default" "$1" || exit 0' _ {} \;
- grep -r "export {" apps/cli/src/utils packages/*/src --include="*.ts" | wc -l
- find . -path '*/utils/*.ts' -o -path '*/src/*-client.ts' -o -path '*/src/layers.ts' | xargs -I {} sh -c 'echo "Checking: {}"; head -20 {}'
- eslint --rule 'import/no-default-export: error' 'apps/cli/src/utils/**/*.ts' 'packages/*/src/**/*-client.ts'

Accept when:
- All utility modules in utils/ directories use named exports exclusively (no default exports)
- Each utility module file name clearly describes a single responsibility using kebab-case
- Utility modules contain no side effects at module initialization (verified by static analysis)
- New utility modules pass code review checklist confirming single-purpose design

## Enforcement

- Verified by: ESLint rules enforcing named exports and module structure patterns
- Verified by: Code review checklist requiring verification of single-purpose design
- Verified by: Automated CI checks scanning for utility module compliance
- Verified by: Architecture review for new packages or major utility additions
- Violation handling: CI pipeline fails if ESLint rules detect default exports in utility modules
- Violation handling: Code review requires changes before approval if modules violate single-purpose principle
- Violation handling: Existing violations are tracked in technical debt backlog with refactoring plans
- Violation handling: Quarterly architecture reviews identify and prioritize cleanup of non-compliant modules
- Exception process: Developer documents exception rationale in module JSDoc comments
- Exception process: Exception request submitted to tech lead or architecture review board
- Exception process: Approved exceptions are logged in ADR amendments or architecture decision log
- Exception process: All exceptions require periodic review (every 6 months) to determine if still necessary