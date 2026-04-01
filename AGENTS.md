---

## Architecture Decision Records

<adr_governance source="docs/adr/">
ADRs govern validated architectural standards for this project.
Full ADR documents: @docs/adr/
</adr_governance>

<activation>
These directives are ALWAYS ACTIVE. All AI coding agents MUST apply all rules in this
document to every code generation, modification, and review action within this
project. No exceptions unless explicitly noted per-rule.
</activation>

---

### Verification Protocol

<verification_protocol>
All rules in this document follow the **Verify → Fix → Repeat** loop.
</verification_protocol>

After generating or modifying code for any rule, the agent MUST:

1. **RUN** the targeted verification command(s) in the rule's **Verify** block.
2. **CAPTURE** the full command output (stdout + stderr).
3. **EVALUATE** whether the **Accept when** criteria are satisfied.
4. **IF FAILING:** diagnose the root cause, apply a fix, and re-run from step 1.
5. **IF PASSING:** include the passing output as inline evidence before proposing further changes.
6. **MAX ITERATIONS:** 5 attempts per rule. If still failing after 5 attempts, STOP and report the failure with all captured outputs.

<enforcement>
Compliance is not optional. Agents must not skip verification steps, assume
correctness, or defer verification to a later task. Evidence of a passing
verification run must accompany every code change that touches a governed area.
</enforcement>

---

## External Http Clients

External HTTP clients MUST implement proper timeout configuration to prevent indefinite blocking on external service calls

---

## Browser Automation Code

Browser automation code SHOULD be organized into reusable modules with clear separation between browser lifecycle management, locator resolution, and test/demo logic

---

## Browser Control Utilities

Browser control utilities SHOULD provide abstraction over underlying browser automation libraries to enable library substitution without widespread code changes

---

## Element Location Strategies

Element location strategies SHOULD prefer semantic locators (ARIA roles, labels, test IDs) over brittle CSS selectors when possible

---

## Components Use Structured

All components MUST use a structured logging framework that supports contextual metadata and consistent formatting

---

## Environment Variables Read

Environment variables MUST be read and validated at application initialization time, not lazily throughout execution

---

## Locator Resolution Support

Locator resolution SHOULD support multiple locator strategies (CSS selectors, XPath, text content, etc.) to maximize element identification reliability

---

## Browser Rendering Modules

Browser rendering modules SHOULD expose locator resolution as part of their public API contracts for external consumers

---

## Screen Components Minimize

Screen components SHOULD minimize direct state manipulation and delegate complex logic to custom hooks or utility functions

---

## Logger Utilities Imported

Logger utilities MUST be imported from centralized locations (e.g., utils/logger.ts) to ensure consistent behavior across the application

---

## Proxy Server Implementations

Proxy server implementations MUST maintain protocol transparency, forwarding messages without modifying semantic content unless explicitly documented

---

## Video Replay Rendering

Video replay rendering (rrvideo) MUST use the same locator resolution mechanism as standard browser rendering to ensure consistency

---

## Components Use Additional

Components MAY use additional React hooks from the ecosystem (useMemo, useRef, useContext) when performance optimization or context access is required