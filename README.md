# deep-impl

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that implements **one clear goal
to expert depth**. Deeper, not bigger: the agent chases the root cause, every call site and the
blast radius of the change, and proves it with an outside judge. It never adds new goals; off-goal
findings go to a parking lot.

## What it does

1. **Goal line** - restates the goal as one line plus a *done-when* an outside judge can check.
2. **Depth pass** - baseline, expert move, rawest material, instruments, outside judge, blast radius.
3. **One-screen plan** - goal, done-when, approach, blast radius, assumptions, parking lot.
   Waits for one approval unless you pass `--go`.
4. **Implement** - decides and keeps moving inside your authorization; logs assumptions.
5. **Report** - leads with the done-when result and its evidence.

It sets the bar and direction only. Your normal workflow (repo `CLAUDE.md`, TDD, review and PR
skills) still does the work.

## Example plan

What the dev sees before implementation starts (mock task: duplicate orders on webhook retry):

```
Goal:         A retried payment webhook never creates a second order; one order per payment event.
Done when:    A test replaying the same event 5x concurrently fails before the change and
              passes after (1 order, 4 no-ops). CI green. After deploy: 0 payment events in
              prod with more than one order (query in PR).
Approach:     - Expand: add orders.payment_event_id with a unique index; backfill from the
                stored webhook payloads (live: 41k rows, 12 duplicates).
              - Handler: insert with ON CONFLICT DO NOTHING and return the existing order,
                so a retry answers 200 instead of 500.
              - Replace the in-memory dedupe cache, which misses retries that land on
                another instance (root cause).
              - Contract: drop the cache and its config once the index is live.
              - Short spec in docs/; implementation and an independent review.
Blast radius: - Old and new versions run side by side during rollout: PR 1 adds the column
                and index and switches writes; PR 2 removes the cache after PR 1 is live.
              - The 12 existing duplicates block the unique index: the backfill keeps the
                first order and marks the rest void (refund state checked per row).
              - Order-created webhook to merchants fires once per order, so no change there.
Assumptions:  - The payment event id is stable across retries (provider docs and 3 observed
                retries agree).
              - Voided duplicates stay visible in the admin history.
Parking lot:  - Refund webhook has the same retry pattern (no duplicates seen yet).
              - Order list query is slow on large accounts; unrelated to this goal.
Decisions:    - Refund the 2 duplicate orders that were charged twice? That is a live money
                movement, so it waits for you; the fix ships without it.
```

How each field earns its place:

| Field | Rule |
|---|---|
| Goal | One line. Everything in the run traces back to it. |
| Done when | A check an outside judge can run: test red then green, CI, a prod query, a number. |
| Approach | 3-6 one-line bullets, root cause included. |
| Blast radius | Each thing the change touches and how it stays safe (rollout, data, consumers). |
| Assumptions | Choices made without stopping; the dev may veto. |
| Parking lot | Off-goal findings. Listed, not built. |
| Decisions | Only choices outside the authorization. Omitted when there are none. |

## Install

```bash
git clone https://github.com/AppLoidx/deep-impl ~/.claude/skills/deep-impl
```

or just the one file:

```bash
mkdir -p ~/.claude/skills/deep-impl && \
curl -fsSL https://raw.githubusercontent.com/AppLoidx/deep-impl/main/SKILL.md \
  -o ~/.claude/skills/deep-impl/SKILL.md
```

Start a new Claude Code session so the skill is discovered.

## Usage

```
/deep-impl make the export endpoint return in under 200ms for 10k rows
/deep-impl --go fix the duplicate webhook delivery on retry
```

The skill sets `disable-model-invocation: true`, so it runs only when you invoke it.

## Optional integrations

The skill mentions a few things that are used when present and skipped otherwise:

- `gh` CLI with sub-issues, for a parent issue when the work spans several PRs.
- `spawn_task` (Claude desktop app), to track parking-lot items as separate tasks.
- A "don't ask questions" mode (`daq`), treated like `--go`.
- `implementation-notes.md`, a log of deviations from the plan.

## License

MIT
