# Standardize on TypeScript Test Files with .test.ts Extension

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all TypeScript test file creation and organization within the codebase.

## Context

- The codebase contains multiple packages and applications (apps/cli, packages/browser, packages/agent) that require consistent test organization and discovery mechanisms
- Test frameworks and CI/CD pipelines rely on predictable file naming patterns to automatically discover and execute test suites without manual configuration
- TypeScript projects benefit from explicit test file identification through naming conventions that distinguish test code from production code
- Evidence shows consistent adoption of .test.ts extension across 4 files spanning different packages, indicating an emergent standard pattern with 91.07% confidence

## Problem Statement

Without a standardized test file naming convention, test discovery becomes unreliable, CI/CD pipelines require manual configuration for each new test file, and developers lack clear guidance on where to place test code. This leads to inconsistent test organization, potential gaps in test coverage reporting, and increased maintenance overhead.

## Decision

1. SHOULD: Each package or application SHOULD maintain its own tests/ directory for test isolation

## Policy Block

- MUST All TypeScript test files MUST use the .test.ts file extension
- MUST Test files MUST be placed in a tests/ directory at the appropriate package or application level
- SHOULD Test file names SHOULD reflect the module or feature being tested (e.g., agent.test.ts for agent functionality)
- MUST_NOT Test files MUST NOT use alternative extensions such as .spec.ts, .test.js, or _test.ts
- SHOULD Each package or application SHOULD maintain its own tests/ directory for test isolation
- MAY Placeholder test files (e.g., placeholder.test.ts) MAY be used temporarily during initial project setup

In scope:
- All TypeScript test files across all packages and applications
- Unit tests, integration tests, and end-to-end tests written in TypeScript
- Test files for CLI applications, browser packages, and agent packages
- New test files created during development

Out of scope:
- JavaScript test files in legacy codebases (if any)
- Test files in non-TypeScript languages
- Documentation or example files that are not executable tests
- Third-party test files in node_modules

Exceptions:
- EXC-001: Migrating from a legacy codebase that uses .spec.ts convention
- EXC-002: Third-party framework explicitly requires different naming convention

## Rationale

- Pattern detected across 4 files with 91.07% confidence indicates strong organic adoption and proven effectiveness in the existing codebase
- The .test.ts extension is widely recognized by popular TypeScript testing frameworks (Jest, Vitest, Mocha) and provides automatic test discovery without additional configuration
- Consistent naming conventions reduce cognitive load for developers moving between packages and enable reliable glob patterns for CI/CD pipelines
- Separation of test files into dedicated tests/ directories maintains clear boundaries between production and test code while supporting package-level test isolation

## Consequences

Positive:
- Automated test discovery works reliably across all packages without manual configuration
- CI/CD pipelines can use simple glob patterns (e.g., **/*.test.ts) to identify and execute all tests
- Developers have clear, consistent guidance on test file naming and organization
- IDE and editor tooling can automatically recognize and provide appropriate support for test files
- Test coverage reporting tools can accurately identify test files versus production code

Negative:
- Existing codebases using .spec.ts or other conventions will require migration effort
- Teams familiar with alternative conventions (e.g., .spec.ts from Angular) may face initial adjustment period
- Strict enforcement may create friction during rapid prototyping or experimentation phases

## Alternatives

- Use .spec.ts extension following Angular and Jasmine conventions (rejected)
  Rejected because: Pattern detection shows no evidence of .spec.ts usage in the codebase, and .test.ts is already the established convention with 91% confidence
  When valid: Valid for Angular-specific projects or when migrating from Angular ecosystem
- Co-locate test files with source files using .test.ts suffix (rejected)
  Rejected because: Evidence shows consistent use of separate tests/ directories, which provides clearer separation and easier test file management
  When valid: Valid for small single-file modules where co-location improves discoverability
- Allow multiple test file extensions (.test.ts, .spec.ts, .test.tsx) based on developer preference (rejected)
  Rejected because: Multiple conventions would undermine consistency, complicate test discovery patterns, and increase cognitive load
  When valid: Not recommended; consistency is more valuable than flexibility in this context

## Risks

- Developers may inadvertently create test files with non-standard extensions, causing tests to be silently skipped in CI/CD
  Mitigation: Implement linting rules and CI checks to detect non-compliant test file names; provide clear error messages guiding developers to the standard
  Owner: Engineering team / DevOps
- Migration of existing non-compliant test files may be overlooked, creating inconsistency
  Mitigation: Run automated scan to identify all existing test files; create migration checklist; track completion in project management tool
  Owner: Engineering team leads
- Third-party tools or frameworks may have conflicting expectations for test file naming
  Mitigation: Document any framework-specific requirements; configure test runners explicitly when needed; maintain exception list for justified cases
  Owner: Platform team

## Implementation Notes

- Update test runner configuration (Jest, Vitest, etc.) to explicitly include **/*.test.ts pattern in testMatch or similar configuration
- Add ESLint or custom linting rules to enforce test file naming conventions and flag violations during development
- Update project templates and scaffolding tools to generate test files with .test.ts extension by default
- Document the standard in developer onboarding materials and contribution guidelines with examples from existing codebase (init.test.ts, agent.test.ts, diff.test.ts)
- Consider adding pre-commit hooks to validate test file naming before code is committed

## Continuation Context


Verify commands:
- find . -type f -name '*.test.ts' | wc -l
- find . -type f \( -name '*.spec.ts' -o -name '*_test.ts' \) -not -path '*/node_modules/*' | wc -l
- grep -r 'testMatch.*\.test\.ts' --include='*.config.*' --include='package.json'

Accept when:
- All TypeScript test files use .test.ts extension (first command returns count > 0, second command returns 0)
- No test files exist with alternative extensions like .spec.ts or _test.ts outside of node_modules
- Test runner configuration explicitly includes .test.ts pattern in testMatch or equivalent setting
- CI/CD pipeline successfully discovers and executes all test files using .test.ts glob pattern

## Enforcement

- Verified by: Automated CI/CD checks scanning for non-compliant test file extensions
- Verified by: ESLint or custom linting rules enforcing file naming conventions
- Verified by: Code review checklist including test file naming verification
- Verified by: Pre-commit hooks validating test file patterns
- Violation handling: CI/CD pipeline fails with clear error message identifying non-compliant files
- Violation handling: Linter warnings or errors displayed in IDE and during pre-commit
- Violation handling: Code review requires correction before approval
- Violation handling: Automated PR comments flag violations with links to this ADR
- Exception process: Developer submits exception request to tech lead with justification
- Exception process: Tech lead reviews against policy exceptions (EXC-001, EXC-002)
- Exception process: If approved, exception is documented in project README or ADR amendments
- Exception process: Exception is time-bound with review date for potential migration