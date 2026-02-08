# Extreme Vibe Coding

> Vibe Coding with all dials turned to 11.
> All the flow. All the rigor. Zero compromise.

---

## The Problem with Vibe Coding

Vibe Coding unlocked something real: **flow state programming through natural language**.
You describe intent, the AI writes code, you ship fast. It feels effortless.

But it earned a reputation. "Vibe Coding" became synonymous with:
- No tests
- No architecture
- No review
- No safety nets
- Ship and wing it

The industry created a false dichotomy:
**either** you vibe and move fast with no guardrails,
**or** you follow best practices and move slow with heavy process.

## The Thesis

**What if the guardrails were the vibe?**

Extreme Vibe Coding (XVC) is Vibe Coding fused with every best practice from
XP, Continuous Delivery, Software Craftsmanship, and DevOps --
all automated, all enforced, all the time.

You still describe intent in natural language.
AI agents still write the code.
But **the machine enforces the discipline** so the human stays in flow.

The developer never leaves the vibe. The system never drops its standards.

```
Traditional Vibe Coding:   Intent -> Code -> Ship -> Wing it
Extreme Vibe Coding:       Intent -> Code -> Wall of Automated Discipline -> Ship -> Confidence
```

The key insight: **rigor does not require human effort if the toolchain enforces it**.

---

## Principles

### 1. The Human Vibes, the Machine Enforces

The developer stays in creative flow: describing features, thinking about users,
exploring ideas. Every engineering practice is automated or agent-enforced.
No manual checklists. No "remember to run the tests."
The pipeline is the discipline.

### 2. Code First, Always

Nothing exists without code. No design docs that rot.
No architecture diagrams that diverge. The code is the source of truth.
Tests are written before implementation -- enforced by hooks,
not by willpower.

### 3. Every Commit is a Candidate for Production

Trunk-based development. No long-lived branches. No PRs that sit for days.
Every commit passes through a gate of automated quality checks before integration.
If it passes, it ships. If it doesn't, it's rejected instantly.

### 4. Coverage is Not Optional, It's Structural

99% code coverage threshold. Not as a vanity metric, but as a structural guarantee.
Mutation testing validates that tests actually catch bugs,
not just execute lines. Dead code is detected and rejected.

### 5. Continuous Everything

Continuous integration. Continuous delivery. Continuous review.
Continuous security scanning. Continuous architecture validation.
Continuous feedback. The word "batch" is not in the vocabulary.

### 6. Agents Review Agents

AI agents write code. Other AI agents review it. Continuously.
Not at PR time -- there are no PRs. Review happens on every commit,
asynchronously, in parallel. Findings feed back into the next cycle.

---

## The Stack

### Pre-Commit Hooks: The First Wall

Before a commit even exists locally, the pre-commit hook runs:

| Check                      | Tool / Approach                           | Purpose                                |
|----------------------------|-------------------------------------------|----------------------------------------|
| Formatting                 | `gofmt` / `prettier` / language formatter | Zero formatting debates                |
| Linting                    | `golangci-lint` / `eslint` / equivalent   | Catch code smells instantly            |
| Static analysis            | NilAway / Semgrep / language-specific     | Catch bugs at compile time             |
| Secret detection           | `gitleaks` / `trufflehog`                 | Never commit credentials               |
| Commit message format      | Conventional Commits enforcer             | Machine-readable history               |
| File size limits           | Custom hook                               | No accidental binary commits           |
| Architecture boundary test | `go-arctest` / ArchUnit / equivalent      | Hexagonal boundaries enforced          |

If any check fails, the commit is rejected. The developer fixes and re-commits.
No `--no-verify`. Ever.

### Pre-Push Hooks: The Second Wall

Before code leaves the developer's machine:

| Check                  | Tool / Approach                     | Purpose                                    |
|------------------------|-------------------------------------|--------------------------------------------|
| Full unit test suite   | `go test` / `pytest` / equivalent   | No broken tests leave the machine          |
| Coverage threshold     | 99% minimum, enforced               | Structural coverage guarantee              |
| Integration tests      | Tagged test suite                   | Adapter contracts verified                 |
| Acceptance tests       | BDD / Gherkin / Godog               | User-facing behavior verified              |
| Build verification     | Compile all targets                 | Code compiles cleanly                      |
| Dependency audit       | `govulncheck` / `npm audit`        | No known vulnerabilities in dependencies   |
| License compliance     | License checker                     | No license contamination                   |

