---
description: Build and run your first Acme pipeline in under 5 minutes with a step-by-step walkthrough using CSV data.
---

# Quickstart

Build and run your first Acme pipeline in under 5 minutes.

## Prerequisites

- Acme CLI installed ([[Getting Started/Installation|Installation Guide]])
- A running PostgreSQL database (or use our demo database)

## Step 1: Create a new project

```bash
mkdir my-first-pipeline && cd my-first-pipeline
acme init
```

This creates the following structure:

```
my-first-pipeline/
├── acme.yml        # Pipeline configuration
├── .env                # Environment variables
└── transforms/         # Custom transform functions
    └── .gitkeep
```

## Step 2: Define your pipeline

Edit `acme.yml`:

```yaml
name: hello-world
version: "1.0"

sources:
  - type: csv
    name: sample_data
    path: ./data/users.csv

transforms:
  - type: filter
    condition: "age >= 18"
  - type: rename
    columns:
      email_address: email
      full_name: name

destinations:
  - type: json
    path: ./output/filtered_users.json
    pretty: true
```

## Step 3: Add sample data

Create `data/users.csv`:

```csv
full_name,email_address,age,country
Alice Johnson,alice@example.com,28,US
Bob Smith,bob@example.com,16,UK
Carol White,carol@example.com,34,CA
David Brown,david@example.com,12,AU
Eva Martinez,eva@example.com,45,MX
```

## Step 4: Run the pipeline

```bash
acme run
```

You should see output like:

```
✓ Loaded pipeline: hello-world v1.0
✓ Source: sample_data (5 rows)
✓ Transform: filter (3 rows passed)
✓ Transform: rename (3 columns mapped)
✓ Destination: filtered_users.json (3 rows written)

Pipeline completed in 0.3s
```

> [!success] Congratulations!
> You've just built and run your first Acme pipeline. The filtered data is in `./output/filtered_users.json`.

## Step 5: Check the output

```json
[
  {
    "name": "Alice Johnson",
    "email": "alice@example.com",
    "age": 28,
    "country": "US"
  },
  {
    "name": "Carol White",
    "email": "carol@example.com",
    "age": 34,
    "country": "CA"
  },
  {
    "name": "Eva Martinez",
    "email": "eva@example.com",
    "age": 45,
    "country": "MX"
  }
]
```

## What's next?

- [[Getting Started/Project Structure|Project Structure]] — understand the files in a Acme project
- [[Getting Started/Your First Pipeline|Your First Pipeline]] — a more detailed walkthrough with a real database
- [[Core Concepts/Pipelines|Pipelines]] — learn how pipelines work under the hood
