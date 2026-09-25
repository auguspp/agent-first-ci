# Benchmark Notes

This repository began as a distilled record of a real CI redesign in [auguspp/decision-kernel](https://github.com/auguspp/decision-kernel).

The numbers below are historical observations from September 2026. They are included to show that the architecture emerged from a real development loop, not to claim universal performance.

## Reference samples

### CI v2 adoption

[Decision Kernel PR #558](https://github.com/auguspp/decision-kernel/pull/558) introduced a GitHub Actions matrix using mature execution tools including pytest-split, xdist, uv, and native `needs`.

Observed samples reported in the repository:

| Scope | Previous sample | New sample |
| --- | ---: | ---: |
| PR: first job start → final gate | 381s | 193s |
| main: first job start → final gate | 356s | 169s |
| PR runner-minutes | 6.70 | 8.40 |
| main runner-minutes | 6.28 | 8.72 |
| unique tests | 6140 | 6145 |

The important trade-off is visible: wall-clock latency dropped while total runner time increased.

That is consistent with an agent-first optimization target: reduce blocking latency when the additional compute is acceptable.

These were real normal PR/main samples, not a controlled identical-run benchmark. They should not be interpreted as a precise causal percentage.

### Draft / Ready / reuse

[Decision Kernel PR #559](https://github.com/auguspp/decision-kernel/pull/559) exercised the staged model on a useful test refactor rather than a synthetic benchmark.

Observed path:

```text
Draft feedback        ~56s
Ready full            ~183s
main qualified reuse   ~36s
```

The Ready run covered 6,146 unique tests.

The main path did not rerun the four full test shards. It reused the exact Ready full-suite evidence after the repository's qualification checks and ran the smaller main smoke/readback path.

## What the benchmark does not prove

These observations do not prove that:

- every repository should use four shards;
- 4×2 runner/worker topology is generally optimal;
- uv always outperforms pip by the same amount;
- every main merge can reuse PR evidence;
- Draft feedback can safely use the same scope elsewhere;
- 6,000 tests is an inherently good or bad number.

The architecture is about the separation of responsibilities.

The execution layout should be measured in each repository.

## Metrics worth recording

A useful benchmark packet for agent-heavy CI should distinguish:

- total unique tests;
- installation time;
- independent collection time;
- test execution wall-clock;
- workflow wall-clock;
- queue delay;
- runner count;
- total runner-minutes;
- cache hit/miss;
- Draft vs Ready vs main behavior;
- time until the next agent can safely continue.

That last measure is the motivation for **Time to Next Agent Action (TTNAA)**.

## Source-of-truth note

The Decision Kernel repository is the canonical source for the historical run IDs, artifacts, implementation details, and contemporaneous caveats.

This document is intentionally a summary, not a duplicate evidence store.
