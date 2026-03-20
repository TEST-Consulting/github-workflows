# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains reusable GitHub Actions workflows for the TEST-Consulting organization. Workflows are defined in `.github/workflows/` and designed to be called from other repositories via `workflow_call`.

## Architecture

The repo currently contains one reusable workflow:

- **`version-bump.yml`** — A reusable workflow (triggered via `workflow_call`) that automates version bumping on PRs. It:
  1. PR title starts with `[MAJOR]`, `[MINOR]`, or `[PATCH]`
  2. `version.json` exists and was updated with the correct semver bump
  3. `CHANGELOG.md` exists, was updated, and contains an entry matching the format `## [X.Y.Z] - YYYY-MM-DD`

### Versioning Contract

Consuming repos must maintain:
- A `version.json` file with `{"major": N, "minor": N, "patch": N}` structure
- A `CHANGELOG.md` with entries formatted as `## [X.Y.Z] - YYYY-MM-DD`

The workflow compares the PR branch's files against `origin/main` to validate correct increments (e.g., MINOR bump resets patch to 0, MAJOR resets both minor and patch to 0).

## Development Notes

- There is no build system, test suite, or linter — this is a pure GitHub Actions workflow repo.
- Workflows use `jq` for JSON parsing and `grep -P` (Perl regex) for pattern matching.
- The checkout step uses `fetch-depth: 0` to access full git history for version comparison.
