# Agentic Repo Standard

A framework for how agent teams deconstruct and rebuild codebases into foundations that can be built on.

What [agentic-api-standard](https://github.com/nexus-marbell/agentic-api-standard) is for APIs, this is for repository transformation.

## The Problem

Most codebases accumulate **cognitive debt** — the invisible cost of understanding code that wasn't designed to be understood. God files, scattered concerns, implicit conventions, hardcoded secrets, missing boundaries. An agent team encountering this codebase wastes most of its capacity navigating, not building.

Incremental patches don't fix this. They add complexity on top of complexity. The result is "one giant web of monkey patches built from crisis."

## The Standard

This framework prescribes **structured deconstruction** — a phased, reversible, independently verified process for transforming a codebase from cognitive debt into a foundation.

## Exit Criterion: The Cognitive Debt Standard

> A brand new naive agent should be able to load a repo and understand it without Glob.

Filenames are the documentation. Directory structure is the architecture map. If an agent needs search tools to find things, the refactor isn't done.

**Test**: Hand the repo to a fresh agent (ideally a different model). They navigate using `ls` only. Measure comprehension. If they can identify the purpose and boundaries of each module from filenames and structure alone, the standard is met.

## The Eight Phases

### Phase 0: Govern

Establish governance **before** touching code. Branch protection, review requirements, CI pipeline. If you can't enforce review on changes, the phases that follow have no safety net.

Retrofitting governance after structural changes are in flight is painful and often blocked by tooling constraints. Do it first.

### Phase 1: Index (Diagnosis)

Map the mess before cutting. Quantify:
- God files (files with disproportionate responsibility)
- Coupling hotspots (files that change together)
- Execution flow density (functions called from too many places)
- Security surface (exposed endpoints, auth patterns, secrets handling)

Output: a **shared map**, not individual hunches. The index becomes the before-measurement for the transformation. Without metrics, "we improved it" is a feeling, not a fact.

This phase prevents the "let me just refactor this one file" impulse. You don't cut until you see the whole picture.

### Phase 2: Stop the Bleeding

Security and correctness fixes **only**. No structural changes. Hard gate.

This is where you catch SQL injection, header leaks, fabricated stats, hardcoded credentials. The temptation to restructure here is enormous — resist it. Security fixes must land on the existing structure so they're reviewable in isolation.

### Phase 3: Split

God files into cohesive modules. One PR per god file. Tests ship **with** splits, not after.

Single Responsibility enforced by filename: if you can't name the file after its single responsibility, the split isn't clean.

### Phase 4: Deduplicate

Pick canonical versions. Find the hub objects that touch disproportionate flows and distribute their responsibilities to focused modules.

Pattern: trace flow density → find the hub → distribute to spokes.

### Phase 5: Isolate

Cross-cutting concerns get their own boundaries. Database layer, auth layer, config layer — each decomposed into dedicated modules with explicit interfaces.

Context managers properly scoped. Connection management separated from business logic. Config separated from code.

### Phase 6: Standardize

Apply consistent patterns across the codebase. API compliance tiers (if applicable), naming conventions, error handling patterns, logging patterns. Environment variables replace hardcoded values.

### Phase 7: Harden

Infrastructure hardening comes **last** — you can't harden what isn't structured.

- Network segmentation (services that don't need to communicate, can't)
- Resource limits (memory, CPU per service)
- Secrets detection in CI
- Docker socket isolation (never mount directly)
- Deployment hardening (graceful shutdown, health checks, restart policies)

## Principles

### One PR Per Phase

Each phase produces one or more self-contained, reversible pull requests. Any phase can be reverted without affecting others. This is the single most important structural decision.

### Cross-Review Minimum

Two reviewers per PR. The builder does not review their own work. The reviewer does not rubber-stamp — they read the artifacts and verify against the phase's done criteria.

A self-merge during Phase 6 taught this lesson. It was caught, but the gap shouldn't have existed.

### Independent Verification

A separate agent — ideally running a different model — verifies the transformation by **being the naive agent**. They load the repo cold, navigate with basic tools, and assess whether the cognitive debt standard is met.

The builder can't be the verifier. Different agents have different blind spots. The verifier catches what the builder has normalized.

### Before/After Measurement

Non-negotiable. Index before the transformation (Phase 1). Index after Phase 7. Without quantitative comparison, improvement claims are unfounded.

Metrics that matter:
- Lines of code per file (max, mean, median)
- God file count (files > 500 LOC with > 3 concerns)
- Import coupling (files importing the same module)
- Secrets in tracked files (count → 0)
- CI pipeline coverage (syntax → integration → security)

### Environment Impact Assessment

Before each phase merge, assess: **what does this change for each deployment target?** Security hardening (Phase 7) removing port exposure is correct for production but breaks staging environments that proxy through localhost. This must be caught before merge, not after.

## Deployment Chain

The transformation isn't done when code merges. It's done when it runs.

1. **Staging environment exists before refactor starts** — not after
2. **Validate after every phase merge** — health endpoints, integration tests, log review
3. **Security sweep is a gate, not a step** — it can reject phases and send them back
4. **Production deploy is separate from merge** — code is deployed when validated, not when merged
5. **Rollback plan per phase** — what to revert if production breaks after deploy

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Incremental patches | Adds complexity on top of complexity. Patches on patches. |
| Self-review | Builder normalizes their own decisions. Blind spots compound. |
| Governance as afterthought | Can't enforce review after structural changes are in flight. |
| "Let me just refactor this one file" | Without the index, you don't know what you're breaking. |
| Merge without cross-review | Every uncaught issue in a structural change propagates to all subsequent phases. |
| Deploy without staging | Merging is not deploying. Code that passes CI can still break in production. |
| Verifier arrives mid-process | Independent verification should start at Phase 1, not Phase 7. |
| No before/after metrics | "We improved it" without numbers is a feeling, not a fact. |

## Roles

| Role | Responsibility |
|---|---|
| **Builder** | Executes phases. Writes PRs. Owns the implementation. |
| **Reviewer** | Cross-reviews every PR. Verifies against phase done criteria. |
| **Verifier** | Independent agent (different model preferred). Tests cognitive debt standard. |
| **Orchestrator** | Coordinates phases, tracks dependencies, ensures deployment chain. |
| **Principal** | Sets direction, approves governance decisions, validates outcome. |

## Phase Done Criteria Template

Each phase should define explicit done criteria before execution begins:

```
## Phase [N]: [Name]

### Scope
- [ ] [Specific files/modules in scope]

### Done When
- [ ] [Measurable criterion 1]
- [ ] [Measurable criterion 2]
- [ ] CI passes
- [ ] Cross-review approved (2 reviewers)
- [ ] No regressions in existing functionality

### Environment Impact
- [ ] Staging: [impact assessment]
- [ ] Production: [impact assessment]

### Dependencies
- Blocked by: [Phase N-1 if sequential]
- Blocks: [Phase N+1 if sequential]
```

## Proof Case

This framework was developed and proven during a 7-phase transformation of a production API codebase:

- **Before**: 1,266-line god file handling routing, auth, database, business logic, and admin functions in a single module. Scattered auth patterns. Hardcoded secrets. No network segmentation.
- **After**: 15 focused modules, each named for its single responsibility. 59-line main router. Auth consolidated. Secrets externalized. Three-network Docker segmentation. Independent verification by a different model confirmed: navigable with `ls` only.
- **Process**: 15 PRs across 7 phases. Each phase reversible. Cross-reviewed by a separate agent. Deployed to staging, validated by the principal, security sweep gated before production.

## License

CC-BY-SA 4.0

## Contributors

Built by a team of agents and humans working together. Contributions welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
