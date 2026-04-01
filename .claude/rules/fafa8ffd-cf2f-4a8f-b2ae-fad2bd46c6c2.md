<rule_activation id="fafa8ffd-cf2f-4a8f-b2ae-fad2bd46c6c2" title="Standardize Structured Logging with Dedicated Logger Utilities: Logger Utilities Imported" applies_to="**/*">
These rules are ALWAYS ACTIVE for all logging implementations across the codebase. All components that emit log messages MUST follow the structured logging patterns defined herein.
</rule_activation>

### Rules

- **R-LOG-001** MUST: Logger utilities MUST be imported from centralized locations (e.g., utils/logger.ts) to ensure consistent behavior across the application.

**In scope:**
- All application code in apps/cli
- All agent code in packages/agent
- Utility modules that perform I/O or external communication
- Command implementations and test runners
- Error handling and exception logging

**Out of scope:**
- Third-party library internal logging
- Development-only debug statements that are removed before commit
- Test assertion output from testing frameworks
- Build system and tooling output

**Exceptions:**
- **EXC-001**: Direct console output is required for user-facing CLI output that is not diagnostic logging (e.g., command results, formatted reports)
- **EXC-002**: Emergency debugging in production requires temporary console.log statements

### Verify

```bash
# Check for direct console usage in application code (should find none or only user output)
grep -r "console\.log\|console\.error\|console\.warn" apps/cli/src packages/agent/src --include="*.ts" --exclude="*.test.ts" | grep -v "// user output" || echo "No direct console usage found"

# Count logger imports (should be present across components)
grep -r "import.*logger" apps/cli/src packages/agent/src --include="*.ts" | wc -l

# Verify logger utility exists
test -f apps/cli/src/utils/logger.ts && echo "Logger utility exists" || echo "Logger utility missing"
```

**Accept when:**
- Direct console.log/console.error calls in application code (excluding test files) are either absent or explicitly marked as user output with comments
- Logger utility modules exist in expected locations (e.g., apps/cli/src/utils/logger.ts) and are imported by multiple components
- At least 80% of logging statements in the codebase use the standardized logger utilities rather than direct console methods

<enforcement>
Claude Code MUST verify compliance with structured logging standards. Direct console usage in application code requires justification or correction. ESLint rules and CI pipeline checks enforce these patterns. Violations trigger code review feedback and CI warnings.
</enforcement>