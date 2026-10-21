---
name: simplify
description: Clean up changed code without changing its behavior. Reviews the current diff for reuse, simplification, efficiency, and altitude problems, then applies the fixes. Use after implementing a change, before committing or opening a PR, or when the user asks to simplify, tidy, or clean up recent changes. Quality only; it does not look for correctness bugs, so use a code-review workflow for that.
argument-hint: "[<target>]"
---

# Simplify

Improve the quality of the changed code. Review it from four angles, then fix what you find.

Do not look for correctness bugs. Do not change intended behavior. If the user passed an argument (a PR number, branch name, commit range, or file path), it is the review target.

## Phase 0: Get the diff

1. If the user gave a target, get the diff for that target and skip to step 4.
2. Run `git diff @{upstream}...HEAD`. If there is no upstream, run `git diff main...HEAD` (or the repository's default branch). If that also fails, run `git diff HEAD~1`.
3. If there are uncommitted changes, or the range diff is empty, also run `git diff HEAD` and include the working-tree changes. The cleanup often runs before the commit.
4. Treat the result as the review scope. If there is no Git repository and no target, ask the user which files or changes to review.

## Phase 1: Review

Every finding has the same four fields:

- `file`: the path.
- `line`: the line number in the new code.
- `summary`: one line that names the problem and the better form.
- `cost`: what is duplicated, wasted, or harder to maintain.

Choose one mode:

- **Parallel mode.** Use this if you can spawn subagents. Start four independent subagents in one message so they run at the same time. Give each subagent the full diff, one angle from the list below, and the finding format above. Tell each subagent to report findings only and not to edit files. Then wait for all four to finish.
- **Single-pass mode.** Use this if you cannot spawn subagents, for example when you are already a subagent. Review the diff against each angle in turn, yourself, in this context. Do not skip an angle because you work alone.

### Angle 1: Reuse

Find new code that re-implements something the codebase already has. Search shared or utility modules and the files next to the change. Name the existing function, type, or helper to call instead.

### Angle 2: Simplification

Find complexity the diff adds without need: redundant or derivable state, copy-paste with small variations, deep nesting, and dead code left behind. Name the simpler form that does the same job.

### Angle 3: Efficiency

Find wasted work the diff adds: repeated computation or I/O, independent operations run in sequence, and blocking work on startup or hot paths. Also find long-lived objects built from closures that capture a large enclosing scope; the whole scope stays alive as long as the object does. Prefer a type that holds only the fields it needs. Name the cheaper alternative.

### Angle 4: Altitude

Check that each change fixes the root cause at the correct depth, and does not patch a symptom. Special cases added to shared infrastructure show that the fix is not deep enough. Prefer one simpler, more general change to the underlying mechanism, and name that change.

## Phase 2: Apply the fixes

1. Merge findings that point at the same line or the same mechanism.
2. Skip a finding if any of these is true. Record each skip with a short reason; do not argue with the finding.
   - The fix changes intended behavior.
   - The fix needs changes far outside the reviewed diff.
   - You judge the finding to be a false positive.
3. Fix each remaining finding directly in the code.
4. Run the project's fastest relevant check (formatter, type checker, lint, or the affected tests) to confirm the fixes did not change behavior.

## Report

End with a short summary:

- What you fixed, one line per fix with `file:line`.
- What you skipped, and why.
- If there were no findings, say that the code was already clean.
- In single-pass mode, state that the review was a single pass without subagents and not the four-agent parallel review, so the reader knows what actually ran.
