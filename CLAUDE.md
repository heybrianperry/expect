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

## ADR 1: Standardize External HTTP Client Usage for Third-Party API Integration: External Http Clients

1. External HTTP clients MUST implement proper timeout configuration to prevent indefinite blocking on external service calls

---

## ADR 2: Adopt Playwright Browser Automation for E2E Testing and Demo Recording: Browser Automation Code

1. Browser automation code SHOULD be organized into reusable modules with clear separation between browser lifecycle management, locator resolution, and test/demo logic

---

## ADR 3: Adopt Browser-Based Rendering with Locator Resolution for UI Automation: Browser Control Utilities

1. Browser control utilities SHOULD provide abstraction over underlying browser automation libraries to enable library substitution without widespread code changes

---

## ADR 4: Adopt Playwright Browser Automation for E2E Testing and Demo Recording: Element Location Strategies

1. Element location strategies SHOULD prefer semantic locators (ARIA roles, labels, test IDs) over brittle CSS selectors when possible

---

## ADR 5: Standardize Structured Logging with Contextual Metadata: Components Use Structured

1. All components MUST use a structured logging framework that supports contextual metadata and consistent formatting

---

## ADR 6: Standardize Environment Variable Access Through Centralized Configuration Constants: Environment Variables Read

1. Environment variables MUST be read and validated at application initialization time, not lazily throughout execution

---

## ADR 7: Adopt Browser-Based Rendering with Locator Resolution for UI Automation: Locator Resolution Support

1. Locator resolution SHOULD support multiple locator strategies (CSS selectors, XPath, text content, etc.) to maximize element identification reliability

---

## ADR 8: Adopt Browser-Based Rendering with Locator Resolution for UI Components: Browser Rendering Modules

1. Browser rendering modules SHOULD expose locator resolution as part of their public API contracts for external consumers

---

## ADR 9: Standardize React Hooks for UI State Management in CLI Components: Screen Components Minimize

1. Screen components SHOULD minimize direct state manipulation and delegate complex logic to custom hooks or utility functions

---

## ADR 10: Standardize Structured Logging with Dedicated Logger Utilities: Logger Utilities Imported

1. Logger utilities MUST be imported from centralized locations (e.g., utils/logger.ts) to ensure consistent behavior across the application

---

## ADR 11: Standardize MCP Server Implementation Pattern for Service Integration: Proxy Server Implementations

1. Proxy server implementations MUST maintain protocol transparency, forwarding messages without modifying semantic content unless explicitly documented

---

## ADR 12: Adopt Browser-Based Rendering with Locator Resolution for UI Components: Video Replay Rendering

1. Video replay rendering (rrvideo) MUST use the same locator resolution mechanism as standard browser rendering to ensure consistency

---

## ADR 13: Standardize React Hooks for UI State Management in CLI Components: Components Use Additional

1. Components MAY use additional React hooks from the ecosystem (useMemo, useRef, useContext) when performance optimization or context access is required