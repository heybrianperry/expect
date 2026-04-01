# Standardize React Hooks for UI State Management in CLI Components: Components Use Additional

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The CLI application uses React-based terminal UI components that require state management for interactive screens and user workflows
- Multiple screen components (port-picker-screen, cookie-sync-confirm-screen) and hooks (use-test-coverage) follow a consistent pattern of using React hooks for managing component state and side effects
- The codebase demonstrates a pattern of custom hooks for encapsulating complex logic (test coverage tracking) and screen components using standard React hooks for UI state
- Message queue boundaries are present in the facet analysis, suggesting asynchronous communication patterns that benefit from React's state management capabilities

## Problem Statement

CLI applications with interactive terminal UIs need a consistent approach to managing component state, handling user interactions, and coordinating asynchronous operations across multiple screens and workflows. Without standardized patterns, state management becomes fragmented and difficult to maintain.

## Decision

1. MAY: Components MAY use additional React hooks from the ecosystem (useMemo, useRef, useContext) when performance optimization or context access is required

## Policy Block

- MAY Components MAY use additional React hooks from the ecosystem (useMemo, useRef, useContext) when performance optimization or context access is required

In scope:
- All React-based CLI screen components in apps/cli/src/components/screens/
- Custom hooks in apps/cli/src/hooks/
- Interactive terminal UI components requiring state management
- Components handling user input, async operations, or multi-step workflows

Out of scope:
- Non-React CLI utilities and pure functions
- Backend services or API handlers
- Static configuration files
- Build scripts and tooling configuration

Exceptions:
- EXC-001: Integrating with third-party libraries that require class components or legacy patterns

## Rationale

- Pattern detected across 3 files with 88.67% confidence indicates established practice in the codebase
- React hooks provide a functional, composable approach to state management that aligns with modern React best practices and improves code reusability
- Custom hooks enable separation of concerns by isolating complex stateful logic (like test coverage tracking) from presentation logic
- The message queue facet boundary suggests asynchronous patterns that are well-served by useEffect and custom hooks for managing side effects

## Consequences

Positive:
- Consistent state management patterns across all CLI screen components improve code readability and maintainability
- Custom hooks enable better code reuse and testing isolation for complex stateful logic
- Functional components with hooks are more concise and easier to reason about than class-based alternatives
- Better alignment with React ecosystem best practices and modern tooling support

Negative:
- Developers unfamiliar with React hooks may face a learning curve when working on CLI components
- Hook dependency arrays and closure behavior can introduce subtle bugs if not properly understood
- Refactoring existing class components to hooks requires development effort
- Over-abstraction into custom hooks can sometimes obscure simple logic

## Alternatives

- Use class-based React components with lifecycle methods for state management (rejected)
  Rejected because: Class components are legacy patterns in React, more verbose, and don't support the composition benefits of hooks
  When valid: Only when integrating with legacy third-party libraries that require class components
- Use external state management libraries (Redux, MobX, Zustand) for all component state (rejected)
  Rejected because: Adds unnecessary complexity and dependencies for local component state; hooks provide sufficient state management for CLI UI needs
  When valid: Consider for truly global state that needs to be shared across many unrelated components
- Mix hooks and class components based on developer preference (rejected)
  Rejected because: Inconsistent patterns across the codebase reduce maintainability and create confusion about which approach to use
  When valid: Never; consistency is critical for team productivity

## Risks

- Improper hook dependency arrays leading to stale closures or infinite re-render loops
  Mitigation: Enable eslint-plugin-react-hooks with exhaustive-deps rule; conduct code reviews focusing on hook usage patterns
  Owner: Engineering team
- Over-engineering simple components with unnecessary custom hooks
  Mitigation: Establish guidelines for when to extract custom hooks (reused in 2+ places, or complex logic >50 lines); review hook abstractions in PR process
  Owner: Tech leads
- Performance issues from excessive re-renders due to improper hook usage
  Mitigation: Use React DevTools Profiler to identify performance bottlenecks; apply useMemo/useCallback judiciously for expensive operations
  Owner: Engineering team

## Implementation Notes

- Place custom hooks in apps/cli/src/hooks/ directory following the 'use-*.ts' naming convention
- Screen components should be placed in apps/cli/src/components/screens/ and use hooks for all state management
- Enable eslint-plugin-react-hooks in ESLint configuration to catch common hook mistakes at lint time
- Document complex custom hooks with JSDoc comments explaining parameters, return values, and usage examples
- Consider using TypeScript generics in custom hooks to maintain type safety across different use cases

## Continuation Context


Verify commands:
- grep -r 'class.*extends.*Component' apps/cli/src/components/screens/ | wc -l | grep -q '^0$'
- find apps/cli/src/hooks -name 'use-*.ts' -o -name 'use-*.tsx' | wc -l
- grep -r 'useState\|useEffect\|useCallback' apps/cli/src/components/screens/*.tsx | wc -l

Accept when:
- No class-based components exist in apps/cli/src/components/screens/ directory
- All custom hooks in apps/cli/src/hooks/ follow the 'use-*' naming convention
- Screen components demonstrate usage of React hooks (useState, useEffect, useCallback) for state management
- ESLint configuration includes react-hooks plugin with recommended rules enabled

## Enforcement

- Verified by: ESLint checks in CI pipeline with react-hooks plugin enabled
- Verified by: Code review checklist includes verification of hook usage patterns
- Verified by: Automated grep-based checks in pre-commit hooks to detect class components in screen directories
- Violation handling: CI build fails if ESLint detects hook rule violations
- Violation handling: PR reviews must address any class components introduced in screen directories
- Violation handling: Tech lead review required for any exceptions to the hooks-first approach
- Exception process: Document the technical constraint requiring an exception in the PR description
- Exception process: Obtain approval from tech lead or architecture review board
- Exception process: Add inline comments explaining the exception and any planned migration path
- Exception process: Track exceptions in technical debt backlog for future resolution