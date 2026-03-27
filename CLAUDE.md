---

## Architecture Decision Records

<adr_governance source="docs/adr/">
ADRs govern validated architectural standards for this project.
Full ADR documents: @docs/adr/
</adr_governance>

<activation>
These directives are ALWAYS ACTIVE. Claude Code MUST apply all rules in this
document to every code generation, modification, and review action within this
project. No exceptions unless explicitly noted per-rule.
</activation>

---

### Verification Protocol

<verification_protocol>
All rules in this document follow the **Verify → Fix → Repeat** loop.
</verification_protocol>

After generating or modifying code for any rule, Claude Code MUST:

1. **RUN** the targeted verification command(s) in the rule's **Verify** block.
2. **CAPTURE** the full command output (stdout + stderr).
3. **EVALUATE** whether the **Accept when** criteria are satisfied.
4. **IF FAILING:** diagnose the root cause, apply a fix, and re-run from step 1.
5. **IF PASSING:** include the passing output as inline evidence before proposing further changes.
6. **MAX ITERATIONS:** 5 attempts per rule. If still failing after 5 attempts, STOP and report the failure with all captured outputs.

<enforcement>
Compliance is not optional. Claude Code must not skip verification steps, assume
correctness, or defer verification to a later task. Evidence of a passing
verification run must accompany every code change that touches a governed area.
</enforcement>

---

## Standardize React Hooks for Component State Management in CLI Screens

1. Screen components MUST NOT contain direct business logic or data fetching code; these MUST be delegated to custom hooks
2. Screen components MAY compose multiple custom hooks to build complex functionality from simpler, tested units
3. Custom hooks that interact with message queues or asynchronous boundaries SHOULD handle cleanup in useEffect return functions to prevent memory leaks
4. Screen components MUST separate presentation logic from business logic, with business logic encapsulated in custom hooks
5. Custom hooks SHOULD encapsulate a single concern or responsibility to promote reusability and testability
6. Complex or reusable state logic MUST be extracted into custom hooks following the 'use-' naming convention (e.g., use-test-coverage)
7. All CLI screen components MUST use React hooks (useState, useEffect, useCallback, etc.) for managing component state and side effects rather than class-based component lifecycle methods

---

## Adopt Message Queue Integration Pattern for External System Communication

1. Message queue consumers MAY implement caching strategies for frequently accessed external API data to reduce latency and external service load
2. Teams MAY implement circuit breaker patterns to prevent overwhelming failing external services
3. Message queue implementations SHOULD provide observability through logging, metrics, and tracing for external API interactions
4. External API responses SHOULD be validated and sanitized before being passed to internal consumers
5. Message queue integrations SHOULD implement retry logic with exponential backoff for transient external service failures
6. Components integrating with external APIs (browser detection, test coverage, system monitoring) MUST implement asynchronous message handling patterns
7. Message queue implementations MUST provide failure isolation such that external service failures do not cascade to internal application components
8. External API integrations MUST use message queue boundaries for communication rather than direct synchronous calls

---

## Adopt Message Queue Pattern for Asynchronous Component Communication

1. Components MAY implement priority queues when message ordering by importance is required
2. Dead letter queues SHOULD be implemented for messages that fail processing after retry attempts
3. Queue implementations MUST handle backpressure gracefully to prevent memory exhaustion under high load
4. Messages SHOULD include correlation IDs and timestamps to enable tracing and debugging across component boundaries
5. Message queues SHOULD implement at-least-once delivery semantics with idempotent consumers to ensure reliability
6. Message producers MUST NOT block waiting for consumer acknowledgment unless explicitly required by business logic
7. Components communicating across architectural boundaries MUST use message queue patterns rather than direct synchronous method calls

---

## Standardize External Client Libraries for Third-Party Service Integration

1. Client libraries MAY implement caching or connection pooling when appropriate for the service characteristics
2. Client libraries SHOULD provide configuration options for connection parameters, timeouts, and service endpoints
3. External client modules SHOULD be placed in dedicated directories or packages that clearly identify them as external integration points
4. Client libraries SHOULD implement consistent error handling patterns including retry logic and timeout management
5. Connection management and authentication logic MUST be centralized within the client library implementation
6. Client libraries MUST provide typed interfaces that abstract the underlying service API details from consuming code
7. External service integrations MUST be encapsulated in dedicated client library modules with clear boundaries

---

## Standardize Cache Layer Implementation for Module Resolution and State Management

