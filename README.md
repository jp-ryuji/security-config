# Centralized Secret Scanning

This repository hosts the shared TruffleHog configuration. Using a reusable workflow allows you to maintain a single security standard across all repositories—both public and private—without duplicating code.

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
