# Plan: Sprint 0 - Quality Pipeline

**Date**: 2026-02-08
**Goal**: Full Extreme Vibe Coding infrastructure before the first line of feature code
**Master plan**: `2026-02-07-master-plan-bootstrap-3months.md`
**Manifesto**: `../../EXTREME-VIBE-CODING.md`

## Philosophy

Sprint 0 is the foundation. No feature code is written until every quality gate
is in place and enforced. The first test in the repo is an architectural test.
The first CI run validates the quality pipeline itself. From the very first
feature commit, everything is gated.

---

## Steps

### S0.1 - Project skeleton

- [ ] `go mod init github.com/DevOpsCraftsman/gogo`
- [ ] Directory structure (hexagonal):
  ```
  internal/domain/
  internal/app/
  internal/port/
  internal/adapter/
  internal/config/
  cmd/gg/
  cmd/gogod/
  test/arch/
  test/acceptance/features/
  test/acceptance/steps/
  ```
- [ ] Empty `main.go` in `cmd/gg/` and `cmd/gogod/` (compilable skeletons)
- [ ] `.gitignore` (binaries, `.gogo/state/`, coverage files, vendor)

### S0.2 - Architectural tests (FIRST tests in the repo)

- [ ] `test/arch/hex_boundaries_test.go`:
  - `domain/` must NEVER import `adapter/`
  - `domain/` must NEVER import `app/`
  - `app/` must NEVER import `adapter/`
  - `port/` must contain ONLY interfaces (no structs with methods)
- [ ] Verify tests run green on empty skeleton
- [ ] Verify tests would FAIL if a bad import is introduced (red-green validation)

### S0.3 - Linting (strict, all dials to 11)

- [ ] `.golangci.yaml` with strict config:
  - `errcheck`, `govet`, `staticcheck`, `unused`, `gosimple`
  - `revive` (strict rules), `gocritic`, `gocyclo` (threshold: 10)
  - `dupl` (duplicate detection), `goconst`, `misspell`
  - `exhaustive` (switch exhaustiveness), `nolintlint`
  - `exportloopref`, `prealloc`, `bodyclose`
  - NilAway enabled
- [ ] Verify: `golangci-lint run ./...` passes on skeleton

### S0.4 - Makefile

- [ ] Targets:

  | Target       | Command                                                    |
  |--------------|------------------------------------------------------------|
  | `lint`       | `golangci-lint run ./...`                                  |
  | `test`       | `go test -race -count=1 ./...`                             |
  | `test-unit`  | `go test -race -count=1 -short ./...`                      |
  | `test-arch`  | `go test ./test/arch/...`                                  |
  | `test-acc`   | `go test ./test/acceptance/...`                            |
  | `coverage`   | `go test -coverprofile -covermode=atomic` + threshold check|
  | `mutation`   | `go-mutesting ./...` + score check                         |
  | `security`   | `govulncheck ./...`                                        |
  | `secrets`    | `gitleaks detect --source .`                               |
  | `licenses`   | `go-licenses check ./...`                                  |
  | `build`      | `go build ./cmd/...`                                       |
  | `arch`       | alias for `test-arch`                                      |
  | `all`        | `lint + test + coverage + security + arch + build`         |
  | `gate`       | same as `all` (what the gated pipeline runs)               |

### S0.5 - Pre-commit hooks

- [ ] Install mechanism: script or `Makefile` target (`make install-hooks`)
- [ ] `.githooks/pre-commit`:
  - `gofmt -l` (formatting check, zero tolerance)
  - `golangci-lint run --fix=false` on staged files
  - `gitleaks protect --staged` (secret detection)
  - `go test ./test/arch/...` (arch boundary check)
- [ ] Configure git: `git config core.hooksPath .githooks`
- [ ] Document: `--no-verify` is forbidden (mentioned in CONTRIBUTING or rules)

### S0.6 - Pre-push hooks

- [ ] `.githooks/pre-push`:
  - `go test -race -count=1 ./...` (full test suite)
  - Coverage check: fail if below 99%
  - `go build ./cmd/...` (build verification)
  - `govulncheck ./...` (dependency vulnerabilities)
