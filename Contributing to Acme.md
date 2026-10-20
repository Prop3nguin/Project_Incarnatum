---
description: Guidelines for contributing to Acme, including development setup, project structure, and how to submit pull requests.
---

# Contributing to Acme

We welcome contributions of all kinds — bug fixes, new connectors, documentation improvements, and feature ideas.

## Getting started

1. Fork the repository on GitHub
2. Clone your fork locally
3. Install dependencies:

```bash
git clone https://github.com/your-username/acme.git
cd acme
make install
```

4. Create a feature branch:

```bash
git checkout -b feature/my-new-connector
```

## Development setup

> [!important]
> Acme requires Python 3.10+ and Node.js 18+ for development.

```bash
# Install development dependencies
pip install -e ".[dev]"

# Run the test suite
make test

# Start the development server
make dev
```

## Project structure

```
acme/
├── src/
│   ├── connectors/     # Source and destination connectors
│   ├── transforms/     # Built-in transform functions
│   ├── engine/         # Core pipeline engine
│   ├── scheduler/      # Cron and trigger scheduling
│   └── api/            # REST API server
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/               # Documentation (you are here!)
└── examples/           # Example pipeline configurations
```

## Submitting a pull request

1. Ensure all tests pass: `make test`
2. Add tests for any new functionality
3. Update documentation if needed
4. Submit your PR with a clear description

> [!tip]
> See the [[Getting Started/Project Structure|Project Structure]] page for a deeper dive into how the codebase is organized.

## Code of conduct

Be respectful, be kind, be constructive. See `CODE_OF_CONDUCT.md` for details.
