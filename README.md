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