1. Modules MAY implement custom cache eviction policies (LRU, LFU, TTL-based) based on specific use case requirements
2. Cache implementations SHOULD include metrics or logging to monitor cache hit rates and effectiveness
3. Cache layers SHOULD be implemented for operations that are computationally expensive, involve I/O operations, or produce deterministic results
4. Modules implementing cache layers SHOULD document cache behavior, including TTL policies, eviction strategies, and memory constraints
5. Cache implementations MUST provide explicit cache invalidation mechanisms to prevent stale data issues
6. Cache keys MUST be deterministic and based on input parameters to ensure consistent cache hits for identical inputs
7. All cache layer implementations within libraries and modules MUST use a consistent caching interface or abstraction layer

---

## Adopt Browser-Based End-to-End Testing for CI/CD Pipelines

1. Teams MAY use browser automation frameworks such as Playwright, Puppeteer, or Selenium based on project requirements
2. E2E test failures MUST block deployment in CI/CD pipelines
3. E2E tests SHOULD be organized by functional area (e.g., act.test.ts for interactions, cookie-injection.test.ts for cookie handling)
4. Visual regression testing through snapshot comparisons SHOULD be implemented for UI-critical components
5. Cookie handling and injection mechanisms MUST be validated through dedicated E2E test suites
6. E2E tests MUST cover critical user interaction patterns including DOM manipulation, event handling, and state changes
7. Browser packages MUST include end-to-end tests that execute in real or headless browser environments as part of the CI/CD pipeline

---

## Establish Service Definition Boundaries for Browser Automation Modules

1. Modules MAY use dependency injection patterns to allow runtime selection of platform-specific implementations
2. Platform-specific client implementations MUST NOT be directly imported by cross-platform utility modules
3. Utility modules SHOULD be stateless and composable to facilitate reuse across different automation contexts
4. Service modules SHOULD expose minimal public APIs that hide internal implementation details
5. DOM element resolution logic MUST be encapsulated in separate utility modules with well-defined input/output contracts
6. Runtime evaluation utilities MUST be isolated in dedicated modules that do not depend on specific WebDriver client implementations
7. Browser automation modules MUST separate platform-specific client implementations from cross-platform utility functions

---

## Adopt Browser-Based Rendering with rrweb for E2E Testing and Demo Recording

1. Implementations MAY extend rrweb with custom event types or plugins to capture application-specific interactions
2. Browser rendering modules SHOULD provide utilities for resolving locators across different selector strategies (CSS, XPath, text content)
3. Demo recording implementations SHOULD separate recording logic from replay logic to enable independent testing and optimization
4. Recording scripts MUST capture sufficient DOM state and mutation information to enable faithful replay of user sessions
5. Browser rendering utilities MUST provide locator resolution capabilities to accurately identify and interact with DOM elements during replay
6. E2E testing and demo recording implementations MUST use browser-based rendering with rrweb or equivalent DOM replay technology for capturing user interactions

---

## Adopt Browser-Based Rendering Model with Session Replay Capabilities

1. Implementations MAY optimize replay performance by pre-processing recorded events or using virtual DOM techniques
2. The rendering model SHOULD support video output generation from recorded sessions for documentation and demonstration purposes
3. Recording and replay functionality SHOULD be implemented as separate, composable modules to support different use cases (debugging, demos, testing)
4. Locator resolution utilities MUST provide consistent element identification across live and replay contexts
5. Session replay components MUST capture sufficient state information to reconstruct user interactions with high fidelity
6. All frontend rendering MUST be performed in a browser environment with full DOM access to enable accurate event capture and replay

---

## Adopt Proxy Server Pattern for External Service Integration

1. Proxy servers MAY implement request transformation or protocol translation when integrating heterogeneous systems
2. Browser-based integrations MUST NOT establish direct connections to internal services without proxy mediation
3. Proxy server implementations SHOULD be reusable across different integration contexts (CLI, browser, testing)
4. Proxy servers SHOULD implement rate limiting and throttling to protect backend services from abuse
5. Proxy implementations MUST provide request/response logging capabilities for debugging and audit purposes
6. Proxy servers MUST implement input validation for all incoming requests before forwarding to backend services
7. All external service integrations MUST use a proxy server pattern to mediate communication between clients and backend services

---

## Adopt Event-Driven Integration Pattern for Proxy and CLI Components

