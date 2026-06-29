---
name: github-actions-latest
description: Get the latest version of GitHub Actions from github.com/actions/ and update workflow files. Use when the user asks about Action versions, wants to upgrade CI/CD workflows, pin actions to specific versions, or batch-check versions across a repo.
---

# GitHub Actions Latest

Fetch the latest release version of any GitHub Action and apply it to workflow
files (`.github/workflows/*.yml`).

## When to Use

- User asks "what's the latest version of actions/checkout?"
- User wants to upgrade or pin an action to a specific version
- User asks to batch-check all action versions used in a repo
- User mentions an action that may be out of date

## Current Versions

The latest versions of every repo under [`github.com/actions/`](https://github.com/actions/)
are tracked in [`actions-versions.md`](actions-versions.md). **Always read this file
first** before answering a version question — it is auto-refreshed weekly.

## Workflow

### 1. Look up the target action

If the user names a specific action (e.g. `actions/setup-go`), start by reading
[`actions-versions.md`](actions-versions.md) for the cached latest version.

If they ask about an action **not** in that file, fetch it live:

```bash
gh release view -R <owner>/<repo> --json tagName -q '.tagName'
```

### 2. Find where it's used

```bash
grep -rn "uses: <owner>/<repo>@" .github/workflows/
```

### 3. Apply the update

Replace the `@<old-version>` with `@<new-major-version>` in each workflow file,
then verify the YAML is still valid.

### Example

```bash
# 1. Read cached versions
cat actions-versions.md | grep setup-go
# → actions/setup-go | 5 | v5.4.0 | 2026-06-29

# 2. Find usages
grep -rn "uses: actions/setup-go@" .github/workflows/
# → .github/workflows/ci.yml:15: uses: actions/setup-go@v4

# 3. Edit: v4 → v5
```

## Version Pinning Reference

| Pattern | Upgrade behavior | Risk |
|---|---|---|
| `@v4` | Auto within major | Medium — safe for most cases |
| `@v4.2.0` | Manual, exact minor | Low — pinned to release |
| `@11bd719...` | Manual, commit SHA | Lowest — supply-chain safe |
| `@main` | Always bleeding edge | High — breaks on upstream changes |

**Recommendation**: Use `@v4` style for dev/CI, pin to SHA in production.

## Batch Check

To check every action used across a repo's workflows:

```bash
grep -rohP 'uses:\s*\K[^@]+' .github/workflows/ | sort -u |
  while read action; do
    latest=$(gh release view -R "$action" --json tagName -q '.tagName' 2>/dev/null || echo "N/A")
    echo "$action → $latest"
  done
```

## Dependabot Automation

Add `.github/dependabot.yml` for automated PRs on new releases:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```
