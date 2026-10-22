---
description: Introduction to Acme's key abstractions — pipelines, connectors, transforms, schedulers, and events — and the design principles behind them.
---

# Core Concepts

Before diving into advanced usage, it's helpful to understand the building blocks of Acme.

## Key abstractions

```mermaid
graph TB
    subgraph Pipeline
        S[Source] --> T1[Transform]
        T1 --> T2[Transform]
        T2 --> D[Destination]
    end

    subgraph Scheduling
        SC[Scheduler] --> Pipeline
    end

    subgraph Monitoring
        Pipeline --> E[Events]
        E --> A[Alerts]
    end
```

| Concept       | Description                                               | Learn more                                 |
| ------------- | --------------------------------------------------------- | ------------------------------------------ |
| **Pipeline**  | A complete data workflow: extract → transform → load      | [[Core Concepts/Pipelines\|Pipelines]]     |
| **Connector** | A source or destination adapter (PostgreSQL, S3, etc.)    | [[Core Concepts/Connectors\|Connectors]]   |
| **Transform** | A data manipulation step (filter, map, aggregate, custom) | [[Core Concepts/Transforms\|Transforms]]   |
| **Scheduler** | Controls when and how often pipelines run                 | [[API Reference/Scheduler API\|Scheduler API]] |
| **Event**     | Metadata emitted during pipeline execution                | [[API Reference/Events API\|Events API]]       |

## Design principles

Acme is built around a few core beliefs:

1. **Configuration over code** — Most pipelines don't need custom code. YAML should be enough for 80% of use cases.
2. **Incremental by default** — Pipelines track their last run and only process new data.
3. **Fail loudly** — When something breaks, you should know immediately. See [[Guides/Error Handling|Error Handling]].
4. **Testable** — Every pipeline can be tested locally before deployment. See [[Guides/Testing Pipelines|Testing]].

> [!abstract] Architecture deep dive
> For a complete overview of how Acme processes data internally, see [[Core Concepts/Architecture|Architecture]].
