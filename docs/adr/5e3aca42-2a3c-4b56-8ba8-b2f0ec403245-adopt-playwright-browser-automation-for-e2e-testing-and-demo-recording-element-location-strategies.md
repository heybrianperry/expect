# Adopt Playwright Browser Automation for E2E Testing and Demo Recording: Element Location Strategies

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The project requires automated browser testing capabilities to validate UI rendering and user interactions across different browsers and environments
- Demo recording and documentation generation need programmatic browser control to capture consistent, reproducible screenshots and videos of application behavior
- Browser-based testing infrastructure must support element location strategies (CSS selectors, XPath, text content) to interact with dynamically rendered UI components
- The testing.e2e facet indicates a need for end-to-end testing that exercises the full rendering pipeline from server response to client-side DOM manipulation

## Problem Statement

The project needs a reliable, maintainable approach to browser automation that supports both automated testing and documentation generation, with consistent element location strategies and cross-browser compatibility, while avoiding brittle test implementations that break with minor UI changes.

## Decision

1. SHOULD: Element location strategies SHOULD prefer semantic locators (ARIA roles, labels, test IDs) over brittle CSS selectors when possible

## Policy Block

- SHOULD Element location strategies SHOULD prefer semantic locators (ARIA roles, labels, test IDs) over brittle CSS selectors when possible

In scope:
- All end-to-end testing that requires browser rendering
- Automated demo recording and screenshot generation for documentation
- Visual regression testing that validates UI rendering
- Integration tests that verify client-side JavaScript behavior

Out of scope:
- Unit tests that do not require DOM rendering
- API integration tests that do not involve browser UI
- Server-side rendering tests that can use simpler HTML parsing
- Performance benchmarks that use specialized profiling tools

Exceptions:
- EXC-001: Legacy tests using alternative frameworks (Selenium, Puppeteer) may remain during migration period

## Rationale

- Pattern detected across 3 files with 91.23% confidence, indicating consistent adoption of browser automation infrastructure in the codebase
- Playwright provides cross-browser support, modern async/await APIs, and built-in waiting mechanisms that reduce test flakiness compared to older frameworks
- Centralized locator resolution (resolve-locator.ts) enables consistent element location strategies and easier maintenance when UI structure changes
- The presence of both testing utilities (browser.ts) and demo recording scripts (record-demo.ts) demonstrates dual-purpose infrastructure that serves both quality assurance and documentation needs

## Consequences

Positive:
- Consistent browser automation approach across testing and documentation reduces learning curve and maintenance burden
- Playwright's modern API and built-in retry mechanisms improve test reliability and reduce flakiness
- Centralized locator resolution enables easier refactoring of element location strategies without touching individual tests
- Automated demo recording ensures documentation stays synchronized with actual application behavior

Negative:
- Playwright dependency adds significant package size and installation time to development environments
- Browser automation tests are slower than unit tests and require more CI resources
- Maintaining browser automation infrastructure requires specialized expertise in Playwright APIs and debugging techniques
- Cross-browser testing multiplies test execution time and infrastructure costs

## Alternatives

- Use Puppeteer for browser automation (rejected)
  Rejected because: Puppeteer only supports Chromium-based browsers, limiting cross-browser testing capabilities. Playwright offers better API design and multi-browser support.
  When valid: Valid for projects that only need to support Chrome/Chromium and want a lighter-weight solution
- Use Selenium WebDriver for browser automation (rejected)
  Rejected because: Selenium has more verbose APIs, requires separate driver management, and has less reliable waiting mechanisms compared to Playwright's modern approach.
  When valid: Valid for organizations with existing Selenium infrastructure and expertise, or when testing very old browser versions
- Use Cypress for E2E testing (rejected)
  Rejected because: Cypress runs inside the browser and has limitations with multi-tab scenarios, iframes, and programmatic browser control needed for demo recording.
  When valid: Valid for pure E2E testing scenarios without demo recording requirements, especially when developer experience is prioritized over flexibility

## Risks

- Playwright version updates may introduce breaking API changes that require widespread test updates
  Mitigation: Pin Playwright version in package.json, test upgrades in isolated branch, maintain changelog of breaking changes
  Owner: QA Engineering Team
- Browser automation tests may become flaky due to timing issues, network conditions, or environment differences
  Mitigation: Use Playwright's built-in waiting mechanisms, implement retry logic, run tests in isolated containers with controlled network conditions
  Owner: QA Engineering Team
- Centralized locator resolution utility may become a bottleneck if not designed for extensibility
  Mitigation: Design locator resolver with plugin architecture, document extension points, regularly review and refactor based on usage patterns
  Owner: Frontend Architecture Team

## Implementation Notes

- Create a packages/browser module that exports browser lifecycle management, locator resolution, and common test utilities
- Implement resolve-locator utility with support for CSS selectors, XPath, text content, ARIA roles, and custom test ID attributes
- Establish naming conventions for test IDs (e.g., data-testid attributes) and document in testing guidelines
- Configure Playwright to run in headless mode in CI, headed mode for local debugging, with video recording on test failure

## Continuation Context


Verify commands:
- grep -r "@playwright/test" package.json packages/*/package.json apps/*/package.json
- find . -name "*browser*.ts" -o -name "*locator*.ts" | grep -E "(packages/browser|utils)"
- grep -r "import.*playwright" --include="*.ts" --include="*.js" | wc -l

Accept when:
- Playwright is declared as a dependency in relevant package.json files
- Browser automation utilities exist in a centralized location (packages/browser or similar)
- At least one demo recording script or E2E test file imports and uses Playwright APIs

## Enforcement

- Verified by: Automated dependency scanning in CI checks for Playwright presence in browser automation code
- Verified by: Code review checklist includes verification that new E2E tests use Playwright and centralized locator utilities
- Verified by: Architecture review process validates that browser automation follows established patterns
- Violation handling: Pull requests introducing alternative browser automation frameworks are flagged for architecture review
- Violation handling: Tests that bypass centralized locator resolution are marked for refactoring in technical debt backlog
- Violation handling: CI warnings are generated when browser automation code is detected outside approved packages
- Exception process: Submit exception request to architecture review board with justification for alternative approach
- Exception process: Document exception rationale in ADR amendments or inline code comments
- Exception process: Set expiration date for temporary exceptions with migration plan to standard approach