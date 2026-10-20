---
description: API reference for managing source and destination connectors, including endpoints for listing, testing connections, and building custom connectors.
---

# Connector API

The Connector API provides programmatic access to manage source and destination connectors — test connections, check health, and register custom connectors.

## Endpoints

### List connectors

```
GET /api/v1/connectors
```

**Response:**

```json
{
  "data": [
    {
      "name": "users_db",
      "type": "postgres",
      "role": "source",
      "status": "healthy",
      "last_check": "2026-02-15T10:28:00Z",
      "pipelines": ["user-analytics", "user-export"]
    },
    {
      "name": "analytics",
      "type": "bigquery",
      "role": "destination",
      "status": "healthy",
      "last_check": "2026-02-15T10:28:00Z",
      "pipelines": ["user-analytics", "order-sync"]
    }
  ]
}
```

### Test a connection

```
POST /api/v1/connectors/test
```

**Request:**

```json
{
  "type": "postgres",
  "connection": "postgresql://user:pass@host:5432/db",
  "ssl": {
    "mode": "verify-full"
  }
}
```

**Response:**

```json
{
  "data": {
    "status": "connected",
    "server_version": "PostgreSQL 15.2",
    "database": "mydb",
    "tables": 47,
    "latency_ms": 12
  }
}
```

### Get connector health

```
GET /api/v1/connectors/:name/health
```

## Building custom connectors

### Source connector interface

```python
from acme.sdk import SourceConnector, Row, ConnectorConfig
from typing import Iterator, Optional
from datetime import datetime

class MySource(SourceConnector):
    """Template for a custom source connector."""

    @classmethod
    def config_schema(cls) -> dict:
        """Define required configuration fields."""
        return {
            "api_url": {"type": "string", "required": True},
            "api_key": {"type": "string", "required": True, "secret": True},
            "page_size": {"type": "integer", "default": 100},
        }

    def setup(self, config: ConnectorConfig) -> None:
        """Initialize the connector. Called once at pipeline start."""
        self.api_url = config["api_url"]
        self.api_key = config["api_key"]
        self.page_size = config.get("page_size", 100)

    def extract(self, last_run: Optional[datetime] = None) -> Iterator[Row]:
        """
        Extract data from the source.

        Args:
            last_run: Timestamp of last successful extraction.
                      None on first run.

        Yields:
            Row objects containing source data.
        """
        page = 1
        while True:
            response = self.http.get(
                f"{self.api_url}/data",
                params={
                    "since": last_run.isoformat() if last_run else None,
                    "page": page,
                    "per_page": self.page_size,
                },
                headers={"Authorization": f"Bearer {self.api_key}"},
            )
            data = response.json()

            for item in data["results"]:
                yield Row(item)

            if not data.get("has_more"):
                break
            page += 1

    def health_check(self) -> bool:
        """Check if the source is reachable."""
        resp = self.http.get(f"{self.api_url}/health")
        return resp.status_code == 200

    def teardown(self) -> None:
        """Clean up resources. Called when pipeline completes."""
        pass
```

### Destination connector interface

```python
from acme.sdk import DestinationConnector, Row, WriteMode
from typing import List

class MyDestination(DestinationConnector):
    """Template for a custom destination connector."""

    def setup(self, config):
        self.endpoint = config["endpoint"]

    def load(self, rows: List[Row], write_mode: WriteMode) -> int:
        """
        Load rows into the destination.

        Returns:
            Number of rows successfully written.
        """
        payload = [row.to_dict() for row in rows]
        response = self.http.post(
            self.endpoint,
            json={"data": payload, "mode": write_mode.value},
        )
        return response.json()["rows_written"]

    def teardown(self):
        pass
```

> [!tip] Connector testing
> Use the built-in test harness to validate your connector:
>
> ```bash
> acme test-connector ./connectors/my_source.py \
>   --config '{"api_url": "http://localhost:8080", "api_key": "test"}'
> ```

## Related

- [[Core Concepts/Connectors|Connector Concepts]] — available connectors and configuration
- [[Guides/Connecting Databases|Connecting Databases]] — database setup guide
- [[API Reference/Pipeline API|Pipeline API]] — using connectors in pipelines
