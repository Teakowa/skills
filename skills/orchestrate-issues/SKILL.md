---
name: orchestrate-issues
description: Drive ready Issues through implementation, independent review, and merge as the orchestrator. Pick the next Issue, dispatch an implementer session in its own worktree, dispatch a separate reviewer session, decide whether the PR may merge, merge it, and clean up. Use for an issue loop or backlog sweep, for dispatching implementer or reviewer sessions, or for applying the merge gate to an agent-authored PR that already has a review verdict. Also use for "clear the ready queue", "keep implementing the next issue", "stop the loop after #N". Do not use to implement one Issue yourself, to review one PR as the reviewer (use verify-change), for release or tag work, or to triage which Issues are ready.
---

# Orchestrate Issues

The orchestrator schedules work, integrates results, and owns the merge decision. It does not implement or review the Issues it dispatches.

Selection order, readiness, concurrency limits, and blocking labels come from the project's issue-readiness policy. Implementation follows `AGENTS.md`. Review follows the project's PR review policy. Find them through the nearest `AGENTS.md`. They override this skill. This skill adds the procedure that connects them, and the merge gate.

## Inputs

- **Scope**: the repos to sweep. Exclude repos that the project guidance or the user marks as frozen.
- **Stop condition**: the queue is empty, named Issues are done, or the user says stop.

## Cycle

### 1. Pick

Take the next Issue in the readiness policy's order, within its concurrency limit. Without a policy, use: open, no open `blocked-by` Issue, no open sub-issue, not labeled blocked or waiting for a decision. Order by priority, then bugs before features, then oldest first.

Before dispatch, re-read the Issue and its comments. Check for an open or merged PR that already covers it. Report an Issue that is already done or no longer matches the code. Do not reimplement it.

If an Issue is too broad to verify as one outcome, or needs an owner decision, comment the exact question, apply the repo's blocked or decision label, and move on. The orchestrator cannot split scope or choose product behavior.

### 2. Implement

Create one worktree per Issue from a fresh `origin/main`. Dispatch one implementer session into it. The implementer opens the PR.

- Never touch the shared checkout or a worktree that another session owns.
- Push with an explicit refspec (`git push origin <branch>`). Never rely on the checked-out branch.
- Check the worktree for file changes soon after launch. Stop and restart a session that makes no progress. Do not wait on it.

### 3. Review

Dispatch a reviewer in a session separate from the implementer. The reviewer checks out the PR head, runs the Issue's acceptance checks and the relevant gates, and tries to falsify each requirement. It does not trust the implementer's tests or claims. Use `verify-change` for this. Findings reach the implementer as review threads or one PR comment.

Allow at most two review rounds. If round two still has findings, close the PR and redo the Issue from scratch with the findings as input. Exception: if round two leaves one narrow finding, fix only that finding and run round three.

Why: a third round of patches on a design that failed twice rarely converges. A fresh attempt does.

### 4. Merge

Read PR state, checks, and review verdict in one call, with a project script if one exists, instead of repeated `gh pr view` and `gh pr checks` calls. Wait on a pending check with a monitor, not a `sleep` loop.

Merge only when all of these hold on the current head commit:

- The independent reviewer's latest verdict is approval.
- Every required check is green.
- The PR changes no path that CODEOWNERS assigns to a human owner, or that owner has approved.
- The PR is not a release PR, unless the user authorized that specific PR in the conversation.
- The user's loop request authorizes merging.

Before merging, re-read the Issue and compare the PR with its acceptance criteria. Squash-merge with an explicit subject and body. The body carries `Fixes #N` only when the whole Issue scope is delivered. Otherwise it carries `Refs #N`.

Why: the default squash body concatenates all commit messages. A stale `Fixes` from an early commit closes an Issue whose scope is still open.

After the merge, confirm that the Issue state matches the reference used. Do not tag, publish, or deploy.

### 5. Clean up

Remove an Issue's worktree and branch only after its PR shows `merged_at`. Never remove a worktree that has uncommitted or unpushed work. Use `trash`, not permanent deletion.

## Stop

When the stop condition is met or the user says stop, bring each in-flight Issue to a safe state: committed and pushed, or left untouched in its worktree. Start no new Issues. Delete no worktrees.

Stop and ask before any action this skill does not authorize. Examples: force-pushing a branch the orchestrator did not create, merging outside the gate, editing Issues beyond labels and comments.

## Report

For each Issue touched, give the PR, the review rounds, the merge commit or current state, and the Issue state. Then list decisions queued for the owner, PRs waiting on a code owner, and sessions that stalled or were redone.
