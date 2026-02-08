# GoGo - Bootstrap Plan (3 months)

## Context

GoGo is a **VCS + CD pipeline** in Go, without branches or PRs, with native **gated commits**.
The project is a challenge: prove that with Go (ultra-fast compilation) + strict CD + solid tests,
we can industrialize software development with maximally fast feedback loops.

Named after "Go" the CI server by Jez Humble (co-author of Continuous Delivery), but written in Go.
CLI: `gg`. Server: `gogod`.

---

## Technical Decisions

| Decision                | Choice                                                               |
|-------------------------|----------------------------------------------------------------------|
| VCS backend             | Abstraction (git via go-git initially, re-implementable later)       |
| Gated commit            | FIFO queue + ephemeral auto-merge if trunk moved                     |
| Pipeline format         | YAML in `.gogo/pipeline.yaml`                                       |
| GoGo config             | YAML (everything in YAML, single format)                            |
| Server                  | Local first (localhost)                                              |
| Gate steps              | User-configurable (gate vs post-merge)                              |
| Web UI                  | SSR + htmx, minimal: pipeline status green/red, logs                |
| Docker                  | Optional per step (default: shell on host)                          |
| GoGo tests              | Full CD pipeline + Dogfooding                                       |
| Architecture            | Hexagonal from the start                                            |
| CLI                     | Ultra minimal: `gg init / commit / push / status / log`             |
| Rejected commit         | Marked "rejected" locally, client must sync + rebase                |
| Server storage          | Pure filesystem (JSON + text files)                                 |
| Code hosting            | GitHub (`github.com/DevOpsCraftsman/gogo`) + GoGo mirror when stable|
| Go module               | `github.com/DevOpsCraftsman/gogo`                                   |

---

## Gated Commit Flow (FIFO)

```
Client                          Server (gogod)
  |                                |
  | gg commit                      |
  | (auto-add + local commit)      |
  |                                |
  | gg push ---------------------->| 1. Commit received, added to FIFO queue
  |                                |
  |                                | 2. Dequeue next commit
  |                                |
  |                                | 3. Has trunk moved since parent?
  |                                |    NO  -> direct pipeline
  |                                |    YES -> ephemeral auto-merge attempted
  |                                |           FAIL -> reject "merge conflict"
  |                                |           OK   -> pipeline on merged result
  |                                |
  |                                | 4. Pipeline pass -> integrated into trunk
  |                                |    Pipeline fail -> rejected
  |                                |    (if auto-merge was done:
  |                                |     info "post-merge failure, sync required")
  |                                |
  | <--- status update ----------- | 5. Client notified
  |                                |
  | If rejected:                   |
  | gg sync (fetch trunk + rebase) |
  | fix + gg commit + gg push      |
```

**Client-side rebase** (without native git rebase):
1. Extract the diff of the rejected commit vs its parent
2. Reset the working tree to the new trunk tip
3. Re-apply the changes as uncommitted modifications
4. Developer fixes, re-commits, re-pushes

---

## Hexagonal Architecture - Package Structure

