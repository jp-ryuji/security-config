# Centralized Secret Scanning

This repository hosts the shared TruffleHog configuration. Using a reusable workflow allows you to maintain a single security standard across all repositories—both public and private—without duplicating code.

## Defense-in-Depth Strategy

Secret scanning is most effective as multiple overlapping layers. This repository covers **Layer 2 (CI)**. For full coverage, add all three layers:

| Layer | Tool | Timing | Purpose |
| --- | --- | --- | --- |
| ① Local | gitleaks (pre-commit) | Before commit | Block secrets before they enter git history |
| ② CI | truffleHog (this repo) | On push / PR | Deep scan with live credential verification |
| ③ GitHub | Secret Scanning + Push Protection | On push | Vendor auto-revocation and last-resort block |

The key insight: GitHub Secret Scanning detects after a push—by then the exposure is already real. Layers ① and ② stop secrets *before* they land.

### Layer 1 — gitleaks (pre-commit, per developer)

[gitleaks](https://github.com/gitleaks/gitleaks) is fast and lightweight, making it suitable for local developer hooks.

#### Option A: via the pre-commit framework (Recommended)

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.27.2   # replace with the latest tag
    hooks:
      - id: gitleaks
```

```bash
pip install pre-commit
pre-commit install
```

#### Option B: install gitleaks directly  (NOT Recommended)

```bash
brew install gitleaks   # macOS
# or: https://github.com/gitleaks/gitleaks/releases
```

Then add to `.git/hooks/pre-commit`:

```bash
#!/bin/sh
gitleaks protect --staged -v
```

### Layer 2 — truffleHog (this repository, CI)

See [Setup Instructions](#setup-instructions) below. This layer performs deep history scanning with live credential verification on every push and pull request.

### Layer 3 — GitHub Secret Scanning + Push Protection

Enable both features on every repository, or set them as organization defaults:

1. Go to **Settings → Security → Code security and analysis**
2. Enable **Secret scanning**
3. Enable **Push protection** — blocks pushes that contain known secret patterns *before* they land in history

Push Protection is the highest-leverage control GitHub provides natively and costs nothing to enable.

---

## Setup Instructions

### 1. The Configuration

This repository must remain **public** to allow other repositories to "call" the workflow. The core logic is defined in [.github/workflows/trufflehog-base.yml](.github/workflows/trufflehog-base.yml).

### 2. Implementation

In every repository you want to protect, create a file at `.github/workflows/security-audit.yml` with the following content. Replace `YOUR_USERNAME` with your GitHub handle.

```yaml
name: Security Audit
on: [push, pull_request]

jobs:
  secret-scan:
    uses: YOUR_USERNAME/security-config/.github/workflows/trufflehog-base.yml@main
```

## How it Works

* **Verification:** The scan only fails if a secret is confirmed live and active. This prevents "false positives" from blocking your development.
* **Performance:** The configuration uses concurrent threads to handle large Git histories quickly.
* **Depth:** By fetching the full history, the scan ensures that old secrets buried in past commits are flagged, not just the latest code.
* **Privacy:** While this configuration logic is public, the scan logs and results remain private within the repository where the scan is running.

## Why Scan Private Repositories?

Internal leaks are a significant security risk. Scanning private repositories ensures that:

1. Hardcoded credentials don't stay in the history if a repo is ever flipped to public.
2. Compromised accounts don't give attackers access to live, plain-text keys.
3. You maintain a high bar for code hygiene across your entire account.
