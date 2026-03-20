# Versioning Workflow Setup Guide

This guide explains how to integrate the reusable version-bump workflow from `TEST-Consulting/github-workflows` into your repository.

## What the workflow does

When a PR is opened, the workflow automatically:

1. Reads the version bump type (`[MAJOR]`, `[MINOR]`, or `[PATCH]`) from the PR title
2. Calculates the new version based on `version.json` on `main`
3. Updates `version.json` with the new version
4. Replaces the `## Unreleased` header in `CHANGELOG.md` with `## [X.Y.Z] - YYYY-MM-DD`
5. If `RELEASE_NOTES.md` exists and has an `## Unreleased` header, updates it the same way
6. Commits and pushes the changes to your PR branch

## Required files

### 1. `version.json`

Create a `version.json` in the root of your repository:

```json
{
  "major": 1,
  "minor": 0,
  "patch": 0
}
```

You do not need to update this file manually — the workflow handles it.

### 2. `CHANGELOG.md`

Create a `CHANGELOG.md` in the root of your repository. When adding changes, always put them under an `## Unreleased` header:

```markdown
# Changelog

## Unreleased

- Added new feature X
- Fixed bug Y
```

The workflow will replace `## Unreleased` with the versioned header (e.g., `## [1.1.0] - 2026-03-20`). After your PR is merged, add a new `## Unreleased` section at the top for the next round of changes.

### 3. `RELEASE_NOTES.md` (optional)

If your project maintains a separate `RELEASE_NOTES.md`, the workflow will update it the same way — replacing `## Unreleased` with the versioned header. If the file doesn't exist, this step is skipped. If it exists but has no `## Unreleased` header, the workflow prints a warning and continues.

## PR title format

Your PR title **must** start with one of:

- `[MAJOR]` — breaking changes (resets minor and patch to 0)
- `[MINOR]` — new features, backwards-compatible (resets patch to 0)
- `[PATCH]` — bug fixes, backwards-compatible

Examples:
- `[MINOR] Add user export endpoint`
- `[PATCH] Fix null pointer in auth middleware`
- `[MAJOR] Redesign API response format`

## Prerequisites

The workflow uses a GitHub App token to push commits that trigger other status checks. Your org needs:

1. A GitHub App with **Contents: Read & Write** permission, installed on the org
2. Org-level secrets: `APP_ID` (the app's ID) and `APP_PRIVATE_KEY` (the app's generated private key)

If these are already set up, you don't need to do anything — just call the workflow as shown below.

## Calling the workflow

Add a workflow file in your repo at `.github/workflows/version-bump.yml`:

```yaml
name: Version Bump

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  bump:
    uses: TEST-Consulting/github-workflows/.github/workflows/version-bump.yml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

## Typical workflow

1. Create a branch and make your changes
2. Add your changes under `## Unreleased` in `CHANGELOG.md` (and optionally `RELEASE_NOTES.md`)
3. Open a PR with a title starting with `[MAJOR]`, `[MINOR]`, or `[PATCH]`
4. The workflow runs automatically, updates `version.json` and the changelog headers, and pushes a commit to your branch
5. Merge the PR — the version is now recorded in `main`
6. Add a new `## Unreleased` section to `CHANGELOG.md` for the next set of changes