```
gogo/
├── cmd/
│   ├── gg/                          # CLI binary
│   │   └── main.go                  # Wire adapters + start CLI
│   └── gogod/                       # Server daemon binary
│       └── main.go                  # Wire adapters + start HTTP server
│
├── internal/
│   ├── domain/                      # PURE DOMAIN - zero external deps
│   │   ├── vcs/
│   │   │   ├── commit.go            # Commit, CommitID, CommitState, Author
│   │   │   ├── object.go            # Object, ObjectID, Tree, Blob
│   │   │   ├── diff.go              # Diff, FileChange, ChangeType
│   │   │   ├── status.go            # WorkingTreeStatus
│   │   │   └── errors.go            # ErrNoCommits, ErrCleanWorkingTree...
│   │   │
│   │   ├── pipeline/
│   │   │   ├── pipeline.go          # Pipeline aggregate (steps)
│   │   │   ├── step.go              # Step (name, command, docker, phase, timeout)
│   │   │   ├── run.go               # PipelineRun (steps + results + status)
│   │   │   └── result.go            # StepResult (status, exit code, output, duration)
│   │   │
│   │   └── gate/
│   │       ├── evaluator.go         # Pure logic: step results -> accept/reject
│   │       ├── queue.go             # FIFO queue domain logic
│   │       └── decision.go          # GateDecision (accepted/rejected + reason)
│   │
│   ├── app/                         # APPLICATION SERVICES (use cases)
│   │   ├── commit_service.go        # Orchestrates: create commit, store objects
│   │   ├── push_service.go          # Orchestrates: send commit to server
│   │   ├── sync_service.go          # Orchestrates: fetch trunk, rebase local
│   │   ├── status_service.go        # Orchestrates: working tree + pending/rejected
│   │   ├── log_service.go           # Orchestrates: commit history
│   │   ├── pipeline_service.go      # Orchestrates: load config, execute steps
│   │   └── gate_service.go          # Orchestrates: FIFO queue + auto-merge + pipeline + accept/reject
│   │
│   ├── port/                        # PORTS (Go interfaces)
│   │   ├── vcs_backend.go           # VCSBackend (init, stage, commit, diff, refs...)
│   │   ├── commit_state_store.go    # CommitStateStore (pending/rejected/integrated metadata)
│   │   ├── pipeline_config.go       # PipelineConfigLoader (load .gogo/pipeline.yaml)
│   │   ├── step_executor.go         # StepExecutor (run shell or docker step)
│   │   ├── commit_transport.go      # CommitTransport (client->server push)
│   │   ├── trunk_manager.go         # TrunkManager (merge to trunk, conflict check)
│   │   └── event_publisher.go       # EventPublisher (pipeline events for UI)
│   │
│   ├── adapter/                     # ADAPTERS (port implementations)
│   │   ├── cli/                     # PRIMARY: CLI (cobra)
│   │   │   ├── root.go
│   │   │   ├── init.go              # gg init
│   │   │   ├── commit.go            # gg commit
│   │   │   ├── push.go              # gg push
│   │   │   ├── status.go            # gg status
│   │   │   └── log.go               # gg log
│   │   │
│   │   ├── http/                    # HTTP API (server + client)
│   │   │   ├── server.go            # chi router + middleware
│   │   │   ├── handler_push.go      # POST /api/push
│   │   │   ├── handler_status.go    # GET  /api/status
│   │   │   ├── handler_log.go       # GET  /api/log
│   │   │   ├── handler_pipeline.go  # GET  /api/pipeline/:id
│   │   │   └── client.go            # HTTP client (implements CommitTransport)
│   │   │
│   │   ├── web/                     # Web UI (SSR + htmx)
│   │   │   ├── handler.go           # Web routes
│   │   │   ├── templates/           # templ components (.templ)
│   │   │   │   ├── layout.templ
│   │   │   │   ├── dashboard.templ
│   │   │   │   └── pipeline_run.templ
│   │   │   └── static/              # CSS, htmx.min.js
│   │   │
│   │   ├── git/                     # Git backend (implements VCSBackend)
│   │   │   ├── adapter.go           # go-git implementation
│   │   │   ├── mapper.go            # domain objects <-> git objects
│   │   │   └── merge.go             # Auto-merge (three-way via go-git trees)
│   │   │
│   │   ├── fs/                      # Filesystem storage
│   │   │   ├── config_loader.go     # Reads .gogo/pipeline.yaml
│   │   │   ├── state_store.go       # JSON in .gogo/state/
│   │   │   ├── log_store.go         # Pipeline logs as text files
│   │   │   └── queue_store.go       # FIFO queue persisted as JSON
│   │   │
│   │   ├── exec/                    # Step execution
│   │   │   ├── shell.go             # Shell executor
│   │   │   ├── docker.go            # Docker executor
│   │   │   └── factory.go           # Chooses shell vs docker
│   │   │
│   │   └── memory/                  # In-memory (for tests)
│   │       ├── vcs_backend.go
│   │       ├── state_store.go
│   │       └── step_executor.go
│   │
│   └── config/
│       └── config.go                # Config loading (env, flags, defaults)
│
├── test/
│   ├── acceptance/                  # BDD (Godog + Gherkin)
│   │   ├── features/
│   │   │   ├── init.feature
│   │   │   ├── commit.feature
│   │   │   ├── push_gated.feature
│   │   │   ├── pipeline_execution.feature
│   │   │   └── conflict_rejection.feature
│   │   └── steps/
│   │       ├── init_steps.go
│   │       ├── commit_steps.go
│   │       └── push_steps.go
│   │
│   └── arch/                        # Architectural tests
│       └── hex_boundaries_test.go   # Domain must not import adapter, etc.
│
├── .gogo/                           # GoGo project config (versioned)
│   └── pipeline.yaml
│
├── Makefile
├── go.mod
├── go.sum
├── .golangci.yaml
├── Dockerfile
├── docker-compose.yaml
└── README.md
```

