# Adopt Message Queue Pattern for Asynchronous Component Communication

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all integration components that require asynchronous communication between distributed services or modules.

## Context

- The system architecture includes multiple distributed components (browser packages, supervisor services, MCP sessions) that need to communicate asynchronously without tight coupling
- Real-time updates and live view functionality require a mechanism to propagate state changes across component boundaries without blocking operations
- The detected pattern (signature a6f195c041c1fbe8bf1d99d36e62263b) appears consistently across 3 files with high confidence (91.50%), indicating an established architectural practice
- Message queues provide decoupling between producers and consumers, enabling scalability and fault tolerance in integration scenarios
- The facet 'boundaries.message_queues' suggests this pattern is specifically used for managing communication across architectural boundaries

## Problem Statement

How should components communicate asynchronously across architectural boundaries in a distributed system where tight coupling would reduce scalability, maintainability, and fault tolerance? Direct synchronous calls create dependencies that make systems brittle, while ad-hoc event handling leads to inconsistent integration patterns.

## Decision

1. SHOULD: Messages SHOULD include correlation IDs and timestamps to enable tracing and debugging across component boundaries

## Policy Block

- MUST Components communicating across architectural boundaries MUST use message queue patterns rather than direct synchronous method calls
- MUST Message producers MUST NOT block waiting for consumer acknowledgment unless explicitly required by business logic
- SHOULD Message queues SHOULD implement at-least-once delivery semantics with idempotent consumers to ensure reliability
- SHOULD Messages SHOULD include correlation IDs and timestamps to enable tracing and debugging across component boundaries
- MUST Queue implementations MUST handle backpressure gracefully to prevent memory exhaustion under high load
- SHOULD Dead letter queues SHOULD be implemented for messages that fail processing after retry attempts
- MAY Components MAY implement priority queues when message ordering by importance is required

In scope:
- Browser-to-supervisor communication in MCP sessions
- Live view server update propagation
- Cross-package asynchronous event distribution
- Integration points between independently deployable services
- Real-time update streams and notification systems

Out of scope:
- Synchronous request-response APIs within a single service boundary
- Direct function calls within the same module or package
- Database transaction coordination (use transaction patterns instead)
- Simple event listeners within a single component lifecycle

Exceptions:
- EXC-001: Performance profiling demonstrates that message queue overhead exceeds 20% of total operation time for high-frequency, low-latency operations
- EXC-002: Legacy integration with third-party systems that only support synchronous communication patterns

## Rationale

- Pattern detected with 91.50% confidence across 3 critical integration files (live-view-server, mcp-session, updates), indicating this is an established and validated architectural practice
- Message queues provide natural decoupling between components, allowing independent scaling, deployment, and failure isolation
- The facet 'boundaries.message_queues' explicitly identifies this pattern as a boundary management strategy, essential for maintaining clean architecture in distributed systems
- Asynchronous communication prevents cascading failures and improves system resilience by allowing components to continue operating even when downstream consumers are temporarily unavailable

## Consequences

Positive:
- Improved system scalability through decoupled components that can be scaled independently based on load
- Enhanced fault tolerance as message queues buffer requests during temporary component failures
- Better separation of concerns with clear integration boundaries defined by message contracts
- Simplified testing through the ability to mock message producers and consumers independently

Negative:
- Increased system complexity with additional infrastructure components (queue managers, brokers) to maintain
- Debugging becomes more challenging due to asynchronous execution and distributed tracing requirements
- Potential for message ordering issues and eventual consistency challenges
- Additional operational overhead for monitoring queue depths, processing rates, and dead letter queues

## Alternatives

- Direct synchronous HTTP/RPC calls between components (rejected)
  Rejected because: Creates tight coupling, reduces fault tolerance, and causes cascading failures when downstream services are unavailable. Does not support the asynchronous update patterns required by live-view and MCP session components.
  When valid: Only appropriate for simple request-response patterns within a single service boundary where latency is critical and coupling is acceptable
- Shared database with polling for state changes (rejected)
  Rejected because: Introduces database as a bottleneck, creates inefficient polling overhead, and violates service autonomy principles. Does not provide real-time notification capabilities.
  When valid: May be acceptable for batch processing scenarios with low update frequency and existing shared database infrastructure
- Event-driven architecture with pub-sub messaging (accepted)
  When valid: This is the adopted pattern. Message queues implement the underlying mechanism for event-driven pub-sub communication across component boundaries.

## Risks

- Message queue infrastructure failure could cause complete system communication breakdown
  Mitigation: Implement redundant queue instances with automatic failover, circuit breakers for graceful degradation, and local buffering for critical messages
  Owner: Platform Engineering Team
- Message processing delays or queue buildup could lead to stale data and user experience degradation
  Mitigation: Implement comprehensive monitoring with alerting on queue depth thresholds, processing lag metrics, and automatic scaling of consumer instances
  Owner: Engineering Team
- Inconsistent message schemas across components could cause integration failures
  Mitigation: Establish schema registry with versioning, implement contract testing between producers and consumers, and enforce backward compatibility requirements
  Owner: Architecture Team

## Implementation Notes

- Start by identifying all cross-boundary communication points in live-view-server.ts, mcp-session.ts, and updates.ts as reference implementations
- Define clear message schemas with versioning strategy before implementing queue infrastructure
- Implement comprehensive logging with correlation IDs to enable distributed tracing across message flows
- Consider using established message queue libraries or services (e.g., RabbitMQ, Redis Streams, AWS SQS) rather than building custom solutions
- Establish monitoring dashboards for queue metrics (depth, throughput, latency, error rates) before deploying to production

## Continuation Context


Verify commands:
- grep -r "queue\|Queue\|EventEmitter\|Subject\|Observable" packages/browser/src/mcp/ packages/supervisor/src/ --include="*.ts" | grep -v node_modules
- grep -r "async.*emit\|publish\|enqueue\|send.*message" packages/ --include="*.ts" | wc -l
- find packages/ -name "*queue*.ts" -o -name "*message*.ts" -o -name "*event*.ts" | grep -v node_modules | head -20

Accept when:
- All integration points between browser packages and supervisor services use message queue patterns (no direct synchronous calls across package boundaries)
- Message queue implementations include error handling, retry logic, and dead letter queue support
- Monitoring and observability tools can trace message flows across component boundaries using correlation IDs
- Code review checklist includes verification that new cross-boundary communication uses approved message queue patterns

## Enforcement

- Verified by: Automated static analysis tools scan for direct cross-boundary synchronous calls during CI pipeline
- Verified by: Architecture review required for all new integration points between services or packages
- Verified by: Code review checklist includes verification of message queue pattern compliance
- Verified by: Quarterly architecture audits review integration patterns against this ADR
- Violation handling: CI pipeline fails if static analysis detects direct synchronous calls across defined architectural boundaries
- Violation handling: Code review must document justification for any deviations and obtain architecture team approval
- Violation handling: Existing violations identified in audits are tracked as technical debt with remediation timeline
- Violation handling: Critical violations in production code trigger immediate remediation planning
- Exception process: Submit exception request to architecture review board with performance data or technical justification
- Exception process: Document alternative approach and demonstrate it maintains decoupling principles
- Exception process: Obtain written approval from technical lead and architecture team
- Exception process: Add exception documentation to ADR tracking system with review date for re-evaluation