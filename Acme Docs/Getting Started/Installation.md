---
description: How to install the Acme CLI via pip, Homebrew, or Docker, plus post-installation setup, shell completions, and upgrading.
---

# Installation

Acme can be installed via pip, Homebrew, or Docker.

## Using pip (recommended)

```bash
pip install acme-cli
```

Verify the installation:

```bash
acme --version
# Acme CLI v2.4.0
```

## Using Homebrew (macOS)

```bash
brew tap acme/tap
brew install acme
```

## Using Docker

```bash
docker pull acme/acme:latest
docker run --rm acme/acme:latest --version
```

> [!warning] Python version
> Acme requires **Python 3.10 or higher**. If you're using an older version, consider using [pyenv](https://github.com/pyenv/pyenv) to manage multiple Python versions.

## Post-installation setup

After installing, initialize your configuration:

```bash
acme init
```

This creates a `~/.acme/config.yml` file with default settings. See [[Configuration/Configuration File Reference|Configuration]] for all available options.

## Shell completions

Enable tab completions for your shell:

```bash
# Bash
acme completions bash >> ~/.bashrc

# Zsh
acme completions zsh >> ~/.zshrc

# Fish
acme completions fish > ~/.config/fish/completions/acme.fish
```

## Upgrading

```bash
pip install --upgrade acme-cli
```

> [!note]
> After upgrading, run `acme migrate up` to apply any database schema changes. See the [[CHANGELOG|Changelog]] for breaking changes.

## Next steps

Once installed, head to the [[Getting Started/Quickstart|Quickstart]] to build your first pipeline.
