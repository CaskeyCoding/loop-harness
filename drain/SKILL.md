---
name: drain
description: Work exactly one item from BACKLOG.md and stop. Reads the file, runs the freshness check, picks by the loop protocol, does the work in the item's own repo, sets the item's status, and closes with a four-part message. Use when the user says "work the next item", "drain one", "take the next ticket", or invokes /drain; it is the one-iteration step for a watched run and the body of an unattended /loop.
---

# drain: one item, then stop

This skill is one iteration of the loop. It never batches, never merges
unless the project profile says it may, and never edits an item it is not
holding. The `backlog` skill writes the file; this skill works it.

Read `BACKLOG.md` in full before anything else. If it has a project profile,
that block wins over anything you would otherwise guess (test command,
default branch, known failures, merge authorization, hard rules). If the
workspace has a standing file (`CLAUDE.md` or `AGENTS.md`) or an authority
file (`CHARTER.md`), read those too; they bound what you may do while
working the item.

## Steps

1. **Pick.** Take the lowest-numbered item with `status: ready` whose
   `depends_on` are all `done`. Skip any item with `human_gate: true`. If no
   item qualifies, say so in one line, name what is blocking the rest, and
   stop.

2. **Freshness check.** If the item's `pr` is empty, search the default
   branch of its repo for the id before touching anything:
   `git log origin/<default> --oneline --grep=B-NNN`. If the work already
   merged, set the item `done` with a note saying where, report it, and go
   back to step 1 once. Do not spend the iteration re-deriving whether
   shipped work shipped.

3. **Claim.** Set the item `status: in_progress` (`doing` is the accepted
   short name). Create a branch `<branch_prefix>/B-NNN` off the repo's
   default branch. From here on, edit only inside that repo plus this one
   item's block in `BACKLOG.md`.

4. **Work.** Build to the item's `acceptance` line and nothing wider. If the
   item names a `spec`, read it first and treat its Out of scope as a wall.
   For an investigate item (`tier: judge`, acceptance "produces child
   items"), the output is items, not code: append the children at the bottom
   of `BACKLOG.md` as `status: draft`, each with a checkable acceptance line,
   each confined to one repo. Work you discover that is not this item becomes
   a new draft item at the bottom, never a wider current one.

5. **Verify.** Run the acceptance check and the project's checks yourself.
   Quote the exact command and its exit status in the closing message. A
   known failure listed in the project profile is ignorable; anything else
   is not done.

6. **Hand off.** Commit the changed files by path. Push if the repo has a
   remote; open a PR referencing `B-NNN` if you can. Set the item
   `status: in_review` (`review` is the accepted short name) and put the PR
   URL, or the local commit and branch, in `pr`. Merge only if the project
   profile says `merge: authorized` and the checks are green; then set
   `done`. Otherwise `done` is the human's move.

7. **Stop.** Do not take a second item in the same invocation.

## The closing message

Four parts, in this order, every time:

1. What shipped, with the check that ran and its result.
2. Which choices you made between reasonable options, and why.
3. What the item or spec could be read to demand that you left out, and why.
4. Follow-ups that deserve an item of their own.

## When you are not sure

Come back with the question. Write it in one line, name the two answers you
can see, say which you would pick and why, set the item `status: blocked`
with the question in `notes`, and stop. Never guess, and never narrow or
widen the item to avoid asking. If the question is one the repository could
have answered, go and read the repository instead.

## Rules

- One item per invocation.
- Never edit an item you are not holding; appending draft children is the
  one exception.
- The acceptance line is the contract; "looks done" is not a status.
- Respect every repo's standing file and the project profile's hard rules.
- A finished item is `done` only when its change is merged.
