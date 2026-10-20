---
description: Step-by-step guide for connecting PostgreSQL, MySQL, and MongoDB as data sources in Acme, including SSL setup, incremental extraction, and CDC.
---

# Connecting Databases

This guide covers setting up database connections for the most common sources: PostgreSQL, MySQL, and MongoDB.

## PostgreSQL

### Connection string

```yaml
sources:
  - type: postgres
    name: my_postgres
    connection: postgresql://user:password@host:5432/database
```

### Using SSL

```yaml
sources:
  - type: postgres
    name: my_postgres
    connection: ${DATABASE_URL}
    ssl:
      mode: verify-full
      ca_cert: ./certs/server-ca.pem
      client_cert: ./certs/client-cert.pem
      client_key: ./certs/client-key.pem
```

> [!important] SSL in production
> Always use SSL when connecting to production databases. Set `ssl.mode` to `verify-full` to prevent man-in-the-middle attacks.

### Incremental extraction

PostgreSQL supports incremental extraction out of the box:

```yaml
sources:
  - type: postgres
    name: users
    connection: ${DATABASE_URL}
    query: |
      SELECT * FROM users
      WHERE updated_at > :last_run
    incremental:
      column: updated_at
      initial_value: "2025-01-01T00:00:00Z"
```

The `:last_run` parameter is automatically replaced with the timestamp of the last successful pipeline run.

### Change Data Capture (CDC)

For real-time data, use PostgreSQL's logical replication:

```yaml
sources:
  - type: postgres
    name: users_cdc
    connection: ${DATABASE_URL}
    mode: cdc
    publication: acme_pub
    slot: acme_slot
    tables:
      - public.users
      - public.orders
```

> [!note]
> CDC requires PostgreSQL 10+ with `wal_level = logical`. See [[Guides/Real-time Streaming|Real-time Streaming]] for setup instructions.

## MySQL

### Connection string

```yaml
sources:
  - type: mysql
    name: my_mysql
    connection: mysql://user:password@host:3306/database
```

### Character encoding

```yaml
sources:
  - type: mysql
    name: my_mysql
    connection: ${MYSQL_URL}
    charset: utf8mb4
```

> [!warning] MySQL 5.7 vs 8.0
> Acme supports both MySQL 5.7 and 8.0, but some features (like window functions in queries) require MySQL 8.0.

## MongoDB

### Connection string

```yaml
sources:
  - type: mongodb
    name: my_mongo
    connection: mongodb://user:password@host:27017/database
    collection: users
```

### Querying nested documents

MongoDB sources support dot notation for nested fields:

```yaml
sources:
  - type: mongodb
    name: my_mongo
    connection: ${MONGO_URL}
    collection: users
    projection:
      - _id
      - name
      - address.city
      - address.country
    filter:
      status: active
```

## Testing connections

Verify your connection before running a pipeline:

```bash
acme test-connection --type postgres --connection "${DATABASE_URL}"
```

```
✓ Connected to PostgreSQL 15.2
✓ Database: mydb
✓ Tables accessible: 47
✓ Latency: 12ms
```

## Related

- [[Core Concepts/Connectors|Connectors]] — full list of available connectors
- [[Configuration/Environment Variables|Environment Variables]] — secure credential management
- [[Guides/Real-time Streaming|Real-time Streaming]] — CDC and streaming setup
