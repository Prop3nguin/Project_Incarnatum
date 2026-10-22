---
description: How to manage secrets and per-environment settings using environment variables, .env files, and CI/CD integration in Acme.
---

# Environment Variables

Environment variables are the recommended way to manage secrets (database passwords, API keys) and per-environment settings in Acme.

## How it works

Reference environment variables in `acme.yml` using `${VARIABLE_NAME}`:

```yaml
sources:
  - type: postgres
    connection: ${DATABASE_URL}

notifications:
  on_failure:
    - type: slack
      webhook: ${SLACK_WEBHOOK}
```

Acme resolves these at runtime from:

1. System environment variables
2. `.env` file in the project root
3. `.env.{environment}` file (when using `--env`)

## The `.env` file

Create a `.env` file in your project root:

```bash
# Database
DATABASE_URL=postgresql://localhost:5432/mydb
ANALYTICS_DB_URL=postgresql://localhost:5432/analytics

# API Keys
SLACK_WEBHOOK=https://hooks.slack.com/services/T00/B00/xxx
CLEARBIT_KEY=sk_abc123...

# Acme settings
ACME_LOG_LEVEL=info
ACME_WORKERS=4
```

> [!danger] Never commit `.env` files
> Add `.env` to your `.gitignore` immediately. Instead, commit a `.env.example` with placeholder values:
>
> ```bash
> # .env.example
> DATABASE_URL=postgresql://user:password@host:5432/dbname
> SLACK_WEBHOOK=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
> ```

## Per-environment configuration

Use environment-specific files for different deployment targets:

```
.env                    # Shared defaults (not committed)
.env.development        # Local development
.env.staging            # Staging environment
.env.production         # Production environment
```

Activate an environment:

```bash
acme run --env production
```

This loads variables in order:

1. `.env` (base)
2. `.env.production` (overrides)
3. System environment (highest priority)

### Example

```bash
# .env.development
DATABASE_URL=postgresql://localhost:5432/mydb_dev
ACME_LOG_LEVEL=debug
ACME_WORKERS=2

# .env.production
DATABASE_URL=postgresql://prod-db:5432/mydb
ACME_LOG_LEVEL=warning
ACME_WORKERS=8
```

## Built-in variables

Acme recognizes these environment variables:

| Variable            | Description                 | Default  |
| ------------------- | --------------------------- | -------- |
| `ACME_LOG_LEVEL`    | Logging level               | `info`   |
| `ACME_WORKERS`      | Number of transform workers | `4`      |
| `ACME_BATCH_SIZE`   | Default batch size          | `5000`   |
| `ACME_TIMEOUT`      | Pipeline timeout (seconds)  | `300`    |
| `ACME_METRICS_PORT` | Prometheus metrics port     | `9090`   |
| `ACME_API_PORT`     | API server port             | `8080`   |
| `ACME_STATE_DIR`    | State directory path        | `.acme/` |

## Runtime variables

Acme also provides runtime variables you can use in your pipeline configuration:

| Variable         | Description                      | Example                |
| ---------------- | -------------------------------- | ---------------------- |
| `:last_run`      | Timestamp of last successful run | `2026-02-15T06:00:00Z` |
| `:run_id`        | Current run identifier           | `run_2026_02_15_001`   |
| `:pipeline_name` | Current pipeline name            | `user-analytics`       |
| `:environment`   | Current environment name         | `production`           |

```yaml
sources:
  - type: postgres
    query: "SELECT * FROM users WHERE updated_at > :last_run"

error_handling:
  dead_letter:
    path: ./errors/${pipeline_name}_${run_id}.json
```

## Secrets in CI/CD

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
    steps:
      - run: acme run --env production
```

### Docker

```bash
docker run --env-file .env.production acme/acme:latest
```

> [!tip] Secret management tools
> For production environments, consider using a dedicated secret manager:
>
> - **AWS Secrets Manager** or **Parameter Store**
> - **HashiCorp Vault**
> - **Google Secret Manager**
> - **Azure Key Vault**
>
> Acme can read secrets from these sources with plugins.

## Related

- [[Configuration/Configuration File Reference|Configuration File]] — full `acme.yml` reference
- [[Guides/Deployment|Deployment]] — environment setup for production
- [[Guides/Authentication|Authentication]] — API key management