### Strict Dependency Rule

```
adapter -> app -> port <- domain
          app -> domain
```

- `domain/`: zero external imports. Pure Go only.
- `port/`: interfaces only. Depends on `domain/` only.
- `app/`: depends on `domain/` and `port/`. Never on `adapter/`.
- `adapter/`: implements `port/` interfaces. Depends on everything.
- Enforced by architectural tests in `test/arch/`.

---

## Commit State Machine

```
                  gg commit
  [nothing] ──────────────────> LOCAL
                                  │
                                  │ gg push
                                  ▼
                               PENDING  (in the server FIFO queue)
                              /        \
            pipeline passes  /          \  pipeline fails
            + merge OK      /            \  OR merge conflict
                           ▼              ▼
                     INTEGRATED        REJECTED
                                   (reason + auto-merge info)
```

| Transition             | Trigger                                      | Actor   |
|------------------------|----------------------------------------------|---------|
| (none) -> Local        | `gg commit`                                  | Client  |
| Local -> Pending       | `gg push`                                    | Client  |
| Pending -> Integrated  | Pipeline pass (+ auto-merge OK if needed)    | Server  |
| Pending -> Rejected    | Pipeline fail OR auto-merge fails            | Server  |

**Rejected is terminal.** The user creates a NEW commit after sync.

---

## Pipeline YAML Schema

```yaml
# .gogo/pipeline.yaml
version: 1

defaults:
  timeout: 5m
  environment:
    GOGO: "true"
    CI: "true"

steps:
  - name: lint
    phase: gate              # gate = must pass BEFORE integration
    command: golangci-lint run ./...
    timeout: 3m

  - name: test
    phase: gate
    command: go test -race -count=1 ./...
    timeout: 10m

  - name: build
    phase: gate
    command: go build -o /dev/null ./cmd/...

  - name: integration-test
    phase: gate
    command: go test -tags=integration ./test/...
    docker:                  # Optional: run in a container
      image: golang:1.23
      volumes:
        - ./:/workspace
      workdir: /workspace
    timeout: 15m

  - name: deploy-staging
    phase: post-merge        # post-merge = after integration
    command: ./scripts/deploy.sh staging
```

### Variables Injected by GoGo

| Variable               | Description                                  |
|------------------------|----------------------------------------------|
| `GOGO`                 | Always "true"                                |
| `CI`                   | Always "true"                                |
| `GOGO_COMMIT_ID`       | Hash of the commit being tested              |
| `GOGO_COMMIT_MESSAGE`  | Commit message                               |
| `GOGO_COMMIT_AUTHOR`   | Author name                                  |
| `GOGO_STEP_NAME`       | Name of the current step                     |
| `GOGO_RUN_ID`          | Unique pipeline run ID                       |
| `GOGO_AUTO_MERGED`     | "true" if an auto-merge was performed        |

