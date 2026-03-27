<rule_activation id="7eb8d13a-1afe-4a68-8dcb-599fcb45fe2f" title="Standardize Cache Layer Implementation for Module Resolution and State Management" applies_to="**/*">
These rules are ALWAYS ACTIVE for all files. All cache layer implementations within libraries and modules must use a consistent caching interface or abstraction layer to ensure predictable behavior, maintainability, and optimal resource utilization.
</rule_activation>

### Rules

- **R-CACHE-001** MUST: All cache layer implementations within libraries and modules MUST use a consistent caching interface or abstraction layer.
- **R-CACHE-002** MUST: Cache keys MUST be deterministic and based on input parameters to ensure consistent cache hits for identical inputs.
- **R-CACHE-003** MUST: Cache implementations MUST provide explicit cache invalidation mechanisms to prevent stale data issues.
- **R-CACHE-004** SHOULD: Modules implementing cache layers SHOULD document cache behavior, including TTL policies, eviction strategies, and memory constraints.
- **R-CACHE-005** SHOULD: Cache layers SHOULD be implemented for operations that are computationally expensive, involve I/O operations, or produce deterministic results.
- **R-CACHE-006** SHOULD: Cache implementations SHOULD include metrics or logging to monitor cache hit rates and effectiveness.
- **R-CACHE-007** MAY: Modules MAY implement custom cache eviction policies (LRU, LFU, TTL-based) based on specific use case requirements.

#### Scope

**In scope:**
- All library and module code that implements caching mechanisms for data, computation results, or API responses
- Utility functions that perform expensive operations and benefit from memoization
- Browser automation modules that cache WebDriver client state or session information
- CLI command processors that cache audit results or intermediate computation states

**Out of scope:**
- Application-level caching strategies (e.g., Redis, CDN caching)
- Database query result caching managed by ORM layers
- HTTP response caching handled by web frameworks or middleware
- Browser-native caching mechanisms (localStorage, sessionStorage, IndexedDB)

**Exceptions:**
- EXC-001: Performance-critical hot paths require inline caching without abstraction overhead
- EXC-002: Third-party libraries impose their own caching mechanisms that cannot be adapted

### Verify

```bash
# Count cache implementations using Map/WeakMap/Set
grep -r 'cache' packages/browser/src apps/cli/src --include='*.ts' --include='*.tsx' | grep -E '(Map|WeakMap|Set|cache)' | wc -l

# Find files defining Cache classes/interfaces/types
find . -name '*.ts' -o -name '*.tsx' | xargs grep -l 'class.*Cache\|interface.*Cache\|type.*Cache' | wc -l

# Run cache-related tests
npm run test -- --grep 'cache' --reporter json | jq '.stats.passes'
```

**Accept when:**
- All cache implementations in the identified modules (webdriver-client, resolve-nth-duplicates, audit) use the standardized caching interface
- Cache-related test coverage is at least 80% for all modules implementing cache layers
- No direct usage of Map, WeakMap, or object literals for caching purposes outside the standardized abstraction (exceptions documented)
- Cache behavior documentation exists for each module implementing caching, including invalidation strategies

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated linting rules detect non-standard cache implementations during CI builds. Code review checklist includes verification of cache layer compliance. CI build fails if non-standard cache implementations are detected without documented exceptions.
</enforcement>