1. Integration components MAY implement event replay or buffering mechanisms when reliability requirements demand guaranteed delivery
2. Components SHOULD log event emissions and receptions at appropriate log levels for debugging and observability
3. Event-driven integrations SHOULD implement timeout mechanisms to handle scenarios where expected events do not arrive
4. Integration components SHOULD use event namespacing or prefixing to avoid event name collisions in shared event buses
5. Event handlers MUST implement error handling and MUST NOT allow uncaught exceptions to propagate across integration boundaries
6. Event emitters and listeners MUST be properly typed with explicit event schemas to ensure type safety across integration points
7. Integration components MUST use event-driven communication patterns when crossing architectural boundaries (process, network, or module boundaries)

---

## Standardize Browser Runtime and Cookie Management as Public API Surface

1. Packages MAY provide advanced configuration options for power users while maintaining sensible defaults for common use cases
2. Public API packages MUST NOT expose internal protocol-specific types or implementation classes that could create tight coupling with specific browser engines
3. Configuration objects for browser features SHOULD use TypeScript interfaces to provide compile-time type safety for API consumers
4. Browser-specific implementation details (CDP clients, Chromium-specific logic) SHOULD be encapsulated in separate modules and not directly exposed in public APIs
5. Public API modules MUST maintain semantic versioning and document breaking changes when modifying exported interfaces
6. Cookie management APIs MUST provide browser-agnostic configuration interfaces that abstract underlying protocol implementations (CDP, native APIs)
7. Browser runtime packages MUST export a clearly defined public API surface through dedicated index.ts entry points that serve as the contract for external consumers

---

## Implement Cache Layer for Browser Runtime and Cookie Management APIs

1. Cache implementations MAY use in-memory storage, persistent storage, or hybrid approaches based on data volatility and performance requirements
2. Cache implementations MUST NOT cache sensitive data (authentication tokens, user credentials, private browsing data) without explicit encryption
3. Cache entries SHOULD include TTL (time-to-live) metadata to prevent serving stale data in long-running browser sessions
4. Cache keys SHOULD be derived from stable identifiers (browser instance ID, configuration hash, session ID) to ensure cache correctness across concurrent operations
5. Cache implementations MUST provide cache invalidation mechanisms to ensure data consistency when underlying browser state changes
6. Cookie management APIs MUST cache CDP client connections, browser configuration, and cookie store metadata to minimize protocol overhead
7. Browser runtime APIs MUST implement a cache layer for frequently accessed browser state, configuration data, and metadata that changes infrequently

---

## Standardize Structured Logging with Public API Contracts

1. Logger contracts MAY provide specialized methods for specific use cases (e.g., audit logging, performance metrics) as long as they maintain interface consistency
2. Logger implementations MUST NOT directly couple to specific logging backends (e.g., console, file, external services) in their public API contracts
3. Logging APIs SHOULD support contextual metadata (request IDs, user IDs, trace IDs) to enable correlation across distributed operations
4. Logger implementations SHOULD provide factory functions or dependency injection patterns to enable testability and runtime configuration
5. Public logging contracts MUST define log levels consistently across all implementations (DEBUG, INFO, WARN, ERROR, FATAL)
6. Logger instances MUST support structured logging with context objects rather than string concatenation to enable machine-readable log analysis
7. All logging implementations MUST expose a public API contract with standardized method signatures (e.g., debug, info, warn, error) that can be consumed by other modules

---

## Standardize Environment Variable Validation and Path Resolution for Runtime Configuration

1. Configuration modules MAY cache resolved and validated configuration values to avoid repeated validation overhead
2. Configuration code MUST NOT trust user-provided paths without validation and sanitization
3. Default values for optional configuration parameters SHOULD be explicitly documented and consistently applied across the codebase
4. Platform-specific path resolution (e.g., HOME directory, application data directories) SHOULD use dedicated utility functions rather than inline string concatenation
5. Configuration resolution logic MUST provide clear error messages indicating which environment variable or configuration parameter is missing or invalid
6. File system paths derived from environment variables or user input MUST be normalized and validated to prevent path traversal attacks
7. All environment variables used for configuration MUST be validated before use, checking for null/undefined values and appropriate data types

---

## Standardize Test File Organization in Dedicated Tests Directories

