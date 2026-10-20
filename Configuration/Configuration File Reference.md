---
description: Complete reference for the acme.yml configuration file, documenting every available setting for sources, transforms, destinations, scheduling, and more.
---

# Configuration File Reference

The `acme.yml` file is the primary configuration file for a Acme project. This page documents every available setting.

## Minimal configuration

```yaml
name: my-project
version: "1.0"

sources:
  - type: csv
    path: ./data/input.csv

destinations:
  - type: json
    path: ./output/result.json
```

## Complete reference

```yaml
# ─── Project ─────────────────────────────────────────
name: my-project # Required. Project name.
version: "1.0" # Required. Configuration version.
description: "My data pipeline" # Optional. Human-readable description.

# ─── Defaults ────────────────────────────────────────
defaults:
  batch_size: 5000 # Rows per batch (default: 5000)
  workers: 4 # Parallel transform workers (default: 4)
  timeout: 300 # Pipeline timeout in seconds (default: 300)
  retry_count: 3 # Max retries on failure (default: 3)
  retry_delay: 10 # Seconds between retries (default: 10)
  buffer_memory: 256mb # Max memory for buffering (default: 256mb)

# ─── Sources ─────────────────────────────────────────
sources:
  - type: postgres # Connector type
    name: main_db # Unique name for this source
    connection: ${DATABASE_URL} # Connection string (use env vars!)
    query: "SELECT * FROM users" # SQL query
    timeout: 60 # Query timeout in seconds
    batch_size: 10000 # Override default batch size
    incremental:
      column: updated_at # Column for incremental extraction
      initial_value: "2025-01-01" # Starting value for first run

# ─── Transforms ──────────────────────────────────────
transforms:
  - type: filter
    condition: "status = 'active'"
  - type: map
    fields:
      full_name: "first_name || ' ' || last_name"
  - type: python
    function: transforms.my_function
  - type: select
    columns: [id, full_name, email]

# ─── Destinations ────────────────────────────────────
destinations:
  - type: bigquery
    dataset: analytics
    table: users
    write_mode: upsert # append | replace | upsert | merge
    key: id # Key column for upsert/merge

# ─── Scheduling ──────────────────────────────────────
schedule: "0 */6 * * *" # Cron expression
timezone: "UTC" # Timezone for schedule (default: UTC)

# ─── Dependencies ────────────────────────────────────
depends_on:
  - load-orders # Wait for these pipelines first
  - load-exchange-rates

# ─── Error Handling ──────────────────────────────────
error_handling:
  strategy: retry # fail | retry | skip
  max_retries: 3
  retry_delay: 10s
  backoff: exponential # linear | exponential | constant
  dead_letter:
    type: json
    path: ./errors/

# ─── Schema Validation ──────────────────────────────
schema:
  id: { type: integer, required: true }
  email: { type: string, format: email }
  age: { type: integer, min: 0, max: 150 }

# ─── Notifications ───────────────────────────────────
notifications:
  on_failure:
    - type: slack
      webhook: ${SLACK_WEBHOOK}
  on_success:
    - type: slack
      webhook: ${SLACK_WEBHOOK}
      only_after_failure: true

# ─── Monitoring ──────────────────────────────────────
monitoring:
  metrics:
    enabled: true
    port: 9090 # Prometheus metrics port
  alerts:
    - name: high-latency
      condition: "duration_seconds > 300"
      channels: [slack]

# ─── Logging ─────────────────────────────────────────
logging:
  level: info # debug | info | warning | error
  format: json # json | text
  output: stdout # stdout | file
  file: /var/log/acme.log # Log file path (when output: file)

# ─── Plugins ─────────────────────────────────────────
plugins:
  connectors:
    - path: ./connectors/my_source.py
      type: my_source
  transforms:
    - path: ./transforms/custom.py

# ─── Auth (API server) ──────────────────────────────
auth:
  provider: api_key # api_key | oauth2
  # For OAuth 2.0:
  # issuer: https://auth.example.com
  # client_id: ${OAUTH_CLIENT_ID}
```

> [!warning] YAML indentation
> YAML is sensitive to indentation. Use 2 spaces (not tabs). Most errors in `acme.yml` are caused by incorrect indentation.

## Validating your configuration

```bash
acme validate
```

```
✓ acme.yml is valid
✓ All sources configured correctly
✓ All transforms found
✓ All destinations configured correctly
✓ Schedule expression valid
```

## Related

- [[Configuration/Environment Variables|Environment Variables]] — secret management
- [[Getting Started/Project Structure|Project Structure]] — file organization
- [[Core Concepts/Pipelines|Pipelines]] — pipeline configuration concepts
