# Validation Lifecycle

CI tends to accumulate.

A feature adds tests.  
A recovery path adds tests.  
A migration adds compatibility checks.  
A workflow experiment adds structural assertions.  
The feature later disappears.

The validation often remains.

Over time, the repository pays for historical obligations that no longer protect live behavior.

## Principle

> **Validation should have the same lifecycle awareness as the capability it protects.**

This does not mean deleting history.

It means separating:

- evidence that must remain readable;
- shared contracts that still protect live behavior;
- dedicated validation that exists only because a retired capability once existed.

## Capability retirement review

When a capability is retired, review at least:

```text
capability
├─ dedicated tests
├─ dedicated fixtures
├─ workflow jobs / branches
├─ CI exceptions
├─ compatibility adapters
├─ recovery paths
└─ evidence readers
```

Each item can be:

- **KEEP** — still protects a live/shared contract;
- **CONSOLIDATE** — protection remains but can move to the owning layer;
- **RETIRE** — no live capability depends on it;
- **DEFER** — evidence is insufficient to decide safely.

The important behavior is that retirement creates a review obligation instead of leaving validation behind automatically.

## What should usually remain

Historical evidence may still need to be readable after the execution path that produced it is gone.

Examples:

- parsers for retained CI evidence;
- migration readers;
- compatibility readers for frozen artifacts;
- audit records.

A reader is not the same thing as an active execution path.

Deleting the old executor does not require deleting the ability to understand its historical output.

## What should usually leave

Good retirement candidates include:

- tests that exist only to prove a removed implementation detail;
- fixtures with no live consumer;
- workflow branches for a removed execution path;
- CI exceptions whose triggering capability no longer exists;
- duplicated integration tests after ownership has moved to a lower-level contract plus a small wiring test.

The standard is not "old code is bad."

The standard is:

> What live behavior does this validation still protect?

If the answer is UNKNOWN, investigate before deleting.

If the answer is nothing, retirement is usually healthier than indefinite preservation.

## Why this matters for agent-heavy repositories

Coding agents lower the cost of adding code and tests.

That is useful, but it also means validation debt can accumulate faster than before.

Without lifecycle discipline:

```text
agent adds capability
→ agent adds tests
→ agent adds edge-case tests
→ architecture changes
→ old tests remain
→ CI gets slower
→ more CI optimization machinery is added
```

The result can be a repository where validation complexity grows faster than product complexity.

Lifecycle review changes the loop:

```text
capability added
→ validation added

capability consolidated
→ validation consolidated

capability retired
→ dedicated validation reviewed for retirement
```

## Avoid the registry trap

This principle does not require a new lifecycle platform.

Start with simple review discipline in normal code changes.

Only build automated orphan detection when the repository has enough recurring structure to justify it.

The purpose of lifecycle awareness is to reduce machinery, not create another permanent control system.
