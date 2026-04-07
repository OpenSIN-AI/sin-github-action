# sin-github-action

> **GitHub Action that delegates CI/CD to n8n — zero GitHub Actions billing.**

Instead of running expensive GitHub-hosted runners, this action simply POSTs your push/PR payload to a self-hosted n8n webhook on OCI. n8n handles the actual build, test, lint — and posts commit status back to GitHub.

## Architecture

```
GitHub Push/PR
     ↓
sin-github-action (composite, uses only curl — ~2s, almost free)
     ↓ POST webhook
n8n @ n8n.delqhi.com
     ↓ SSH
OCI VM (ubuntu@92.5.60.87)
     ↓ npm run build, tsc, lint, tests
     ↓ POST /statuses
GitHub Commit Status ✅ / ❌
```

## Usage

```yaml
# .github/workflows/ci.yml
name: CI → n8n

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  dispatch:
    runs-on: ubuntu-latest
    steps:
      - uses: OpenSIN-AI/sin-github-action@main
        with:
          n8n_webhook_url: ${{ secrets.N8N_CI_WEBHOOK_URL }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `n8n_webhook_url` | ✅ | Full n8n webhook URL (from repo secret) |
| `github_token` | ✅ | GitHub token for commit status auth |
| `repo` | ❌ | Override repo (default: `github.repository`) |
| `sha` | ❌ | Override commit SHA (default: `github.sha`) |
| `ref` | ❌ | Override git ref (default: `github.ref`) |
| `event` | ❌ | Override event name (default: `github.event_name`) |

## Setup

1. Add secret `N8N_CI_WEBHOOK_URL` to your repo pointing to the n8n CI webhook
2. Import the n8n workflow from `n8n-workflows/opensin-ci-pipeline.json`
3. Replace your `ci.yml` with the example above

## Why

GitHub Actions billing was blocking our CI on OpenSIN-AI. n8n runs 24/7 on our OCI VM (always-free tier) — unlimited builds, zero cost.
