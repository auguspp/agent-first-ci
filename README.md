# Agent-First CI

> **Minimize time-to-next-agent-action without weakening authoritative validation.**

Coding agents changed the economics of software delivery.

When code generation becomes fast enough, CI is no longer just a background quality gate. It becomes part of the synchronous agent loop:

```text
agent changes code
        ↓
validation
        ↓
agent reads the result
        ↓
agent changes code again
```

If validation takes minutes and the agent needs to cross that boundary several times per task, CI becomes the bottleneck.

**Agent-First CI** is a small reference architecture for that problem.

It is not a new CI platform, test framework, dependency graph, or autonomous software factory. It is a set of validation rules for high-frequency coding-agent workflows built mostly from existing CI primitives.

## Core model

```text
Draft
  │
  ├─ fast feedback
  │   useful, non-authoritative
  ▼
Ready
  │
  ├─ authoritative full validation
  ▼
Merge / main
  │
  ├─ qualified reuse of existing full-suite evidence
  │
  └─ if identity or evidence is uncertain → run full validation
  ▼
Next agent action
```

The key distinction is:

> **Feedback can be partial. Authority cannot.**

## The metric

Traditional CI optimization asks:

> How long does CI take?

Agent-first CI adds a more useful question:

> **How long until the agent can safely act again?**

This repository calls that **Time to Next Agent Action (TTNAA)**.

TTNAA is not the same thing as runner-minutes. More parallel compute can increase total runner time while reducing the wall-clock delay that blocks the development loop.

## Five principles

1. **Fast feedback is not authoritative validation.**  
   Draft-stage checks may be intentionally narrow as long as they cannot be mistaken for merge authority.

2. **Reuse evidence, not vibes.**  
   A prior green run is reusable only when the relevant identity is proven: code/tree, source run, environment, test collection, result, and merge association where applicable.

3. **UNKNOWN falls back to full.**  
   Missing or ambiguous qualification is not permission to skip validation.

4. **Validation follows capability lifecycle.**  
   When a capability is retired, its dedicated tests, fixtures, workflow exceptions, and CI plumbing should be reviewed for retirement too.

5. **Reuse mature execution tools.**  
   Sharding, parallelism, environment setup, caching, and workflow orchestration should normally come from established tools. Custom code should be reserved for the validation semantics that generic tools do not provide.

More detail: [Principles](docs/principles.md) · [Qualified Evidence Reuse](docs/evidence-reuse.md) · [Validation Lifecycle](docs/validation-lifecycle.md)

## A real reference case

This idea came out of a real Python/GitHub Actions repository with more than 6,000 tests and a coding-agent-heavy development loop.

Observed production samples during the September 2026 CI redesign:

| Stage | Before / result |
| --- | ---: |
| Full PR validation | ~381s → ~193s |
| Draft feedback | ~56s |
| Ready full validation | ~183s |
| Post-merge qualified reuse | ~36s |

The full-suite improvement used more parallel runner time; it optimized waiting, not merely compute cost. The post-merge path reused an exact prior full-suite result only after qualification rather than treating any earlier green run as equivalent.

These are **observed samples, not universal performance claims**. The implementation and evidence trail live in the public [Decision Kernel](https://github.com/auguspp/decision-kernel) repository, especially [PR #558](https://github.com/auguspp/decision-kernel/pull/558) and [PR #559](https://github.com/auguspp/decision-kernel/pull/559).

See [Benchmark notes](docs/benchmark.md) for scope and caveats.

## What this is not

Agent-First CI is deliberately not:

- a universal CI orchestration framework;
- a replacement for GitHub Actions, pytest, Bazel, Pants, Nx, or similar systems;
- a promise that affected-test selection is always safe;
- an excuse to weaken final validation;
- a requirement to maximize parallelism regardless of cost;
- a reason to build a second state or workflow platform.

The reference implementation should stay boring where mature tools already exist.

## Current status

**Thesis + reference architecture.**

The first goal of this repository is to leave a clear, testable statement of the problem and the validation model. A minimal runnable example may be added once the pattern has survived more real development cycles.

That is intentional: define the invariant first, productize only if reality justifies it.

## Short version

```text
Coding agents made CI part of the inner loop.

Optimize:
    time-to-next-agent-action

Keep:
    authoritative final validation

Reuse:
    qualified evidence

Fallback:
    UNKNOWN → full validation

Retire:
    capability gone → dedicated validation reviewed for removal
```

---

If this problem sounds familiar, the most useful contribution is not a star. It is a concrete failure mode, counterexample, or real workflow where this model does or does not hold.
