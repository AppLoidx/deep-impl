---
name: deep-impl
description: Implement one clear goal to expert depth - deeper, not bigger. Usage /deep-impl [--go] <clear goal>.
disable-model-invocation: true
---

# deep-impl

Take the goal in the args and deliver it the way a field-leading expert would: from the rawest
material, with real instruments, proven by an outside judge, with its blast radius handled.
**Deeper, not bigger.** Depth stays on the goal; it never adds new goals.

This skill sets the bar and the direction. Your normal workflow (repo `CLAUDE.md`, routing,
TDD, review and PR skills) still does the work.

## The goal line

Before anything else, write the goal as one line plus a **done-when** that an outside judge can
check. Every action from here traces to that line. Empty args: ask for the goal. Ambiguous goal:
take the most likely reading and state it under Assumptions.

**On-goal** work is anything the goal fails or degrades without, or a direct consequence of the
change on whichever of these exist: callers, stored data, migrations, tests, docs, observability,
consuming repos. A refactor the root cause requires is on-goal, however wide. That is the depth
to chase hard.

**Off-goal** findings (adjacent bugs, refactors the goal does not need, nice ideas) go to the **parking lot**:
listed in the plan and the report, tracked via `spawn_task` or a sub-issue when worth it. They
stay unimplemented in this run.

## 1. Depth pass (before planning)

One line per question; the answers feed the plan fields.

0. **Baseline**: how much of the goal already holds today? Measure it on the current code or
   system, so the plan covers only the real gap.
1. **Expert move**: what would a top professional do here that the request does not spell out?
   Typical: fix the root cause, cover every call site, handle the failure and concurrency paths,
   leave the system easier to change.
2. **Rawest material**: what do they start from instead of digested summaries? Whichever exist
   for this goal: live code on the default branch, prod data and logs, provider contracts and
   API specs, real failing cases. Memory, tickets and docs are leads to verify.
3. **Instruments**: which real methods and tools do they use? Reproductions, queries, profilers,
   benchmarks, property tests, type checks - over reasoning and markdown tables.
4. **Outside judge**: what, other than you or the dev, shows it is right? A test red before and
   green after, a reproduction, CI, a prod query, a benchmark number, a sandbox call, an
   independent reviewer.
5. **Blast radius**: what else does this touch - callers, stored data, concurrency, rollout,
   backward compatibility, consuming repos? Trace each to confirm or clear it.

Set an ambitious bar on the goal itself (the root cause rather than the symptom; 200ms rather
than 5s). Hardening joins the plan when a real input or a code path shows it changes whether
done-when holds; otherwise it goes to the parking lot. Investigate before calling anything a
blocker.

## 2. Plan (shown to the dev)

The plan the dev sees fits on one screen, one line per bullet:

```
Goal:        <one line>
Done when:   <outside-judge check>
Approach:    <3-6 one-line bullets>
Blast radius:<what else changes and how it stays safe>
Assumptions: <choices you made that the dev may veto; no stop needed>
Parking lot: <off-goal findings; anything broken right now goes first>
Decisions:   <only choices outside authorization; else omit>
```

Write a fuller spec when the work is multi-part (into the target repo's `docs/`); decide that
yourself and link it under the short plan.

**Gate**: when the args carry `--go`, the dev already said to proceed without approval, or `daq`
is active, show the plan and start implementing in the same turn. Otherwise wait for one approval.
That is the only planned stop. Items under Decisions still wait for the dev even with `--go`;
build the parts that do not depend on them meanwhile.

## 3. GitHub issues (only when tracking helps)

When the work spans several PRs or the repo tracks work in issues: one **parent issue** holds the
goal and plan. Every issue you create is its sub-issue; existing related issues are attached too.

```bash
gh issue create --parent PARENT ...           # new child
gh issue edit PARENT --add-sub-issue 123,124  # existing related issues
```

Where an issue cannot be a sub-issue (another repo's tracker, permissions), link it in the
parent's body. Every issue ends reachable from the parent.

## 4. Implement

- Decide and keep moving. Log assumptions in the PR or `implementation-notes.md`.
  Ask only for decisions outside the authorization (new provider, outward or irreversible
  action, deployment topology).
- At each milestone, check the work against the goal line; route anything off-goal to the
  parking lot.
- Keep going until the outside judge passes. A wall is a cue for another instrument.
- Evidence that the goal as stated cannot achieve its purpose is a real decision: stop and show
  it.

## 5. Report

Lead with the done-when result and its evidence (command, output, test, query). Then blast
radius handled, assumptions made, parking lot. Nothing else.

## Drift check

| Thought | On-goal move |
|---|---|
| "While I'm here, I'll also clean up X" | X in the parking lot; finish the goal |
| "The plan needs more detail before I show it" | One screen; detail goes in the spec |
| "Better confirm this with the dev" | Inside authorization: decide, log it, continue |
| "Tests pass, done" | Done = the outside judge from done-when |
| "Might as well harden Y too" | Y joins only if a real input breaks done-when |
| "The summary says it works this way" | Verify on the raw source |
| "This is impossible" | Try another instrument before reporting a blocker |
