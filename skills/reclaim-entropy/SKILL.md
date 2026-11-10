---
name: reclaim-entropy
description: Find or remove duplicate truth, dead or redundant abstractions, obsolete fallbacks or compatibility layers, post-migration leftovers, unused consumers, and duplicated state or config. Use for codebase cleanup, entropy audits, migration cleanup, or "is this still load-bearing?" investigations, and when the user asks what can be deleted or simplified. Do not use for routine feature work, correctness-only review, removing public or persisted contracts without owner approval, or approving a new mechanism before implementation.
---

# Reclaim Entropy

Reduce maintenance obligations. Line count is not the goal.

This is cleanup after the fact. It does not decide whether a proposed new mechanism should exist.

Project policy comes first. Find the project's engineering, entropy, and testing policy through the nearest `AGENTS.md` or contributor guide. Repo-local contracts override this skill. This skill defines an investigation workflow, not a compatibility policy.

> Scanners create candidates. Consumer inspection and contract checks justify a cut.

Why: unused-looking code can encode an external contract, a dynamic path, or a lifecycle need. Reachable code can still be accidental duplication.

## Modes

- **Audit**: rank candidates. Do not edit.
- **Apply**: implement simplifications that the user already authorized. Verify them.
- **Focused investigation**: decide whether one named abstraction, state, dependency, fallback, or layer is load-bearing.

A review or audit request does not authorize edits.

## Judge each candidate

1. **Obligation.** Name what is maintained: API, state, wrapper, fallback, dependency, flag, lifecycle mechanism, fixture, generated artifact, or duplicated fact. Deleting lines that only move the obligation elsewhere is not a gain.
2. **Dependents.** Trace callers until you can classify the surface as production, test-only, generated, dynamic, or external. Search strings, config and protocol keys, build and codegen paths, registries, and consumers in sibling repos. A local reference count misses these.
3. **Load-bearing reason.** Find the contract, ownership decision, compatibility need, lifecycle need, or live consumer that justifies it. Do not authorize the cut yourself if it touches a public or persisted contract, declared compatibility, a security or privacy boundary, or an unresolved ownership decision. Route it to its owner.
4. **Net effect.** Prefer one canonical owner and fewer pass-through layers. Count concepts, synchronization points, dependencies, and lifecycle states. A cut that adds equivalent glue elsewhere is not a simplification.

Places to look, not findings: duplicate truth, pass-through layers, abandoned extension points, one-off generality, migration residue, overlapping lifecycle mechanisms, hand-rolled infrastructure, artifacts for removed behavior.

## Apply

- Stay inside the ownership boundary.
- Remove the obligation end to end: exports, docs, config, tests, generated output.
- Keep unrelated cleanup out.
- Keep behavior that a real contract requires.
- Verify the smallest decisive property first. Then run the repo gates for the affected boundary.
- Do not weaken tests, diagnostics, or validation to make a deletion pass.

## Output

Give a few well-supported candidates. Finding no worthwhile cut is a valid result.

```text
Candidate: <maintenance obligation>
Consumers: <production, dynamic, external, or cross-repo>
Contract: <public/compatibility contract or ownership decision>
Change: <what can be removed or consolidated>
Net effect: <concepts removed and any replacement cost>
Authority: <implementation-level, or owner decision required>
Verify: <smallest decisive check>
```

During PR review, do not grow the approved PR into cleanup that its Issue does not include.
