---
name: verify-change
description: Independently falsify whether a material change that someone else implemented is correct and complete. Use in PR review or a QA role for behavior, compatibility, protocol, data-contract, or source-transform changes; when a public API, schema, or boundary is replaced, migrated, or retired and surviving capabilities must be checked; when asked to verify or prove a change or whether a fix is complete; or when a regression escaped coverage and its fix needs re-checking. Do not use on your own implementation in the same task (directly or via a subagent), as a routine test runner, to decide whether a test should exist (use test-design-review), for mechanical changes covered by normal gates, or to approve a design. Returns VERIFIED, NOT VERIFIED, or INCONCLUSIVE.
---

# Verify Change

Run the reviewer or QA pass for a material change whose acceptance needs independent falsification. Do not only rerun tests written with the implementation.

Find the project's verification and testing policy through the nearest `AGENTS.md` or contributor guide. It overrides this skill. This skill is a procedure, not a second policy. It checks correctness only. Whether the mechanism should exist is a design question.

Why: a green suite can agree with a wrong expectation. A replaced boundary can leave legacy tests green while the new one drops capabilities. A check helps only if it constrains the change from outside.

## Procedure

The steps are an order of reasoning. One decisive check can settle several.

1. **Claim.** State the exact observable behavior that must hold. Example: "a user who leaves mid-session has the session's handles released", not "cleanup is improved". If you cannot make it concrete, the result is `INCONCLUSIVE` until the contract is clarified.
2. **Independent authority.** Find what defines correct behavior outside the implementation: an accepted contract, Issue, or ADR; reproducible behavior of the real runtime or platform; a published spec; an established reference; or the accepted pre-migration contract. With none, do not infer correctness from the code or its new tests. Report `INCONCLUSIVE` and name the missing contract.
3. **Smallest decisive falsification.** Pick the narrowest check that fails if the claim is false: a focused test, a minimal input, a real-flow reproduction, or a before/after comparison. Where practical, temporarily invert or remove the key change, confirm the check fails, then restore it. Add no permanent production scaffolding.
4. **Compare.** Compare observed behavior with the authority. Compare pre- and post-change behavior where meaningful. Ask whether a plausible wrong implementation would also pass what you collected. A missing historical baseline does not invalidate a check that the current contract can decide. Record it if it matters.

## Boundary migrations

When a change replaces, hides, or retires a public or authoritative boundary, derive claims from the accepted pre-migration contract. Do not derive them from the replacement or its tests.

Account for each capability as one of: preserved, approved removal or change, or transferred to a named owner. Verify preserved capabilities through the replacement boundary itself. A check through a retired or compatibility-only path cannot support `VERIFIED`.

## Verdict

- `VERIFIED`: behavior matches the independent authority, and the check would catch a plausible wrong implementation.
- `NOT VERIFIED`: behavior conflicts with the contract, shows a material unexpected difference, or silently drops a surviving capability.
- `INCONCLUSIVE`: the authority or checks cannot separate correct from incorrect, or the surviving scope is unclear.

If the verdict is not `VERIFIED`, state which contract, check, or change would resolve it. Never report `VERIFIED` only because existing tests are green.

Verification output is task-local. Keep logs, reports, and minimized inputs ephemeral. If a regression deserves permanent coverage, report it as a candidate for the owning repo's existing test structure.

```text
Claim: <falsifiable behavior>
Authority: <independent contract or source>
Surface: <decisive check>
Observation: <result or comparison>
Verdict: VERIFIED | NOT VERIFIED | INCONCLUSIVE
Limitations: <material uncertainty, or none>
Repository test candidate: <only if durable coverage should be considered>
```
