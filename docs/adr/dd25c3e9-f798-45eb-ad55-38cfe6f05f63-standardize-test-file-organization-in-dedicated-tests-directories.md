# Standardize Test File Organization in Dedicated Tests Directories

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple packages and applications with test files organized in dedicated 'tests' directories, indicating a consistent testing infrastructure pattern
- Test files are located in structured paths like 'apps/cli/tests/', 'packages/browser/tests/', and 'packages/cookies/tests/', suggesting a monorepo architecture with standardized test organization
- The pattern appears across 4 different files with high significance (89-92%), indicating this is an established architectural convention rather than an isolated occurrence
- Consistent test directory structure facilitates CI/CD pipeline configuration, test discovery, and build tooling integration across the entire codebase
- The facet 'data.access.patterns' suggests this relates to how test resources and test data are accessed and organized within the build and delivery workflow

## Problem Statement

Without a standardized approach to test file organization, CI/CD pipelines become fragmented with package-specific configurations, test discovery becomes unreliable, and build tooling requires custom logic for each package. This increases maintenance overhead and creates inconsistencies in how tests are executed across different parts of the codebase.

## Decision

1. MUST: CI/CD pipeline configurations MUST reference the standardized 'tests' directory pattern for test execution

## Policy Block

- MUST All test files MUST be placed in a dedicated 'tests' directory at the package or application root level
- MUST Test file names MUST follow the pattern '*.test.ts' or '*.test.js' to enable consistent test discovery
- SHOULD Test directories SHOULD mirror the structure of the source code they test for maintainability
- MUST CI/CD pipeline configurations MUST reference the standardized 'tests' directory pattern for test execution
- SHOULD Test utilities and fixtures SHOULD be organized in subdirectories within the tests directory (e.g., 'tests/fixtures', 'tests/helpers')
- MUST_NOT Test files MUST NOT be co-located with source files in the same directory unless explicitly justified and documented

In scope:
- All packages within the monorepo (packages/*)
- All applications within the monorepo (apps/*)
- Unit tests, integration tests, and end-to-end tests
- Test utilities, fixtures, and helper functions
- CI/CD pipeline test execution stages

Out of scope:
- Documentation examples that are not executed as tests
- Build scripts and tooling configuration files
- Third-party dependencies and node_modules
- Generated code or artifacts in dist/build directories

Exceptions:
- EXC-001: Legacy packages undergoing migration may temporarily maintain alternative test structures

## Rationale

- Pattern detected across 4 files with 90.60% confidence indicates this is an established and successful architectural convention in the codebase
- Standardized test organization enables CI/CD pipelines to use uniform glob patterns and test discovery mechanisms, reducing configuration complexity
- Dedicated test directories provide clear separation of concerns between production code and test code, improving code organization and build optimization
- Consistent structure across packages in a monorepo reduces cognitive load for developers working across multiple packages and enables shared tooling configurations

## Consequences

Positive:
- CI/CD pipelines can use consistent glob patterns (e.g., '**/tests/**/*.test.ts') for test discovery across all packages
- Build tools can easily exclude test directories from production builds using simple path-based rules
- Developers can quickly locate and navigate test files following a predictable directory structure
- Test coverage reporting and analysis tools can reliably identify test files across the entire codebase
- Monorepo tooling can apply uniform test execution strategies across all packages

Negative:
- Existing packages with different test organization patterns will require migration effort
- Some developers may prefer co-located tests for certain types of components, requiring justification for exceptions
- Additional directory nesting may slightly increase import path lengths in test files
- Strict enforcement may create friction for rapid prototyping or experimental packages

## Alternatives

- Co-locate test files alongside source files with .test.ts suffix (e.g., component.ts and component.test.ts in same directory) (rejected)
  Rejected because: Makes it harder to exclude tests from production builds, increases directory clutter, and complicates CI/CD glob patterns that need to distinguish between source and test files
  When valid: May be acceptable for small, standalone packages with simple build configurations and no monorepo constraints
- Use a top-level 'test' directory at the monorepo root containing all tests for all packages (rejected)
  Rejected because: Breaks package encapsulation, makes it difficult to run tests for individual packages, and complicates dependency management in a monorepo
  When valid: Could work for small projects with a single package and no plans for growth
- Allow each package to define its own test organization strategy without standardization (rejected)
  Rejected because: Creates inconsistency across the codebase, requires package-specific CI/CD configurations, and increases cognitive load for developers working across packages
  When valid: Only appropriate for loosely coupled repositories with independent deployment pipelines

## Risks

- Migration of existing packages with non-standard test organization may introduce test failures or coverage gaps
  Mitigation: Implement gradual migration with validation steps, maintain test coverage metrics before and after migration, and use automated tooling to move files
  Owner: Engineering team leads
- Developers may inadvertently create tests outside the standard directory structure, especially in new packages
  Mitigation: Implement linting rules and CI checks that fail if test files are detected outside approved directories, provide package scaffolding templates
  Owner: DevOps and tooling team
- Third-party tools or frameworks may have expectations about test file locations that conflict with this standard
  Mitigation: Document configuration overrides for common tools, evaluate tool compatibility during adoption decisions, maintain flexibility for justified exceptions
  Owner: Architecture team

## Implementation Notes

- Update package scaffolding templates and generators to create the 'tests' directory structure by default
- Configure test runners (Jest, Vitest, etc.) with glob patterns that explicitly target '**/tests/**/*.test.{ts,js}' for consistency
- Update build tool configurations (TypeScript, webpack, etc.) to exclude 'tests' directories from production builds
- Create migration scripts to help move existing test files to the standard structure while preserving git history
- Document the standard in the project's contributing guidelines with examples from each package type (app vs library)

## Continuation Context


Verify commands:
- find . -name '*.test.ts' -o -name '*.test.js' | grep -v node_modules | grep -v '/tests/' && echo 'FAIL: Test files found outside tests directory' || echo 'PASS'
- test -d apps/cli/tests && test -d packages/browser/tests && test -d packages/cookies/tests && echo 'PASS: Standard test directories exist' || echo 'FAIL'
- grep -r "testMatch\|testRegex" package.json tsconfig.json jest.config.* vitest.config.* 2>/dev/null | grep -q 'tests/.*\.test' && echo 'PASS: Test configs reference tests directory' || echo 'WARN'

Accept when:
- All test files with .test.ts or .test.js extensions are located within a 'tests' directory at the package or app level
- CI/CD pipeline configurations successfully discover and execute tests using the standardized directory pattern
- Build outputs (dist, build directories) do not contain any files from tests directories
- New packages created from templates automatically include the standard tests directory structure

## Enforcement

- Verified by: Automated CI checks that scan for test files outside the standard tests directory structure
- Verified by: Code review checklist items requiring verification of test file locations for new packages
- Verified by: Linting rules configured to warn or error on non-standard test file locations
- Verified by: Periodic architecture audits reviewing test organization across all packages
- Violation handling: CI pipeline fails if test files are detected outside approved directory structures
- Violation handling: Pull requests with non-compliant test organization are blocked from merging until corrected
- Violation handling: Automated notifications sent to package owners when violations are detected in existing code
- Violation handling: Quarterly reports generated showing compliance status across all packages with remediation plans
- Exception process: Submit exception request to architecture team with justification and impact analysis
- Exception process: Document the exception in the package README with rationale and any compensating controls
- Exception process: Exceptions are reviewed quarterly to determine if they can be resolved or if the standard should be updated
- Exception process: All exceptions must have an assigned owner and expiration date or ongoing justification