1. Test files MUST NOT be co-located with source files in the same directory unless explicitly justified and documented
2. Test utilities and fixtures SHOULD be organized in subdirectories within the tests directory (e.g., 'tests/fixtures', 'tests/helpers')
3. CI/CD pipeline configurations MUST reference the standardized 'tests' directory pattern for test execution
4. Test directories SHOULD mirror the structure of the source code they test for maintainability
5. Test file names MUST follow the pattern '*.test.ts' or '*.test.js' to enable consistent test discovery
6. All test files MUST be placed in a dedicated 'tests' directory at the package or application root level

---

## Standardize on TypeScript Test Files with .test.ts Extension

1. Placeholder test files (e.g., placeholder.test.ts) MAY be used temporarily during initial project setup
2. Each package or application SHOULD maintain its own tests/ directory for test isolation
3. Test files MUST NOT use alternative extensions such as .spec.ts, .test.js, or _test.ts
4. Test file names SHOULD reflect the module or feature being tested (e.g., agent.test.ts for agent functionality)
5. Test files MUST be placed in a tests/ directory at the appropriate package or application level
6. All TypeScript test files MUST use the .test.ts file extension

---

## Standardize Utility Module Exports with Single-Purpose Functions

1. Utility modules MAY export TypeScript types and interfaces alongside functions when they form part of the public API contract
2. Library modules providing client interfaces (e.g., query-client.ts, webdriver-client.ts) SHOULD export factory functions or class constructors with clear initialization patterns
3. Utility modules MUST NOT contain side effects at module initialization; all effects must be contained within exported functions
4. Each utility module SHOULD have minimal external dependencies to maximize reusability and minimize coupling
5. Utility modules SHOULD be organized in a dedicated utils/ or similar directory structure to maintain clear separation from business logic
6. Public API exports MUST be explicitly defined using named exports; default exports SHOULD be avoided for utility functions
7. Module file names MUST clearly describe their single responsibility using kebab-case (e.g., format-elapsed-time.ts, copy-to-clipboard.ts)
8. Utility modules MUST export a single primary function or a cohesive set of related functions serving one specific purpose

---

## Standardize Core Utility Libraries for CLI and Browser Packages

1. Application-specific utilities MAY remain in the application's utils/ directory if they are not needed by other packages
2. Utility modules MUST NOT contain business logic specific to individual features; they must remain generic and reusable
3. Utility modules SHOULD export pure functions without side effects where possible to improve testability
4. State management utilities SHOULD be organized in a separate stores/ directory when using state management patterns
5. Each utility module MUST have a single, well-defined responsibility (e.g., formatting, clipboard operations, image processing)
6. Shared utilities used across multiple packages MUST be extracted into the packages/ directory as reusable modules
7. All utility functions MUST be organized into dedicated modules within a utils/ directory structure

---

## Standardize Public API Error Handling and Event Contracts

1. Packages MAY provide utility functions for extracting or transforming contract artifacts (e.g., extract-close-artifacts.ts) as long as the core contracts remain stable
2. Public API contract definitions MUST NOT contain implementation logic or business rules; they MUST only define the interface contract
3. Contract definition files SHOULD be co-located with their respective package modules (e.g., packages/browser/src/*/errors.ts) to maintain clear ownership
4. Public API contracts SHOULD be versioned and changes to error codes, constants, or event schemas SHOULD follow semantic versioning principles
5. Error definitions MUST include machine-readable error codes or types that remain stable across versions to enable programmatic error handling by consumers
6. Event schemas for public APIs MUST be defined in dedicated event definition files (e.g., viewer-events.ts) with explicit type definitions for all event payloads
7. Public API constants that form part of the external contract MUST be defined in dedicated constants files (e.g., constants.ts) separate from implementation logic
8. All public-facing API modules MUST define their error contracts in dedicated error definition files (e.g., errors.ts) that export typed error classes or error code constants

---

## Standardize Custom Error Classes for External API Error Handling

1. Custom error classes MAY implement additional methods for error serialization, logging, or transformation to support observability requirements
2. Public API functions MUST throw custom error instances rather than generic Error objects or string literals
3. Error modules SHOULD be co-located with the API implementation in the same package to maintain cohesion
4. Custom error classes SHOULD include additional context properties beyond the standard message field (e.g., error codes, validation details, HTTP status codes)
5. Custom error classes MUST set the error name property to a descriptive, unique identifier that clearly indicates the error type
6. Custom error classes MUST be exported from dedicated error modules (e.g., errors.ts) to ensure they are accessible to API consumers
7. All public/external API modules MUST define custom error classes that extend the base Error class for domain-specific error conditions