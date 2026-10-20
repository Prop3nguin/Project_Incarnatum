---
description: Overview of Acme configuration sources and precedence, from command-line flags to project and user config files.
---

# Configuration

Acme is configured through a combination of YAML files and environment variables. This section covers all available settings.

## Configuration sources

Acme reads configuration from these sources, in order of precedence (highest first):

```mermaid
graph TD
    A["Command-line flags"] --> E[Final Config]
    B["Environment variables"] --> E
    C["acme.yml"] --> E
    D["~/.acme/config.yml"] --> E

    style A fill:#f87171,stroke:#dc2626,color:#fff
    style D fill:#93c5fd,stroke:#3b82f6,color:#fff
```

1. **Command-line flags** — override everything (`--batch-size 1000`)
2. **Environment variables** — per-environment settings (`ACME_BATCH_SIZE=1000`)
3. **Project config** — `acme.yml` in the project root
4. **User config** — `~/.acme/config.yml` for machine-wide defaults

## Guides

- [[Configuration/Configuration File Reference|Configuration File Reference]] — complete `acme.yml` reference
- [[Configuration/Environment Variables|Environment Variables]] — managing secrets and per-environment settings

> [!tip] Getting started?
> For most projects, the defaults work well. Start with a minimal `acme.yml` and add settings as needed.
