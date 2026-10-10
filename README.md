# loop-harness

Turn a repo into autonomous, dependency-ordered work an agent loop can drive
end-to-end. You write (or generate) a `BACKLOG.md`; an agent loop picks the next
unblocked item, does it, verifies, opens a PR, merges if authorized, updates
status, and repeats, until everything left needs a human.

It's two small things: a **versioned `BACKLOG.md` schema + loop protocol**
([`SCHEMA.md`](SCHEMA.md)), and a **`backlog` skill** ([`SKILL.md`](SKILL.md))
that generates a fitted backlog by surveying your codebase.

## Why it works

The loop is reliable because of three design choices, all in `SCHEMA.md`:

1. **The protocol travels in the file.** `BACKLOG.md`'s header *is* the run
   rules, so the driving prompt is just *"work the next eligible item per the
   loop protocol in BACKLOG.md"*. The loop re-reads its own instructions every
   iteration.
2. **The project profile is discovered once.** Test command, default branch,
   pre-existing failures to ignore, merge authorization, repo-specific hard
   rules, all baked into the file so the loop doesn't rediscover them 18 times.
   This block is the difference between a loop that works and one that flails.
3. **The file is checked against the repository before every pick.** An item
   whose work already merged is closed, not worked again. The loop does
   exactly what the file says, so the file has to be true (schema 1.1.x).

## Start small

If you have never run a queue, start with the five-item version in
[Lesson 6 of the team-of-one series](https://caskeycoding.com/blog/lesson-6-the-backlog-queue):
one file, a pick rule, an acceptance line per item, a `tier` per item, and one
investigate item for the work you cannot write an acceptance line for yet. It
is a teaching subset of `SCHEMA.md`. To adopt the full schema, map `doing` to
`in_progress` and `review` to `in_review`, add the required `size` to each item,
and use `### B-NNN` item headings. Copy the loop protocol and add the project
profile block, or use the `backlog` skill to survey a codebase and write a
fitted file. In this versioned queue, an investigation's backlog changes go
through a PR; merging it closes the parent, while its children remain `draft`
until a human promotes them.

## Quickstart

```text
# 1. Generate a fitted backlog (the "init")
/backlog            # surveys the repo(s), discovers the profile, writes BACKLOG.md

# 2. Drive it, self-paced
/loop Work the next eligible item per the loop protocol in BACKLOG.md

#    …or on a fixed cadence
/loop 30m Work the next eligible item per the loop protocol in BACKLOG.md
```

After reconciling merged PRs, the loop stops when no eligible item remains:
no `ready` item has all dependencies `done` and `human_gate` other than `true`.
This includes queues waiting on `draft` items or unmet dependencies. It tells
you what's left for you.

## The agent-orchestration pattern

This is the run discipline that makes it safe to leave running:

- **One item per iteration, never batch.** Smaller blast radius, clean status.
- **Delegate the build, verify yourself.** Each item's implementation can run as
  a sub-agent; the orchestrator re-runs the checks and reads the diff before
  merging. A sub-agent reporting "tests pass" is a claim, not a result.
- **Verify before merge; ignore only known failures.** The profile lists
  pre-existing failures; anything else blocks the merge.
- **Human-gate the irreversible.** Archiving, billable resources, outward-facing
  publishing, business calls. The loop surfaces these and moves on, never acts.
- **Newly-discovered work becomes new items**, not silent scope creep on the
  current one.

## Proof

First run: the CaskeyCoding workspace, 17 dependency-ordered items across 5
repos (a Python agent-orchestration platform and a multi-repo web product),
**19 PRs opened, verified, and merged** across self-paced iterations, stopping
cleanly at the work that needed a human: one item it refused to spec (a
dependency that lived only on another machine) and four human-gated decisions.
That backlog seeded `SCHEMA.md` v1.0.0.

## Layout

```
loop-harness/
  SCHEMA.md                  # the versioned contract (item fields + protocol + project profile)
  SKILL.md                   # the `backlog` skill: survey, then generate BACKLOG.md
  templates/BACKLOG.template.md
  CHANGELOG.md               # schema semver history
```

## Install the skill

Copy or symlink `SKILL.md` into a skills dir Claude Code reads (e.g.
`~/.claude/skills/backlog/SKILL.md` for user-level, or
`.claude/skills/backlog/SKILL.md` in a project), keeping `SCHEMA.md` alongside
it. Then `/backlog` is available in that scope.
