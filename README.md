# GoGo

> An experimental VCS-integrated CD pipeline with native gated commits.
> No branches. No PRs. Trunk only. All quality dials turned to 11.
> Subject to change. Deliberately a little crazy.

---

## What is GoGo?

GoGo is a version control system coupled with a continuous delivery pipeline.
No feature branches. No pull requests. Just a single trunk, protected by
an automated quality gate that never sleeps.

Every commit is submitted to the gate **before** it reaches trunk.
If it passes, it's integrated. If it fails, it's rejected. There is no middle ground.

Gated commits are enabled by default to encourage adoption, but can be opted out of.
Multiple gating strategies will be available (FIFO queue, merge queue, and more to come).

```
gg commit -m "add user authentication"
gg push
# -> Commit enters the gate queue
# -> Pipeline runs: lint, test, build, security, arch...
# -> Pass? Integrated into trunk.
# -> Fail? Rejected. Fix, re-commit, re-push.
```

CLI: **`gg`** -- Server: **`gogod`**

---

## Why GoGo Exists

### The Problem

Modern software delivery is slow where it shouldn't be.

Feature branches live for days or weeks. Pull requests wait for reviewers.
Merge conflicts accumulate. CI runs after the merge, when it's already too late.
The feedback loop between writing code and knowing if it works on trunk
is measured in hours or days, not seconds.

This is the opposite of Continuous Delivery.

### The Idea

The concept of a **gated commit pipeline** has existed for over a decade.
It was pioneered by a CI server designed around the principle that
no commit should reach the mainline unless it has been proven to work.
That server was called **Go** -- and it was built by the co-authors of
the book *Continuous Delivery*.

GoGo revisits that philosophy and pushes it further:

- **No branches.** Not even short-lived ones. Trunk is the only line of development.
- **No pull requests.** Code review is replaced by continuous automated review --
  deterministic and agentic (AI) tools, on every commit.
- **Gated commits by default.** Every push goes through an automated quality gate
  before integration. Enabled by default to encourage adoption, but can be
  opted out of -- encouragement, not coercion.
- **Multiple gating strategies.** FIFO queue, merge queue, and more to come.
  Different teams have different needs.
- **The gate is the authority.** A human cannot override a failing pipeline.
  Fix the code, not the process.
- **Instant feedback.** The pipeline runs on the commit before integration,
  not after. You know within minutes if your change is safe.

### The Experiment

GoGo is an experiment. Its primary purpose is to test the viability of
**Extreme Vibe Coding** (see below) on a real, non-trivial project --
and to share the results with the community.

Everything in this project is subject to change: the architecture, the features,
the gating strategies, even the methodology itself. The goal is to learn,
iterate, and document what works and what doesn't.

---

## Extreme Vibe Coding

GoGo is both the product and the laboratory for **Extreme Vibe Coding** (XVC) --
a development methodology that fuses the flow state of AI-assisted coding
with the full rigor of XP, Continuous Delivery, Software Craftsmanship, and DevOps.

The core idea: **the human stays in creative flow while the machine enforces
every engineering best practice, automatically, all the time.**

### What does this look like in practice?

| Practice                   | Implementation                                             |
|----------------------------|------------------------------------------------------------|
| Pre-commit hooks           | Formatting, linting, secret detection, arch tests          |
| Pre-push hooks             | Full test suite, 99% coverage, security scan               |
| Gated CI pipeline          | Lint + test + coverage + mutation + security + arch + build|
| Architectural tests        | Hexagonal boundaries enforced from commit 0                |
| Code coverage              | 99% minimum, enforced in pipeline                          |
| Mutation testing           | 85%+ score (tests must catch injected bugs)                |
| Security scanning          | Zero high/critical findings allowed                        |
| Continuous automated review| Deterministic + agentic tools on every commit              |
| Trunk-based development    | Gated commits by default, no long-lived branches           |
| BDD acceptance tests       | DSL + drivers (inspired by the LMAX testing approach)      |

### The human's role

The human author of this project will deliberately **not review code** as a quality gate.
Code review is replaced by 100% automated quality gates, both deterministic (linting,
tests, coverage, arch tests) and non-deterministic (agentic AI review).

The only manual activity is **reading acceptance tests** -- written in a DSL with
drivers, designed to be readable as living documentation. Looking at implementation
code is treated as **exploratory testing**: useful, but never a gate.