### Claude Code Rules: The AI Discipline

`.claude/rules/` files encode the engineering standards directly
into the AI agent's behavior:

```markdown
# .claude/rules/testing.md
- ALWAYS write a failing test before implementing any feature
- ALWAYS write a test that reproduces a bug before fixing it
- Choose the right test level: unit for logic, integration for adapters, acceptance for behavior
- Never mock what you don't own
- Use in-memory adapters for fast tests, real adapters for integration tests
```

```markdown
# .claude/rules/architecture.md
- Domain layer: ZERO external dependencies. Pure language only.
- Ports: interfaces only. Depend on domain only.
- Application services: depend on domain and ports. NEVER on adapters.
- Adapters: implement ports. This is the only place for external libraries.
- Every new package must respect hexagonal boundaries or the arch tests will fail.
```

```markdown
# .claude/rules/cd.md
- Every commit must be deployable
- No feature branches. Trunk only.
- No TODO comments without a linked tracking item
- No commented-out code
- No println/console.log debugging left in code
```

### Claude Code Hooks: Automated Enforcement

Hooks trigger automatically on specific Claude Code events:

```json
{
  "hooks": {
    "postToolUse": [
      {
        "tool": "Write",
        "command": "run-linter-on-changed-files.sh"
      }
    ],
    "postCommit": [
      {
        "command": "run-full-test-suite.sh"
      },
      {
        "command": "check-coverage-threshold.sh --min 99"
      }
    ]
  }
}
```

Every file the AI writes gets immediately linted.
Every commit triggers the full test suite.
Coverage drops below 99%? The commit is flagged.

### Gated Commits: The Server Wall

When code is pushed, it enters a gated pipeline:

```
Push -> FIFO Queue -> Auto-Merge Check -> Pipeline Execution -> Accept/Reject
                                              |
                          +-------------------+-------------------+
                          |         |         |         |         |
                        Lint      Test     Build    Security   Arch
                                  |                    |
                              Coverage              SAST/DAST
                                  |                    |
                             Mutation            Dependency
                              Testing              Audit
```

**All gates must pass.** No exceptions. No manual overrides.
A rejected commit means: fix, re-commit, re-push. The pipeline is the authority.

### Continuous AI Review

Dedicated AI agents perform continuous code review:

| Agent Role              | Trigger                | Focus                                          |
|-------------------------|------------------------|-------------------------------------------------|
| Craft Coach             | Every commit           | Code quality, naming, SOLID, design patterns   |
| Security Reviewer       | Every commit           | OWASP Top 10, injection, auth flaws            |
| Architecture Guardian   | Every commit           | Dependency direction, layer violations         |
| Test Quality Auditor    | Every commit           | Test meaningfulness, mutation score            |
| Performance Sentinel    | Every commit           | Algorithmic complexity, resource leaks         |
| Cross-Cutting Concerned | On request / scheduled | Logging, error handling, observability         |

These agents don't block -- they produce findings that are tracked and addressed.
They run in parallel, asynchronously. The developer never waits for review.

---

## The Testing Pyramid on Steroids

```
                /  Chaos / Resilience  \         <- Periodic: fault injection
               /    E2E / Smoke        \         <- Few: real binary, real server
              /     Contract Tests      \        <- Medium: API contracts between services
             /    Acceptance (BDD)       \       <- Medium: Gherkin scenarios, user behavior
            /    Integration Tests        \      <- Medium: real adapters, real I/O
           /      Unit Tests (fast)        \     <- Many: pure domain + app logic
          /    Property-Based Tests         \    <- Many: generated inputs, invariant checks
         /     Mutation Testing              \   <- Always: validates test effectiveness
        /    Architectural Tests              \  <- Always: enforced structural rules
       /      Static Analysis                  \ <- Always: compile-time bug detection
```

| Testing Practice           | Threshold / Rule                                      |
|----------------------------|-------------------------------------------------------|
| Code coverage              | 99% minimum, enforced in pipeline                     |
| Mutation score             | 85%+ (tests must actually catch injected bugs)        |
| Acceptance test coverage   | Every user story has at least one Gherkin scenario     |
| Architecture compliance    | Zero violations allowed                                |
| Security scan              | Zero high/critical findings allowed                    |
| Dependency vulnerabilities | Zero known vulnerabilities in production dependencies  |
| Contract tests             | All API contracts verified on every commit              |
| Performance benchmarks     | No regression beyond 10% tolerance                     |