- [ ] Same hooks path as pre-commit

### S0.7 - GitHub Actions CI

- [ ] `.github/workflows/gate.yml`:
  ```yaml
  # Triggered on every push to main
  # Matrix: single Go version (latest stable)
  # Steps:
  #   1. Checkout
  #   2. Setup Go
  #   3. Lint (golangci-lint)
  #   4. Arch tests
  #   5. Unit tests + coverage report
  #   6. Coverage threshold check (99%)
  #   7. Mutation testing + score check (85%)
  #   8. Security scan (govulncheck)
  #   9. Secret detection (gitleaks)
  #  10. License check
  #  11. Build all binaries
  ```
- [ ] Badge in README: build status
- [ ] Fail-fast: first failure stops the pipeline

### S0.8 - Claude Code rules and hooks

- [ ] `.claude/rules/testing.md`:
  - ALWAYS write a failing test before implementing any feature
  - ALWAYS write a test that reproduces a bug before fixing it
  - Choose the right test level: unit for domain, integration for adapters, acceptance for behavior
  - Never mock what you don't own
  - Use in-memory adapters for fast tests, real adapters for integration tests
  - Table-driven tests for any function with >2 cases
- [ ] `.claude/rules/architecture.md`:
  - Domain layer: ZERO external dependencies. Pure Go only.
  - Ports: interfaces only. Depend on domain only.
  - App services: depend on domain and ports. NEVER on adapters.
  - Adapters: implement ports. Only place for external libraries.
  - Every new package must respect hex boundaries (arch tests will catch violations)
- [ ] `.claude/rules/cd.md`:
  - Every commit must be deployable
  - No feature branches. Trunk only.
  - No TODO comments without a linked tracking item
  - No commented-out code
  - No println/fmt.Println debugging left in code
  - No `//nolint` without a justification comment
- [ ] `.claude/rules/quality.md`:
  - Run `make all` before committing
  - Coverage must stay at 99%+
  - Zero linting warnings
  - Zero security findings
- [ ] Claude Code hooks (`settings.json` or project-level):
  - Post-write: lint changed files
  - Post-commit: run full test suite

### S0.9 - BDD framework setup

- [ ] Add `github.com/cucumber/godog` dependency
- [ ] `test/acceptance/features/.gitkeep` (ready for first feature files in W1)
- [ ] `test/acceptance/suite_test.go` (Godog test runner, wired to `go test`)
- [ ] Verify: `make test-acc` runs (0 scenarios, 0 steps, green)

### S0.10 - Mutation testing + security + licenses

- [ ] Add `github.com/zimmski/go-mutesting` (or evaluate `gremlins`)
- [ ] `make mutation` runs and reports score
- [ ] `govulncheck` installed and wired into `make security`
- [ ] `gitleaks` installed and wired into `make secrets` + pre-commit
- [ ] `go-licenses` installed and wired into `make licenses`
- [ ] Verify: all `make` targets run green on empty skeleton

---

## Definition of Done for Sprint 0

- [ ] `make all` runs green (lint + test + coverage + security + arch + build)
- [ ] `make gate` runs green (same, the canonical quality gate)
- [ ] Pre-commit hook blocks a bad import (tested manually)
- [ ] Pre-commit hook blocks a committed secret (tested manually)
- [ ] Pre-push hook blocks if coverage < 99% (tested manually)
- [ ] GitHub Actions pipeline runs green on push
- [ ] Arch tests fail if `domain/` imports `adapter/` (red-green validated)
- [ ] Claude Code rules are in `.claude/rules/` (4 files)
- [ ] Zero feature code. Only quality infrastructure.
- [ ] Committed and pushed to main

---

## Decisions

- Arch tests are the FIRST tests. Quality infra before feature code.
- Coverage starts at 99% from commit 0. Easier to maintain than to retrofit.
- Pre-commit + pre-push + CI = 3 walls. Redundant by design.
- Claude Code rules encode the same standards the pipeline enforces.
  Agents get fast feedback; pipeline is the final authority.
- Mutation testing threshold starts at 85%. Ratchet up over time.
- `make gate` is the single command that answers: "is this commit shippable?"
