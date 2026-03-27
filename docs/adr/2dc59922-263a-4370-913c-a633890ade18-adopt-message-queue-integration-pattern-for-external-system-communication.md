# Adopt Message Queue Integration Pattern for External System Communication

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The system requires integration with external browser detection services, test coverage tools, and system monitoring utilities that operate as independent processes
- Multiple components (browser-detector, test-coverage, use-installed-browsers, use-listening-ports) exhibit a consistent pattern of message queue-based communication with external APIs
- The facet 'boundaries.message_queues' indicates a deliberate architectural boundary using asynchronous message passing for external system integration
- This pattern enables loose coupling between internal application logic and external service dependencies, allowing for independent scaling and failure isolation
- The pattern appears across 4 distinct files with 90% confidence, suggesting an established architectural standard rather than ad-hoc implementation

## Problem Statement

How should the system communicate with external APIs and services (browser detection, test coverage analysis, system monitoring) in a way that maintains loose coupling, enables independent scaling, provides failure isolation, and supports asynchronous processing without blocking critical application flows?

## Decision

1. SHOULD: Message queue implementations SHOULD provide observability through logging, metrics, and tracing for external API interactions

## Policy Block

- MUST External API integrations MUST use message queue boundaries for communication rather than direct synchronous calls
- MUST Message queue implementations MUST provide failure isolation such that external service failures do not cascade to internal application components
- MUST Components integrating with external APIs (browser detection, test coverage, system monitoring) MUST implement asynchronous message handling patterns
- SHOULD Message queue integrations SHOULD implement retry logic with exponential backoff for transient external service failures
- SHOULD External API responses SHOULD be validated and sanitized before being passed to internal consumers
- SHOULD Message queue implementations SHOULD provide observability through logging, metrics, and tracing for external API interactions
- MAY Teams MAY implement circuit breaker patterns to prevent overwhelming failing external services
- MAY Message queue consumers MAY implement caching strategies for frequently accessed external API data to reduce latency and external service load

In scope:
- Browser detection service integrations
- Test coverage analysis tool integrations
- System monitoring utilities (installed browsers, listening ports)
- Any external API that provides non-critical, eventually-consistent data
- Third-party service integrations where response time is not critical to user-facing operations

Out of scope:
- Internal service-to-service communication within the same deployment boundary
- Real-time user authentication and authorization flows requiring immediate responses
- Critical path operations where synchronous responses are required for correctness
- Database queries and internal data access patterns

Exceptions:
- EX-001: External API provides sub-100ms SLA guarantees and is critical to user-facing request path
- EX-002: Proof-of-concept or prototype code in non-production environments

## Rationale

- Pattern detected across 4 files with 90% confidence indicates this is an established architectural standard that has proven effective in production
- Message queue boundaries provide natural failure isolation, preventing external service outages from cascading into core application functionality
- Asynchronous communication enables independent scaling of external API consumers without impacting application throughput
- The facet 'boundaries.message_queues' explicitly identifies this as an architectural boundary pattern, suggesting intentional design rather than emergent behavior

## Consequences

Positive:
- Loose coupling between internal application logic and external services enables independent evolution and deployment
- Failure isolation prevents external service outages from causing application-wide failures
- Asynchronous processing improves application responsiveness by not blocking on external API calls
- Message queues provide natural buffering and load leveling for external API requests
- Pattern consistency across multiple components reduces cognitive load and improves maintainability

Negative:
- Increased system complexity due to additional message queue infrastructure and asynchronous processing logic
- Eventually-consistent data model may require additional handling for scenarios requiring immediate external API responses
- Debugging and tracing becomes more complex with asynchronous message flows across system boundaries
- Additional operational overhead for monitoring, maintaining, and scaling message queue infrastructure

## Alternatives

- Direct synchronous HTTP calls to external APIs from application code (rejected)
  Rejected because: Creates tight coupling, lacks failure isolation, blocks application threads, and makes external service failures cascade into application failures
  When valid: Only valid for critical-path operations with sub-100ms SLA guarantees and explicit architecture approval
- Service mesh with circuit breakers for external API calls (rejected)
  Rejected because: While providing failure isolation, still maintains synchronous communication model and blocks application threads during external API calls
  When valid: Valid for services requiring immediate responses where message queue latency is unacceptable
- Event-driven architecture with event bus instead of message queues (deferred)
  Rejected because: Not rejected, but deferred for future consideration as it provides similar benefits with different trade-offs
  When valid: Valid when pub-sub patterns are needed or when multiple consumers need to react to external API events

## Risks

- Message queue infrastructure becomes a single point of failure if not properly configured with high availability
  Mitigation: Implement message queue clustering, persistence, and failover mechanisms. Monitor queue health and set up alerting for queue availability issues.
  Owner: Platform Engineering Team
- Message queue backlog growth during external service outages could lead to memory exhaustion or message loss
  Mitigation: Implement queue depth monitoring, dead letter queues for failed messages, and backpressure mechanisms. Set maximum queue sizes and TTL for messages.
  Owner: Engineering Team
- Inconsistent implementation of message queue patterns across teams could lead to integration issues and maintenance burden
  Mitigation: Provide shared libraries and templates for message queue integration. Conduct code reviews to ensure pattern compliance. Document reference implementations.
  Owner: Architecture Team

## Implementation Notes

- Use established message queue libraries and frameworks rather than implementing custom solutions (e.g., RabbitMQ, Redis Streams, AWS SQS)
- Implement idempotent message handlers to safely handle duplicate message delivery scenarios
- Configure appropriate message TTL, retry limits, and dead letter queue policies for each external API integration
- Include correlation IDs in messages to enable distributed tracing across message queue boundaries
- Document message schemas and contracts for each external API integration to facilitate testing and maintenance

## Continuation Context


Verify commands:
- grep -r 'boundaries.message_queues' --include='*.ts' --include='*.js' | wc -l
- grep -rE '(queue|publish|subscribe|consumer|producer)' packages/cookies/src/browser-detector.ts packages/supervisor/src/test-coverage.ts apps/cli/src/hooks/ 2>/dev/null || echo 'Files not found in expected locations'
- find . -name '*.ts' -o -name '*.js' | xargs grep -l 'external.*api' | xargs grep -l 'queue\|message\|async' | wc -l

Accept when:
- All external API integrations for browser detection, test coverage, and system monitoring use message queue boundaries
- Code review confirms no direct synchronous calls to external APIs in the identified components (browser-detector, test-coverage, use-installed-browsers, use-listening-ports)
- Message queue infrastructure is configured with monitoring, alerting, and failure handling mechanisms
- Documentation exists for message schemas and integration patterns for each external API

## Enforcement

- Verified by: Automated code review checks scanning for direct external API calls without message queue boundaries
- Verified by: Architecture review for new external API integrations
- Verified by: CI/CD pipeline checks validating message queue configuration and error handling
- Verified by: Periodic architecture audits reviewing external integration patterns
- Violation handling: CI/CD pipeline fails if direct synchronous external API calls are detected without approved exceptions
- Violation handling: Code review process blocks PRs that violate message queue integration patterns
- Violation handling: Architecture team conducts remediation planning for existing violations
- Violation handling: Violations are tracked in technical debt backlog with prioritization based on risk
- Exception process: Submit exception request to architecture review board with justification, SLA guarantees, and fallback strategy
- Exception process: Document exception in ADR exceptions registry with approval date and reviewer
- Exception process: Include monitoring and alerting requirements for approved exceptions
- Exception process: Schedule periodic review of exceptions to evaluate if message queue migration is feasible