This is the experiment: can automated enforcement, with all dials turned to 11,
replace human code review as a quality gate?

The quality pipeline was set up **before the first line of feature code**
(see [Sprint 0](ai/plans/todo/2026-02-08-sprint0-quality-pipeline.md)).

Read the full manifesto: **[EXTREME-VIBE-CODING.md](EXTREME-VIBE-CODING.md)**

---

## Why Go

The choice of Go is deliberate. It serves both the product and the methodology.

### For the product

| Requirement                          | Why Go                                                     |
|--------------------------------------|------------------------------------------------------------|
| Ultra-fast compilation               | `go build` takes seconds. Fast gate = fast feedback.       |
| Single binary deployment             | No runtime, no dependencies. Ship a binary, run it.        |
| Built-in concurrency                 | Pipeline steps, queue processing, HTTP server -- all native.|
| Excellent standard library           | HTTP, JSON, filesystem, testing -- no framework needed.     |
| Strong static typing                 | Bugs caught at compile time, before the pipeline even runs. |
| Fast test execution                  | `go test` runs in seconds. Essential for 99% coverage.     |

### For Extreme Vibe Coding

Go is exceptionally well suited for AI-assisted development:

- **Strong type system** -- the compiler catches entire categories of errors
  that would otherwise require human review. The type system acts as a
  permanent, tireless reviewer.
- **Opinionated formatting** -- `gofmt` eliminates all style debates.
  No bikeshedding, not from humans, not from AI agents.
- **Fast feedback loop** -- compile + test in seconds means the AI agent
  gets instant feedback on whether its code works. Short cycles = better output.
- **Explicit error handling** -- no hidden exceptions. Every error path
  is visible and testable. AI agents handle what they can see.
- **Simple language** -- limited syntax, few abstractions, one way to do things.
  This reduces the surface for AI hallucination and makes generated code
  more predictable and reviewable by automated tools.

A CD pipeline tool must be **fast**. If the gate takes 10 minutes,
developers will resent it. If it takes 30 seconds, they'll embrace it.
Go makes the 30-second gate realistic.

---

## Why the Name

The original CI server that pioneered gated pipelines was called **Go**.
This project is a spiritual successor, written in the **Go** programming language.

Go + Go = **GoGo**.

The name was also chosen because it sounds a little crazy --
fitting for a project that sets coverage thresholds to 99%,
runs mutation testing on every commit, and replaces human code review
with AI agents. The name should tell you upfront: this project
doesn't take the safe, conventional path. It goes all in.

The CLI is `gg` -- short, fast to type, fitting for a tool obsessed with speed.
The server daemon is `gogod` -- GoGo Daemon.

---

## Architecture

GoGo follows a **hexagonal architecture** (ports and adapters) with strict dependency rules
enforced by automated architectural tests.

```
adapter -> app -> port <- domain
          app -> domain
```

- **`domain/`** -- Pure business logic. Zero external dependencies. Pure Go only.
- **`port/`** -- Interfaces only. The contracts between the application and the outside world.
- **`app/`** -- Application services (use cases). Orchestrates domain and ports.
- **`adapter/`** -- Implementations of ports. The only place for external libraries.

These boundaries are **tested on every commit**. A domain package importing an adapter
will fail the arch tests and be rejected by the pipeline.

---

## Gated Commit Flow (default)

```
Client                          Server (gogod)
  |                                |
  | gg commit                      |
  | (auto-add + local commit)      |
  |                                |
  | gg push ---------------------->| 1. Commit enters gate queue
  |                                |
  |                                | 2. Trunk moved since parent?
  |                                |    NO  -> run pipeline directly
  |                                |    YES -> auto-merge, then pipeline
  |                                |           (merge conflict -> reject)
  |                                |
  |                                | 3. Pipeline: lint, test, build, security...
  |                                |    PASS -> integrated into trunk
  |                                |    FAIL -> rejected
  |                                |
  | <--- status ------------------|
  |                                |
  | If rejected: gg sync + fix     |
```

This is the default behavior. It can be configured or disabled.
Multiple gating strategies (FIFO, merge queue, etc.) will be available.

---

## Status

GoGo is **experimental and in early development** (Sprint 0 -- quality infrastructure setup).
Everything is subject to change.

See the [roadmap](ai/plans/todo/2026-02-07-master-plan-bootstrap-3months.md) for the 3-month plan.

---

## License

See [LICENSE](LICENSE).
