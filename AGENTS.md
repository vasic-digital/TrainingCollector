# AGENTS.md - Multi-Agent Coordination Guide

## Overview

This document provides guidance for AI agents working with the `digital.vasic.trainingcollector` module.

## Module Identity

- **Module path**: `digital.vasic.trainingcollector`
- **Language**: Go 1.24+
- **Dependencies**: `github.com/stretchr/testify` (tests only)
- **Scope**: Generic, reusable training data collection. No application-specific logic.

## Package Responsibilities

| Package | Owner Concern | Agent Must Not |
|---------|--------------|----------------|
| `pkg/training` | Screenshot saving, JSONL export, statistics | Add ML training logic, import non-stdlib production deps |

## Coordination Rules

### 1. Thread Safety Invariants

Every exported method on `TrainingCollector` is safe for concurrent use. Agents must:

- Never remove mutex protection from shared state.
- Never introduce a public method that requires external synchronization.
- Always run `go test -race` after changes.

### 2. Interface Contracts

The `TrainingCollector` API is a stability boundary. Breaking changes require explicit human approval:

- `NewTrainingCollector(outputDir)` constructor signature
- `Record(...)` parameter list
- `Export(path)` return type
- `TrainingPair`, `Action`, `TrainingStats` struct fields

### 3. JSONL Format Contract

The JSONL export format must remain backwards-compatible. Fields may be added but never removed or renamed.

### 4. Test Requirements

- All tests use `testify/assert` and `testify/require`.
- Test naming convention: `Test<Struct>_<Method>_<Scenario>`.
- Tests use `t.TempDir()` for output directories -- never write to fixed paths.
- Race detector must pass: `go test ./... -race`.

## Agent Workflow

### Before Making Changes

```bash
go build ./...
go test ./... -count=1 -race
```

### After Making Changes

```bash
gofmt -w .
go vet ./...
go test ./... -count=1 -race
```

### Commit Convention

```
<type>(<package>): <description>

# Examples:
feat(training): add filtering by platform and model
fix(training): handle concurrent directory creation
test(training): add export round-trip verification
```

## Boundaries

### What Agents May Do

- Fix bugs in any package.
- Add tests for uncovered code paths.
- Refactor internals without changing exported APIs.
- Add new exported methods that extend existing types.
- Update documentation to match code.

### What Agents Must Not Do

- Break existing exported interfaces or method signatures.
- Remove thread safety guarantees.
- Add application-specific logic (this is a generic library).
- Add ML/training dependencies (this module is data collection only).
- Introduce new external dependencies without human approval.
- Modify `go.mod` without explicit instruction.

## Key Files

| File | Purpose |
|------|---------|
| `pkg/training/collector.go` | All production code |
| `pkg/training/collector_test.go` | All tests |
| `go.mod` | Module definition |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | Agent build/test guidance |
