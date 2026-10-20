---
description: Guide to writing custom Python transforms in Acme, covering row transforms, batch operations, API enrichment, stateful transforms, and performance tips.
---

# Writing Custom Transforms

When built-in transforms aren't enough, Acme lets you write custom data transformations in Python. This guide covers everything from simple row transforms to batch operations and async processing.

## Basic row transform

The simplest custom transform is a function that takes a row (dict) and returns a modified row:

```python
# transforms/clean.py

def normalize_phone(row):
    """Normalize phone numbers to E.164 format."""
    phone = row.get("phone", "")
    # Remove spaces, dashes, parentheses
    digits = "".join(c for c in phone if c.isdigit())
    if len(digits) == 10:
        row["phone"] = f"+1{digits}"
    elif len(digits) == 11 and digits.startswith("1"):
        row["phone"] = f"+{digits}"
    else:
        row["phone"] = None  # Invalid
    return row
```

Use it in your pipeline:

```yaml
transforms:
  - type: python
    function: transforms.clean.normalize_phone
```

## Batch transforms

For operations that need access to multiple rows (like aggregation or lookups), use a batch transform:

```python
# transforms/dedupe.py
from acme.sdk import BatchTransform

class DeduplicateByEmail(BatchTransform):
    """Remove duplicate rows, keeping the most recent."""

    def process_batch(self, rows):
        seen = {}
        for row in rows:
            email = row.get("email")
            if email not in seen or row["updated_at"] > seen[email]["updated_at"]:
                seen[email] = row
        return list(seen.values())
```

```yaml
transforms:
  - type: python
    class: transforms.dedupe.DeduplicateByEmail
```

## Enrichment with external APIs

> [!warning] Rate limiting
> External API calls can significantly slow down your pipeline. Consider:
>
> - Caching responses locally
> - Using batch API endpoints when available
> - Setting appropriate timeouts

```python
# transforms/enrich.py
import requests
from functools import lru_cache

CLEARBIT_API = "https://company.clearbit.com/v2/companies/find"

@lru_cache(maxsize=10000)
def _lookup_company(domain):
    """Cache company lookups to avoid redundant API calls."""
    resp = requests.get(
        CLEARBIT_API,
        params={"domain": domain},
        headers={"Authorization": f"Bearer {os.environ['CLEARBIT_KEY']}"},
        timeout=5
    )
    if resp.status_code == 200:
        return resp.json()
    return None

def enrich_company(row):
    """Add company data based on email domain."""
    email = row.get("email", "")
    domain = email.split("@")[-1] if "@" in email else None

    if domain:
        company = _lookup_company(domain)
        if company:
            row["company_name"] = company.get("name")
            row["company_size"] = company.get("metrics", {}).get("employees")
            row["industry"] = company.get("category", {}).get("industry")

    return row
```

## Transform with state

Some transforms need to maintain state across rows — for example, computing running totals:

```python
# transforms/running_total.py
from acme.sdk import StatefulTransform

class RunningTotal(StatefulTransform):
    def setup(self, config):
        self.total = 0
        self.field = config.get("field", "amount")

    def process_row(self, row):
        self.total += row.get(self.field, 0)
        row["running_total"] = self.total
        return row
```

```yaml
transforms:
  - type: python
    class: transforms.running_total.RunningTotal
    config:
      field: revenue
```

## Testing custom transforms

Write unit tests for your transforms:

```python
# tests/test_transforms.py
from transforms.clean import normalize_phone

def test_normalize_10_digit():
    row = {"phone": "(555) 123-4567"}
    result = normalize_phone(row)
    assert result["phone"] == "+15551234567"

def test_normalize_invalid():
    row = {"phone": "123"}
    result = normalize_phone(row)
    assert result["phone"] is None

def test_normalize_missing():
    row = {"name": "Alice"}
    result = normalize_phone(row)
    assert result["phone"] is None
```

```bash
# Run transform tests
pytest tests/test_transforms.py -v
```

> [!tip] Test-driven transforms
> Write your tests before your transforms. This makes it much easier to handle edge cases and gives you confidence when refactoring. See [[Guides/Testing Pipelines|Testing Pipelines]] for integration tests.

## Performance tips

1. **Avoid per-row API calls** — use batch endpoints or caching
2. **Use generators** for memory efficiency with large datasets
3. **Profile your transforms** with `acme run --profile`
4. **Keep it simple** — complex transforms are harder to debug

## Related

- [[Core Concepts/Transforms|Transform Concepts]] — built-in transform reference
- [[API Reference/Transform API|Transform API]] — SDK reference
- [[Guides/Testing Pipelines|Testing Pipelines]] — integration testing