---

## Filesystem Layout

### Client (.gogo/)

```
my-project/
├── .git/                        # Git internals (managed by go-git)
├── .gogo/                       # GoGo metadata
│   ├── config.yaml              # Client config (remote URL, user)
│   ├── pipeline.yaml            # Pipeline definition (versioned)
│   └── state/                   # Local state (in .gitignore)
│       └── commits/
│           ├── <id>.json        # {"state":"pending","pushed_at":"..."}
│           └── <id>.json        # {"state":"rejected","reason":"..."}
└── src/...                      # Project files
```

### Server

```
/srv/gogo/repos/<repo>/
├── git/                         # Bare git repo
│   └── refs/heads/trunk         # THE ONLY ref: trunk
├── .gogo/
│   ├── config.yaml
│   ├── state/commits/<id>.json  # Commit states (source of truth)
│   ├── queue/pending.json       # FIFO queue
│   ├── pipeline/runs/<id>.json  # Pipeline results
│   └── locks/integration.lock   # File lock to serialize integration
```

---

## 3-Month Roadmap

### Sprint 0: Quality Pipeline (before any feature code)

**Goal**: Full Extreme Vibe Coding infrastructure in place. Zero feature code, 100% quality tooling.
**Details**: `2026-02-08-sprint0-quality-pipeline.md`
**Manifesto**: `../../EXTREME-VIBE-CODING.md`

| Step | Deliverables                                                                    |
|------|---------------------------------------------------------------------------------|
| S0.1 | `go mod init`, directory structure (hexagonal), `go.mod`                        |
| S0.2 | Arch tests: hex boundary enforcement (first tests in the repo)                  |
| S0.3 | `.golangci.yaml` (strict, all dials to 11), NilAway                            |
| S0.4 | Makefile: `lint`, `test`, `coverage`, `mutation`, `security`, `arch`, `all`     |
| S0.5 | Pre-commit hooks: formatting, linting, secret detection, arch tests             |
| S0.6 | Pre-push hooks: full test suite, coverage 99%, security scan                    |
| S0.7 | GitHub Actions: full gated pipeline (lint + test + coverage + security + arch)  |
| S0.8 | Claude Code rules (`.claude/rules/`) + hooks (`.claude/settings.json`)          |
| S0.9 | BDD framework setup (Godog + first empty feature file)                          |
| S0.10| Mutation testing, `govulncheck`, `gitleaks`, license check                      |

### Month 1: VCS Core + CLI (Weeks 1-4)

**Goal**: `gg init && gg commit -m "first" && gg log && gg status` works locally.

| Week | Deliverables                                                                    |
|------|---------------------------------------------------------------------------------|
| W1   | Domain: Commit, CommitID, CommitState, Author, FileChange                       |
|      | Port: VCSBackend interface (1st draft)                                          |
|      | Memory adapter: in-memory VCSBackend (for fast unit tests from day 0)           |
|      | Unit tests for domain (table-driven, pure Go)                                  |
| W2   | Git adapter: go-git implements VCSBackend (Init, StageAll, CreateCommit)        |
|      | `gg init`: creates `.gogo/` + init git backend                                 |
|      | `gg commit`: stage all + commit                                                |
|      | Acceptance test: "Given empty dir, when gg init, then .gogo/ exists"           |
| W3   | `gg status`: working tree changes + commits pending/rejected                    |
|      | `gg log`: trunk history                                                        |
|      | Acceptance tests: commit, status, log                                          |
| W4   | Error handling, edge cases (empty repo, nothing to commit)                     |
|      | CLI polish: colors, help text                                                  |
|      | **Release v0.1.0-alpha**                                                       |

### Month 2: Pipeline Engine + Gated Commits + Server (Weeks 5-8)

**Goal**: `gg push` triggers a pipeline on the server. Accept/reject works.

