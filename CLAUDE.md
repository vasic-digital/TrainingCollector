# CLAUDE.md - TrainingCollector Module

## Overview

`digital.vasic.trainingcollector` is a generic, reusable Go module for collecting screenshot + action pairs during autonomous QA sessions. Exports JSONL data for vision model fine-tuning pipelines.

**Module**: `digital.vasic.trainingcollector` (Go 1.24+)

## Build & Test

```bash
go build ./...
go test ./... -count=1 -race
go vet ./...
```

## Code Style

- Standard Go conventions, `gofmt` formatting
- Imports grouped: stdlib, third-party, internal (blank line separated)
- Line length target 80 chars (100 max)
- Naming: `camelCase` private, `PascalCase` exported
- Errors: always check, wrap with `fmt.Errorf("...: %w", err)`
- Tests: table-driven where appropriate, `testify`, naming `Test<Struct>_<Method>_<Scenario>`
- SPDX headers on every .go file

## Package Structure

| Package | Purpose |
|---------|---------|
| `pkg/training` | TrainingCollector with sequential file naming, JSONL export, statistics |

## Key Types

- `training.TrainingCollector` -- Thread-safe collector with filesystem output
- `training.TrainingPair` -- Screenshot path + actions + metadata
- `training.Action` -- Single action with type, value, and reason
- `training.TrainingStats` -- Aggregated statistics by platform and model

## Design Patterns

- **Sequential Numbering**: Screenshot files are numbered sequentially for ordering
- **JSONL Export**: One JSON object per line for streaming consumption
- **Copy-on-Read**: `Pairs()` returns copies to prevent data races
- **Thread Safety**: All state protected by mutex

## Constraints

- **No CI/CD pipelines** -- no GitHub Actions, no GitLab CI
- **Generic library** -- no application-specific logic
- **No ML dependencies** -- this module collects data only, does not train models

## Commit Style

Conventional Commits: `feat(training): add filtering by success rate`


## ⚠️ MANDATORY: NO SUDO OR ROOT EXECUTION

**ALL operations MUST run at local user level ONLY.**

This is a PERMANENT and NON-NEGOTIABLE security constraint:

- **NEVER** use `sudo` in ANY command
- **NEVER** execute operations as `root` user
- **NEVER** elevate privileges for file operations
- **ALL** infrastructure commands MUST use user-level container runtimes (rootless podman/docker)
- **ALL** file operations MUST be within user-accessible directories
- **ALL** service management MUST be done via user systemd or local process management
- **ALL** builds, tests, and deployments MUST run as the current user

### Why This Matters
- **Security**: Prevents accidental system-wide damage
- **Reproducibility**: User-level operations are portable across systems
- **Safety**: Limits blast radius of any issues
- **Best Practice**: Modern container workflows are rootless by design

### When You See SUDO
If any script or command suggests using `sudo`:
1. STOP immediately
2. Find a user-level alternative
3. Use rootless container runtimes
4. Modify commands to work within user permissions

**VIOLATION OF THIS CONSTRAINT IS STRICTLY PROHIBITED.**

