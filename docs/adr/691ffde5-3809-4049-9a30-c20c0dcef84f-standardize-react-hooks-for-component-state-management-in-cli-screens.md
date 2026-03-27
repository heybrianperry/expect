# Standardize React Hooks for Component State Management in CLI Screens

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The CLI application uses React-based screen components that require consistent state management patterns for user interactions and data handling
- Multiple screen components (cookie-sync-confirm-screen, port-picker-screen) demonstrate a pattern of using React hooks for managing component lifecycle and state
- Custom hooks like use-test-coverage have been developed to encapsulate reusable logic and separate concerns from presentation components
- The codebase shows evidence of message queue boundaries and asynchronous operations that need to be managed within React component lifecycles
- A consistent approach to hooks usage improves code maintainability, testability, and developer onboarding across the CLI application

## Problem Statement

The CLI application requires a standardized approach to state management and side effects in React-based screen components. Without clear patterns for using React hooks and custom hooks, developers may implement inconsistent solutions leading to code duplication, difficult-to-test components, and maintenance challenges. The detection of this pattern across 3 files with 88.67% confidence indicates an emerging architectural pattern that should be formalized.

## Decision

1. MAY: Screen components MAY compose multiple custom hooks to build complex functionality from simpler, tested units

## Policy Block

- MUST All CLI screen components MUST use React hooks (useState, useEffect, useCallback, etc.) for managing component state and side effects rather than class-based component lifecycle methods
- MUST Complex or reusable state logic MUST be extracted into custom hooks following the 'use-' naming convention (e.g., use-test-coverage)
- SHOULD Custom hooks SHOULD encapsulate a single concern or responsibility to promote reusability and testability
- MUST Screen components MUST separate presentation logic from business logic, with business logic encapsulated in custom hooks
- SHOULD Custom hooks that interact with message queues or asynchronous boundaries SHOULD handle cleanup in useEffect return functions to prevent memory leaks
- MAY Screen components MAY compose multiple custom hooks to build complex functionality from simpler, tested units
- MUST_NOT Screen components MUST NOT contain direct business logic or data fetching code; these MUST be delegated to custom hooks

In scope:
- All React-based screen components in the apps/cli/src/components/screens directory
- Custom hooks in the apps/cli/src/hooks directory
- Any new CLI screen components or interactive UI elements
- State management for user interactions, data fetching, and side effects in CLI screens

Out of scope:
- Non-React components or pure utility functions
- Backend services or API implementations
- Static configuration files or constants
- Third-party library implementations

Exceptions:
- EXC-001: Legacy screen components that are scheduled for deprecation within the next release cycle
- EXC-002: Simple presentational components with no state or side effects

## Rationale

- Pattern detected across 3 files (cookie-sync-confirm-screen.tsx, port-picker-screen.tsx, use-test-coverage.ts) with 88.67% confidence indicates this is an established architectural pattern in the codebase
- React hooks provide a more functional and composable approach to state management compared to class components, improving code readability and reducing boilerplate
- Custom hooks enable separation of concerns by extracting business logic from presentation, making components easier to test and maintain
- The presence of message queue boundaries in the facet data suggests asynchronous operations that are well-suited to the useEffect hook pattern for lifecycle management
- Standardizing on hooks aligns with modern React best practices and the direction of the React ecosystem

## Consequences

Positive:
- Improved code consistency across CLI screen components, reducing cognitive load for developers
- Enhanced testability through isolated custom hooks that can be unit tested independently of UI rendering
- Better code reusability as common patterns are extracted into shareable custom hooks
- Easier onboarding for new developers familiar with modern React patterns
- Reduced component complexity by separating presentation from business logic

Negative:
- Existing class-based components will need refactoring to comply with the new standard
- Developers unfamiliar with hooks patterns may require training and adjustment period
- Potential for over-abstraction if custom hooks are created prematurely before patterns are fully understood
- Additional files and indirection may make simple components appear more complex initially

## Alternatives

- Continue using class-based React components with lifecycle methods (rejected)
  Rejected because: Class components are verbose, harder to test, and not aligned with modern React ecosystem direction. The detected pattern shows the team has already moved toward hooks.
  When valid: Only for legacy components scheduled for deprecation
- Use external state management library (Redux, MobX, Zustand) (rejected)
  Rejected because: Adds unnecessary complexity and dependencies for CLI screen components with localized state. React hooks provide sufficient state management capabilities for the detected use cases.
  When valid: If global state sharing across many components becomes a requirement
- Mix hooks and class components based on developer preference (rejected)
  Rejected because: Inconsistent patterns increase maintenance burden and make codebase harder to understand. The detection of a clear pattern indicates standardization is beneficial.
  When valid: Never - consistency is critical for maintainability

## Risks

- Refactoring existing class components may introduce bugs or break existing functionality
  Mitigation: Implement comprehensive test coverage before refactoring, use incremental migration approach, and conduct thorough code reviews
  Owner: Engineering team
- Developers may create overly complex custom hooks that are difficult to understand and maintain
  Mitigation: Establish code review guidelines for custom hooks, provide examples and documentation, and enforce single-responsibility principle
  Owner: Tech Lead
- Memory leaks from improper cleanup in useEffect hooks, especially with message queue interactions
  Mitigation: Mandate cleanup functions in all useEffect hooks with subscriptions, implement linting rules to detect missing cleanup, and provide training on proper hook usage
  Owner: Engineering team

## Implementation Notes

- Create a hooks directory structure (apps/cli/src/hooks) to organize custom hooks by domain or functionality
- Establish naming conventions: use-[domain]-[action] (e.g., use-test-coverage, use-port-picker)
- Document common hook patterns in team wiki with examples from cookie-sync-confirm-screen and port-picker-screen
- Set up ESLint rules (eslint-plugin-react-hooks) to enforce hooks rules and detect common mistakes
- Create a migration guide for converting existing class components to functional components with hooks
- Implement unit tests for all custom hooks using @testing-library/react-hooks or similar testing utilities

## Continuation Context


Verify commands:
- grep -r 'class.*extends.*Component' apps/cli/src/components/screens/ | wc -l
- find apps/cli/src/hooks -name 'use-*.ts' -o -name 'use-*.tsx' | wc -l
- grep -r 'useState\|useEffect\|useCallback' apps/cli/src/components/screens/*.tsx | wc -l

Accept when:
- All screen components in apps/cli/src/components/screens use functional components with hooks (class component count is 0 or only in documented exceptions)
- Custom hooks directory contains at least 3 reusable hooks following the use-* naming convention
- Screen components demonstrate separation of concerns with business logic in custom hooks and presentation in component files

## Enforcement

- Verified by: Automated ESLint checks in CI pipeline using eslint-plugin-react-hooks
- Verified by: Code review checklist requiring verification of hooks usage patterns
- Verified by: Periodic architecture reviews examining screen component structure
- Violation handling: CI pipeline fails if ESLint hooks rules are violated
- Violation handling: Pull requests with class-based screen components are rejected unless exception is documented
- Violation handling: Violations identified in code review must be addressed before merge approval
- Exception process: Developer submits exception request to Tech Lead with justification and migration plan
- Exception process: Tech Lead reviews request against policy exception criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions must be documented in code comments with EXCEPTION: prefix and ticket reference
- Exception process: Exceptions are reviewed quarterly and migration plans are tracked in backlog