| Week | Deliverables                                                                    |
|------|---------------------------------------------------------------------------------|
| W5   | Domain: Pipeline, Step, PipelineRun, StepResult                                |
|      | Port: PipelineConfigLoader, StepExecutor                                       |
|      | Adapter: YAML config loader                                                    |
|      | Adapter: Shell executor (command + capture output/exit code)                   |
| W6   | App: PipelineService (load config, execute steps, collect results)             |
|      | Domain: GateEvaluator (pure accept/reject logic)                               |
|      | App: GateService (FIFO queue + auto-merge + pipeline + decision)               |
|      | Adapter: Docker executor (via Docker SDK)                                      |
| W7   | HTTP server: chi router, POST /api/push, GET /api/status                       |
|      | `gg push`: sends commit to server, server triggers gated pipeline              |
|      | Port: CommitTransport. Adapter: HTTP client                                    |
|      | FIFO queue: auto-merge if trunk moved, conflict = rejection                    |
| W8   | `gg sync`: fetch trunk + local rebase (diff extract + reset + re-apply)        |
|      | `gg status`: shows remote status (pending/running/accepted/rejected)           |
|      | Acceptance tests: push-gate-accept, push-gate-reject, push-conflict-reject     |
|      | **Release v0.2.0-alpha**                                                       |

### Month 3: Web UI + Dogfooding + Hardening (Weeks 9-12)

**Goal**: Working web dashboard. GoGo uses GoGo.

| Week | Deliverables                                                                    |
|------|---------------------------------------------------------------------------------|
| W9   | Web UI: templ + htmx. Dashboard: recent commits + status (green/red/pending)   |
|      | Pipeline run detail page: step-by-step with logs, duration, status             |
| W10  | Real-time: htmx polling for live pipeline progress                             |
|      | Dogfooding: `.gogo/pipeline.yaml` for GoGo itself                              |
|      | GoGo CI uses GoGo (locally, alongside GitHub Actions)                          |
| W11  | Docker packaging: gogod Dockerfile, docker-compose.yaml                        |
|      | Config: server address, port, log level via env vars                           |
|      | Error handling hardening, timeouts, graceful failures                          |
| W12  | Complete README, getting-started guide, pipeline.yaml reference                |
|      | Performance: benchmark gated commit cycle, optimize critical path              |
|      | **Release v0.3.0-beta**                                                        |

---

## Test Strategy (Extreme Vibe Coding)

### Pyramid

```
              /  Chaos / Resilience  \     <- Periodic: fault injection
             /      E2E / Smoke       \    <- Few: gg binary against gogod
            /     Acceptance (BDD)     \   <- Medium: Godog/Gherkin scenarios
           /    Integration Tests       \  <- Medium: adapters (git, Docker, HTTP)
          /      Unit Tests (fast)       \ <- Many: domain + app (pure Go)
         /   Property-Based Tests         \<- Many: generated inputs, invariants
        /     Mutation Testing             \<- Always: validates test effectiveness
       /    Architectural Tests             \<- Always: hex boundary enforcement
      /      Static Analysis                 \<- Always: linters, NilAway, govulncheck
```

### Thresholds (enforced in pipeline, no exceptions)

| Practice                   | Threshold                                              |
|----------------------------|--------------------------------------------------------|
| Code coverage              | 99% minimum                                            |
| Mutation score             | 85%+ (tests must catch injected bugs)                  |
| Architecture compliance    | Zero violations                                        |
| Security scan              | Zero high/critical findings                            |
| Dependency vulnerabilities | Zero known vulns in production deps                    |
| Linting                    | Zero warnings (strict mode)                            |

### Unit Tests (domain + app)

- `*_test.go` files next to source code
- Domain: pure tests, zero mocks (no dependencies)
- App: uses in-memory adapters
- Table-driven tests for GateEvaluator

### Acceptance Tests (BDD - Godog/Gherkin)

- `test/acceptance/features/*.feature`
- Describe user-facing behavior
- Use real application services + in-memory adapters for speed
- Real shell executor (tests actual commands)

