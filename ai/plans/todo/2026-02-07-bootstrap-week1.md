# Plan: Bootstrap GoGo - Week 1

**Date**: 2026-02-07
**Goal**: Full project setup, VCS domain, ports, arch tests, CI
**Master plan**: `2026-02-07-master-plan-bootstrap-3months.md`

## Steps

- [x] Create plan files
- [ ] `go mod init github.com/DevOpsCraftsman/gogo`
- [ ] Directory structure (hexagonal)
- [ ] Makefile
- [ ] `.golangci.yaml`
- [ ] GitHub Actions (lint + test)
- [ ] Domain VCS: Commit, CommitID, CommitState, Author, FileChange, Object, Tree, Blob, Diff
- [ ] Domain Pipeline: Pipeline, Step, PipelineRun, StepResult (skeleton)
- [ ] Domain Gate: GateEvaluator, Queue, Decision (skeleton)
- [ ] Port: VCSBackend interface (1st draft)
- [ ] Port: other interfaces (skeleton)
- [ ] Arch tests: hex boundaries
- [ ] Memory adapter: VCSBackend in-memory (skeleton)
- [ ] cmd/gg/main.go + cmd/gogod/main.go (skeletons)
- [ ] Config package
- [ ] git init + first commit

## Decisions
- Bootstrap the entire W1 skeleton in one go
- Arch tests validate hexagonal boundaries from the start
- Pure domain: zero external imports