### Migration Testing

Database migrations, config changes, API version bumps -- all tested:

- **Forward migration**: apply and verify
- **Backward migration**: rollback and verify
- **Data integrity**: round-trip test (migrate up, verify data, migrate down, verify data)
- **Zero-downtime verification**: simulate rolling deployment during migration

---

## The Feedback Loop

The core metric: **time from intent to production**.

```
Developer describes intent
         |
         v  (seconds)
   AI writes code + tests
         |
         v  (seconds)
   Pre-commit hooks pass
         |
         v  (seconds)
   Pre-push hooks pass
         |
         v  (seconds to minutes)
   Gated pipeline passes
         |
         v  (seconds)
   Integrated into trunk
         |
         v  (minutes)
   Deployed to production
         |
         v  (continuous)
   Observability confirms health
```

**Target: intent to production in under 15 minutes.**

If any step takes too long, it's a smell. Optimize it.
If any step requires manual intervention, automate it.
If any step can be parallelized, parallelize it.

---

## XP Practices, Agentic Edition

| XP Practice              | Extreme Vibe Coding Equivalent                                     |
|--------------------------|---------------------------------------------------------------------|
| Pair Programming         | Human + AI agent, always. Human vibes, agent enforces.             |
| TDD                      | AI writes failing test first, enforced by rules + hooks.           |
| Continuous Integration   | Gated commits. Every push integrates or is rejected.               |
| Refactoring              | AI agents suggest refactoring continuously. Never degrade.         |
| Simple Design            | Rules enforce: no over-engineering, YAGNI, minimal complexity.     |
| Collective Ownership     | Agents work on any part of the code. No silos.                     |
| Coding Standards         | Enforced by linters, formatters, and AI rules. Not by debate.      |
| Sustainable Pace         | Agents don't burn out. Humans stay in flow. No crunch.             |
| Small Releases           | Every commit is a release candidate. Deployment is continuous.     |
| Customer Tests           | BDD scenarios written with stakeholders, verified on every commit. |
| Metaphor                 | Ubiquitous language enforced in code, tests, and AI rules.         |

---

## What This Is NOT

- **Not "move fast and break things."** Nothing breaks. The pipeline catches everything.
- **Not "let the AI do whatever."** The AI operates within strict, enforced boundaries.
- **Not "no human involvement."** The human drives intent, design, and decisions.
  The human is the architect. The agents are the builders and inspectors.
- **Not "testing theater."** Mutation testing ensures tests are meaningful,
  not just green checkmarks.
- **Not "process heavy."** The process is invisible. It's encoded in automation.
  The developer experiences flow, not bureaucracy.

---

## The Manifesto

```
We value:

  Flow state                   over ceremony
  Automated enforcement        over manual discipline
  Instant feedback             over batched review
  Trunk-based delivery         over branch-based isolation
  Meaningful tests             over coverage theater
  Structural guarantees        over wishful thinking
  Continuous improvement       over periodic retrospectives
  Human intent + machine rigor over either alone
```

**Extreme Vibe Coding: all the vibes, all the discipline, all the time.**

---

## Getting Started

To adopt Extreme Vibe Coding on a project:

1. **Set up pre-commit hooks**: formatting, linting, secret detection, arch tests
2. **Set up pre-push hooks**: full test suite, coverage threshold, security scan
3. **Configure AI rules**: testing discipline, architecture boundaries, CD practices
4. **Configure AI hooks**: auto-lint on write, auto-test on commit
5. **Set up gated pipeline**: every push goes through automated quality gates
6. **Enable continuous AI review**: agents review every commit asynchronously
7. **Set coverage threshold to 99%**: with mutation testing to validate test quality
8. **Adopt trunk-based development**: no branches, no PRs, gated commits only
9. **Measure feedback loop time**: intent to production, optimize relentlessly
10. **Iterate**: add more checks, tighten thresholds, improve agent rules

The goal is not perfection on day one.
The goal is a ratchet that only moves in one direction: **better**.
