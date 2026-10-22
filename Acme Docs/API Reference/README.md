---
description: Overview of the Acme REST API, including authentication, response formats, rate limits, and links to all API module references.
---

# API Reference

Acme provides a RESTful API for managing pipelines, triggering runs, and monitoring status programmatically.

## Base URL

```
https://your-acme-instance.com/api/v1
```

## Authentication

All API requests require authentication. See [[Guides/Authentication|Authentication]] for setup.

```bash
curl -H "Authorization: Bearer df_key_abc123..." \
  https://acme.example.com/api/v1/pipelines
```

## API modules

| Module        | Description                               | Reference                              |
| ------------- | ----------------------------------------- | -------------------------------------- |
| **Client**    | SDK initialization and configuration      | [[API Reference/Client API\|Client]]       |
| **Pipeline**  | Create, update, run, and manage pipelines | [[API Reference/Pipeline API\|Pipeline]]   |
| **Connector** | Manage source and destination connectors  | [[API Reference/Connector API\|Connector]] |
| **Transform** | Register and manage transform functions   | [[API Reference/Transform API\|Transform]] |
| **Scheduler** | Schedule and trigger pipeline runs        | [[API Reference/Scheduler API\|Scheduler]] |
| **Events**    | Subscribe to pipeline lifecycle events    | [[API Reference/Events API\|Events]]       |

## Response format

All API responses follow a consistent format:

```json
{
  "data": { ... },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2026-02-15T06:00:00Z"
  }
}
```

### Error responses

```json
{
  "error": {
    "code": "PIPELINE_NOT_FOUND",
    "message": "Pipeline 'user-analytics' not found",
    "details": {
      "pipeline_name": "user-analytics"
    }
  },
  "meta": {
    "request_id": "req_def456",
    "timestamp": "2026-02-15T06:00:01Z"
  }
}
```

## Rate limits

| Plan       | Requests/min | Burst     |
| ---------- | ------------ | --------- |
| Free       | 60           | 10        |
| Pro        | 600          | 100       |
| Enterprise | Unlimited    | Unlimited |

> [!info] Rate limit headers
> Every response includes rate limit headers:
>
> ```
> X-RateLimit-Limit: 600
> X-RateLimit-Remaining: 594
> X-RateLimit-Reset: 1708934460
> ```

## SDKs

Official client libraries:

```bash
# Python
pip install acme-sdk

# JavaScript/TypeScript
npm install @acme/sdk

# Go
go get github.com/acme/acme-go
```

See [[API Reference/Client API|Client]] for SDK usage examples.
