# Plan: Bootstrap GoGo - Week 1

**Date**: 2026-02-07 (updated 2026-02-08)
**Goal**: Domain VCS, ports, memory adapter, first unit tests -- all under full quality pipeline
**Master plan**: `2026-02-07-master-plan-bootstrap-3months.md`
**Prerequisite**: `2026-02-08-sprint0-quality-pipeline.md` (must be completed first)

## Steps

### Sprint 0 (quality pipeline, no feature code)

- [x] Create plan files
- [ ] Sprint 0 -- see `2026-02-08-sprint0-quality-pipeline.md`

### Week 1 (first feature code, under full quality gates)

- [ ] Domain VCS: Commit, CommitID, CommitState, Author
- [ ] Domain VCS: FileChange, ChangeType, Diff
- [ ] Domain VCS: Object, ObjectID, Tree, Blob
- [ ] Domain VCS: WorkingTreeStatus, errors
- [ ] Unit tests for all domain types (table-driven, pure Go)
- [ ] Port: VCSBackend interface (1st draft)
- [ ] Port: CommitStateStore, PipelineConfigLoader (skeletons)
- [ ] Memory adapter: in-memory VCSBackend (for fast tests)
- [ ] Domain Pipeline: Pipeline, Step, PipelineRun, StepResult (skeleton)
- [ ] Domain Gate: GateEvaluator, Queue, Decision (skeleton)
- [ ] Unit tests for pipeline + gate domain (skeleton)
- [ ] cmd/gg/main.go + cmd/gogod/main.go (wiring skeletons)
- [ ] Config package (skeleton)
- [ ] Verify: `make gate` green, coverage >= 99%, arch tests green

## Decisions

- Sprint 0 is a hard prerequisite: no feature code before quality pipeline is live
- Arch tests validate hex boundaries from the first domain commit
- Pure domain: zero external imports
- Every domain type has unit tests from the start
- Memory adapter exists from W1 so app-level tests can run fast
