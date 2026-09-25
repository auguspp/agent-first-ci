# Qualified Evidence Reuse

Qualified Evidence Reuse is the idea that a previous CI result may satisfy a later validation need **only when the result's identity and coverage are proven to match that need**.

It is deliberately stronger than duplicate-run skipping.

## Cache vs evidence

A cache answers:

> Can I reuse some previous computation?

Evidence reuse asks:

> Does this previous result prove the property required at this point in the workflow?

Those are different questions.

A dependency cache can be useful even when stale entries merely cause slower or failed work.

A reused authoritative CI result is more sensitive: if it refers to the wrong code, environment, scope, or run, the repository may incorrectly accept an unvalidated state.

## Reference pattern

A common high-frequency agent workflow looks like this:

```text
PR head H
   │
   ├─ Draft feedback
   │
   └─ Ready full validation F(H)
             │
             ▼
        clean merge M
             │
             ├─ if M has the same validated tree and the full evidence qualifies
             │      reuse F(H)
             │
             └─ otherwise
                    run full validation F(M)
```

The reuse decision is not based on branch names or commit messages.

It is based on identity.

## Typical qualification dimensions

The exact contract belongs to the repository, but useful dimensions include:

### Code identity

- the validated PR head is known exactly;
- the resulting merge tree is identical where that is required;
- CI policy files did not change in a way that invalidates reuse.

### Run identity

- the source run belongs to the expected workflow;
- the source run belongs to the expected repository;
- the event type is the expected one;
- the run attempt is acceptable;
- the result is complete and successful;
- a newer run has not superseded it.

### Environment identity

Where environment equivalence matters, compare things such as:

- Python/runtime version;
- runner OS and architecture;
- image version;
- installed package versions;
- relevant tool versions.

The point is not to fingerprint everything forever. The point is to make the reuse assumptions explicit.

### Test-set identity

A full result should establish what "full" actually meant.

Useful evidence can include:

- collected test identities;
- shard membership;
- union and intersection checks;
- JUnit or equivalent execution records;
- zero unexpected skips/failures/errors where required.

A green job without coverage identity may be insufficient evidence for reuse.

### Artifact identity

If the proof is carried through artifacts:

- bind the artifact to its source run;
- verify expected names and sizes;
- verify integrity where available;
- parse artifacts as data, not executable input;
- reject missing, malformed, or ambiguous proof.

## Fail closed

Every qualification step should be allowed to say:

```text
UNKNOWN
```

Examples:

- the merge association cannot be proven;
- the artifact is missing;
- collection identity is incomplete;
- the environment changed;
- there are multiple plausible source runs.

The consequence should be:

```text
reuse = false
run full validation
```

This is what keeps the reuse layer from becoming a second authority system.

## Why this matters more with coding agents

Coding agents can create many more validation boundaries per unit of human time.

Without reuse, a typical loop may repeatedly reproduce the same proof:

```text
agent iteration
→ full CI
→ agent iteration
→ full CI
→ Ready full CI
→ merge
→ full CI again
```

The goal is not to skip proof.

The goal is to avoid **reproducing an already valid proof when its applicability can be established exactly**.

## Non-goals

Qualified Evidence Reuse does not imply:

- all merge CI should be skipped;
- SHA equality alone is always enough;
- environment identity must be infinitely strict;
- every repository needs artifact-level proof;
- test-impact analysis is required;
- a prior green result should override a failed or ambiguous newer result.

The implementation should be as small as the repository's actual validation contract allows.
