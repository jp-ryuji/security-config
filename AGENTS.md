# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Purpose

This repository provides a centralized, reusable GitHub Actions workflow for TruffleHog secret scanning. It acts as a single source of truth for secret scanning configuration across multiple repositories (public and private).

## Repository Architecture

**This repo must remain public** so other repositories can reference its workflows via the `uses:` syntax.

The sole workflow, `.github/workflows/trufflehog-base.yml`, is a reusable workflow (triggered via `workflow_call`) that other repositories invoke from their own `security-audit.yml`:

```yaml
# In a consuming repo: .github/workflows/security-audit.yml
jobs:
  secret-scan:
    uses: YOUR_USERNAME/security-config/.github/workflows/trufflehog-base.yml@main
```

## Workflow Design Decisions

- `fetch-depth: 0` — scans full git history, not just the latest commit
- `--only-verified` — only fails on confirmed live secrets to avoid false positives
- `--concurrency=10` — concurrent threads for performance on large histories
- `--include-detectors=all` — broad coverage across secret types
- `trufflesecurity/trufflehog@main` — pinned to main branch of the action
