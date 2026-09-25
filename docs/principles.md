# Principles

Agent-First CI starts from one observation:

> When coding agents can make useful changes faster than CI can validate them, validation latency becomes part of the development loop.

The response should not be to weaken validation. It should be to separate **feedback**, **authority**, **reuse**, and **lifecycle** more carefully.

## 1. Feedback is not authority

A coding agent benefits from fast feedback while it is still changing code.

That feedback can be narrower than the final merge gate:

- syntax checks;
- targeted tests;
- directly modified test files;
- cheap contract checks;
- static analysis;
- small deterministic smoke tests.

But narrow feedback must be labeled honestly.

A draft-stage PASS means:

> no failure was found in the draft feedback scope.

It must not silently mean:

> the repository is fully validated.

This distinction lets the development loop move quickly without laundering partial validation into release authority.

## 2. Authoritative validation has an explicit boundary

There should be a clear point at which the repository requires the complete validation contract.

In the reference pattern, that boundary is **Ready**.

A Ready/full result should prove the full current obligation for that repository, not merely run "more tests than Draft."

The exact obligation is repository-specific. The architectural point is that it is explicit.

## 3. Evidence reuse requires qualification

A green result is data.

A qualified green result may be evidence for another state only when the relevant identity is preserved.

Depending on the repository, qualification may include:

- exact source commit or tree;
- exact PR / merge association;
- workflow and attempt identity;
- environment and dependency identity;
- expected test collection;
- complete result coverage;
- artifact identity and integrity;
- absence of a newer disqualifying run.

The rule is not:

> it passed before.

The rule is:

> this prior result is demonstrably evidence for the thing we need to validate now.

See [Qualified Evidence Reuse](evidence-reuse.md).

## 4. UNKNOWN means validate

Optimization logic is allowed to be uncertain.

What it is not allowed to do is turn uncertainty into permission.

Examples:

- changed-file scope could not be established;
- the prior full run cannot be uniquely identified;
- the environment identity changed;
- test collection completeness cannot be proven;
- the merge relationship is ambiguous.

The safe response is simple:

> **UNKNOWN → full validation.**

This keeps optimization logic outside the authority boundary.

## 5. Optimize the loop, not just the bill

Runner-minutes matter. So do infrastructure cost and queue pressure.

But for an agent-heavy workflow, the dominant product metric may be the wall-clock delay before useful work can continue.

That motivates **Time to Next Agent Action (TTNAA)**.

A change can be successful even if it uses slightly more parallel runner time, provided it materially reduces blocking latency and the cost remains acceptable.

The trade-off must be visible rather than hidden.

## 6. Validation has a lifecycle

CI tends to grow monotonically because adding protection is easy and removing obsolete protection is socially and technically harder.

Agent-First CI treats validation as owned by the capability it protects.

When a capability is retired, review its dedicated:

- tests;
- fixtures;
- workflow branches;
- exceptions;
- adapters;
- compatibility readers;
- retry/recovery paths.

Shared protection and evidence readers may remain. Dedicated validation with no live capability should not survive by inertia.

See [Validation Lifecycle](validation-lifecycle.md).

## 7. Reuse existing execution machinery

The architecture is intentionally conservative about tooling.

Use mature components for ordinary execution concerns:

- GitHub Actions or another CI platform;
- pytest or the repository's native test runner;
- test sharding;
- worker parallelism;
- environment setup;
- dependency caching;
- static analysis.

Custom logic should earn its existence by protecting a semantic boundary that those tools do not already provide.

## 8. Do not turn optimization into a second platform

A common failure mode is to solve CI complexity by building another CI system on top of CI.

Avoid introducing:

- a universal workflow DSL;
- a generic dependency graph unless the repository actually needs one;
- a permanent selector service before simpler rules are exhausted;
- a second canonical state store;
- opaque heuristics that can silently reduce validation.

The goal is shorter reliable feedback loops, not a larger control plane.

## Compact invariant

```text
fast feedback may be partial
final authority may not be partial
prior results may be reused only when qualified
uncertainty falls back to full
validation should die with the capability it exclusively protects
```
