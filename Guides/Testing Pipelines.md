---
description: How to test Acme pipelines locally using validation tests, fixture-based tests, and snapshot tests before deploying to production.
---

# Testing Pipelines

Every Acme pipeline can be tested locally before deployment. The `acme test` command validates your configuration, checks connections, and runs your pipeline against fixture data.

## Why test?

> [!quote]
> "If it's not tested, it's broken." — Every data engineer, eventually.

Testing prevents:

- Schema mismatches that silently drop columns
- Transform logic errors that corrupt data
- Connection failures that only surface in production

## Test types

### 1. Validation tests

Check that your YAML is valid and all referenced transforms exist:

```bash
acme test --validate pipelines/user-analytics.yml
```

```
✓ YAML syntax valid
✓ Source 'users_db' configuration complete
✓ Transform 'filter' has valid condition
✓ Transform 'python/hash_email' function found
✓ Destination 'bigquery/active_users' configuration complete
✓ All 5 checks passed
```

### 2. Fixture tests

Run your pipeline against local test data:

```yaml
# tests/test_user_analytics.yml
pipeline: pipelines/user-analytics.yml

fixtures:
  users_db: tests/fixtures/sample_users.csv

assertions:
  - row_count: { min: 1, max: 100 }
  - columns:
      [id, full_name, email_hash, account_age_days, is_recent, created_at]
  - no_nulls: [id, full_name, email_hash]
  - unique: [id]
```

Create the fixture file:

```csv
id,email,first_name,last_name,status,created_at,last_login_at
1,alice@example.com,Alice,Johnson,active,2025-01-15,2026-02-10
2,bob@example.com,Bob,Smith,inactive,2024-06-01,2025-03-01
3,carol@example.com,Carol,White,active,2025-11-20,2026-02-14
```

Run the test:

```bash
acme test tests/test_user_analytics.yml
```

```
✓ Pipeline loaded: user-analytics
✓ Fixture loaded: sample_users.csv (3 rows)
✓ Source: users_db (3 rows extracted)
✓ Transform: filter → 2 rows (filtered 1 inactive)
✓ Transform: python/hash_email → 2 rows
✓ Transform: map → 2 rows (3 fields computed)
✓ Transform: select → 2 rows (6 columns)

Assertions:
  ✓ row_count: 2 (within 1-100)
  ✓ columns: all 6 present
  ✓ no_nulls: no null values found
  ✓ unique: id is unique

All tests passed!
```

### 3. Snapshot tests

Compare pipeline output against a known-good snapshot:

```bash
# Generate a snapshot
acme test tests/test_user_analytics.yml --update-snapshot

# Run against the snapshot
acme test tests/test_user_analytics.yml --snapshot
```

> [!tip] CI integration
> Add `acme test` to your CI pipeline to catch regressions before they reach production:
>
> ```yaml
> # .github/workflows/test.yml
> - name: Test pipelines
>   run: acme test tests/
> ```

## Writing good tests

1. **Test edge cases** — empty datasets, null values, special characters
2. **Test transforms individually** — isolate complex transforms with targeted fixtures
3. **Test schema evolution** — what happens when a source adds or removes a column?
4. **Keep fixtures small** — 5-20 rows is usually enough

## Related

- [[Getting Started/Your First Pipeline|Your First Pipeline]] — build a pipeline to test
- [[Guides/Deployment|Deployment]] — CI/CD integration
- [[Guides/Error Handling|Error Handling]] — handling unexpected data
