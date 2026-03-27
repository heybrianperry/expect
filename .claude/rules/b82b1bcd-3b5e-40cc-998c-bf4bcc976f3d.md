<rule_activation id="b82b1bcd-3b5e-40cc-998c-bf4bcc976f3d" title="Adopt Browser-Based Rendering with rrweb for E2E Testing and Demo Recording" applies_to="**/*">
These rules apply to E2E testing and demo recording implementations, browser-based rendering utilities, and DOM locator resolution mechanisms.
</rule_activation>

### Rules

- **R-RRWEB-001** MUST: E2E testing and demo recording implementations MUST use browser-based rendering with rrweb or equivalent DOM replay technology for capturing user interactions.
- **R-RRWEB-002** MUST: Browser rendering utilities MUST provide locator resolution capabilities to accurately identify and interact with DOM elements during replay.
- **R-RRWEB-003** MUST: Recording scripts MUST capture sufficient DOM state and mutation information to enable faithful replay of user sessions.
- **R-RRWEB-004** SHOULD: Demo recording implementations SHOULD separate recording logic from replay logic to enable independent testing and optimization.
- **R-RRWEB-005** SHOULD: Browser rendering modules SHOULD provide utilities for resolving locators across different selector strategies (CSS, XPath, text content).
- **R-RRWEB-006** MAY: Implementations MAY extend rrweb with custom event types or plugins to capture application-specific interactions.

### Verify

```bash
grep -r "rrweb" packages/browser/src/ apps/website/scripts/ --include="*.ts" --include="*.js"
grep -r "record.*demo\|rrvideo\|resolve-locator" packages/browser/src/ apps/website/scripts/ --include="*.ts"
npm list rrweb || yarn list --pattern rrweb
```

**Accept when:**
- rrweb or equivalent DOM replay library is present in project dependencies
- At least one recording script and one replay utility exist in the codebase
- Locator resolution utilities are implemented with support for multiple selector strategies
- E2E tests or demo scripts successfully use browser-based rendering for interaction capture

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>