Example:
```gherkin
Feature: Gated commit via push

  Scenario: Commit accepted when all gate steps pass
    Given a GoGo repository with a pipeline:
      """
      version: 1
      steps:
        - name: check
          phase: gate
          command: "true"
      """
    And I have committed a change with message "add feature"
    When I push to the server
    Then the pipeline should run
    And the commit status should be "integrated"

  Scenario: Commit rejected when a gate step fails
    Given a GoGo repository with pipeline step "check" with command "exit 1"
    And I have committed a change
    When I push to the server
    Then the commit status should be "rejected"
```

### Architectural Tests

```go
// test/arch/hex_boundaries_test.go
// domain/ must NEVER import adapter/
// domain/ must NEVER import app/
// app/ must NEVER import adapter/
// port/ must contain ONLY interfaces
```

### Integration Tests

- Git adapter: real temporary git repo, round-trip objects
- Docker executor: real container, trivial command
- HTTP handlers: `httptest.NewServer`
- Tagged `//go:build integration`, skipped in fast CI

---

## Go Dependencies

### Core

| Lib                                   | Usage                            |
|---------------------------------------|----------------------------------|
| `github.com/go-git/go-git/v5`        | Git storage backend              |
| `github.com/go-chi/chi/v5`           | HTTP router (stdlib-compatible)  |
| `github.com/spf13/cobra`             | CLI framework                    |
| `github.com/a-h/templ`               | HTML templating SSR (type-safe)  |
| `github.com/docker/docker/client`     | Docker SDK                       |
| `gopkg.in/yaml.v3`                   | YAML parsing                     |
| `github.com/cucumber/godog`          | BDD acceptance tests             |

### Dev / Quality

| Lib / Tool                            | Usage                            |
|---------------------------------------|----------------------------------|
| `github.com/golangci/golangci-lint`   | Linter aggregator (strict)       |
| `go.uber.org/nilaway`                 | Nil safety static analysis       |
| `github.com/mstrYoda/go-arctest`      | Arch tests hex boundaries        |
| `github.com/zimmski/go-mutesting`     | Mutation testing                 |
| `golang.org/x/vuln/cmd/govulncheck`  | Dependency vulnerability scan    |
| `github.com/zricethezav/gitleaks`     | Secret detection (pre-commit)    |
| `github.com/google/go-licenses`       | License compliance check         |
| `log/slog` (stdlib)                   | Structured logging               |

### Not Using

- No ORM / database (pure filesystem)
- No DI framework (manual injection in cmd/)
- No logging framework (stdlib slog is enough)

---

## Plan Verification

### How to Test End-to-End

1. **Month 1**: `go build ./cmd/gg && ./gg init /tmp/test-repo && cd /tmp/test-repo && echo "hello" > file.txt && gg commit -m "first" && gg log && gg status`
2. **Month 2**: Create `.gogo/pipeline.yaml` with a step `echo ok`, `gg push`, verify commit is integrated. Change the step to `exit 1`, re-push, verify rejection.
3. **Month 3**: Open `http://localhost:8080`, see the dashboard with latest commit and pipeline status.

### Common Verification Commands

```bash
# Run all tests
make test

# Run acceptance tests
go test ./test/acceptance/...

# Run arch tests
go test ./test/arch/...

# Linter
golangci-lint run ./...

# Build both binaries
go build -o bin/gg ./cmd/gg
go build -o bin/gogod ./cmd/gogod
```

---

## Identified Risks

| Risk                                  | Mitigation                                             |
|---------------------------------------|--------------------------------------------------------|
| go-git has no native rebase           | Client-side: diff extract + reset + re-apply           |
| go-git merge limited (fast-forward)   | Server-side: merge via tree manipulation in go-git     |
| Godog (BDD) maturity in Go            | Fallback: acceptance tests in pure Go with helpers     |
| go-arctest maturity                   | Fallback: custom tests with `go/packages`              |
| Ambitious scope for 3 months          | Prioritize critical path: VCS -> pipeline -> gate      |
