---
name: backlog
description: Generate or update a dependency-ordered BACKLOG.md for the current repo(s) so an agent loop (/loop) can drive it autonomously — survey the codebase, discover the project profile, and write loop-protocol-compliant items. Use when the user wants to "set up a backlog", "init the loop harness", "bootstrap autonomous work", or regenerate/rescan an existing BACKLOG.md.
---

# backlog — bootstrap an autonomous work loop

This skill turns a repo (or a workspace of repos) into a `BACKLOG.md` that
Claude Code's `/loop` can drive end-to-end. It does the **"dynamically create
the fit"** step: survey what exists, discover the project profile, and emit a
dependency-ordered, loop-protocol-compliant backlog. It does **not** run the
loop — that's `/loop`.

Read `SCHEMA.md` (next to this file) first; it is the contract you generate
against. Generate against the schema version pinned there.

## When invoked

### `init` (default) — generate a new BACKLOG.md

1. **Survey.** Identify the repo(s) in scope. For each: read README/CLAUDE.md/docs, find any specs or roadmap/TODO/FIXME, check `git log` and open PRs/issues, and infer the tech stack and how tests/lint run. Fan this out with parallel read-only agents when there's more than one repo or a large tree — you want the conclusions, not the file dumps.
2. **Discover the project profile.** Concretely determine, by running things where safe: the exact test/lint command, the default branch per repo (watch for repos that differ), any *pre-existing* test failures (run the suite on a clean checkout so the loop knows what to ignore), merge style and whether merging is authorized, repo-specific hard rules (privacy/IAM invariants, config-pair lockstep, "never log X"), and where canonical specs live + whether they're reachable from this machine. This block is what makes the loop reliable — invest here.
3. **Draft items.** Convert discovered work into `B-NNN` items with real `depends_on` edges and concrete `acceptance`. Group into phases (housekeeping/quick wins first, then dependency-ordered features). Mark `human_gate: true` on anything irreversible, billable, outward-facing, or a business decision. Be honest about `blocked` items (e.g. specs that live only on another machine).
4. **Write `BACKLOG.md`** at the workspace/repo root: loop protocol (verbatim from SCHEMA, version-pinned) → project profile → items.
5. **Hand off.** Tell the user the one-line command to start it:
   `/loop Work the next eligible item per the loop protocol in <path>/BACKLOG.md`
   and which items are `human_gate`/`blocked` so they know what needs them.

### `rescan` / `update` — refresh an existing BACKLOG.md

Re-survey, append newly-discovered work as new `B-NNN` items, refresh the
project profile if the toolchain changed, and **never** rewrite or renumber
existing items or flip their status — the loop owns status transitions.

## Principles

- **One opinionated default, no flavors.** Don't add config knobs nobody asked for.
- **The acceptance criteria are the contract** — write them concrete and checkable, not "make it good".
- **Don't oversell.** If work can't be specced without information you don't have, mark it `blocked` with what's missing rather than inventing scope.
- **Scale to the ask.** A small repo gets a short backlog; don't manufacture phases.
