# BACKLOG schema & loop protocol

**Schema version: 1.1.1** (additive over 1.0.0; a 1.0.0 file is still valid)

This is the contract a `BACKLOG.md` file follows so an agent loop (e.g. Claude
Code's `/loop`) can drive it autonomously: pick the next unblocked item, do it,
verify, merge, update status, repeat. The protocol travels *inside* the
generated `BACKLOG.md` header so the loop is self-describing: the driving
prompt only needs to say "work the next eligible item per the loop protocol in
BACKLOG.md".

---

## File layout

A `BACKLOG.md` has three parts, in order:

1. **Loop protocol**: the run rules (copied verbatim from §"Loop protocol" below, pinned to a schema version).
2. **Project profile**: the per-project context the loop needs but shouldn't rediscover each iteration (§"Project profile").
3. **Items**: dependency-ordered work, grouped into phases (§"Item schema").

---

## Item schema

Each item is a markdown `###` heading `B-NNN <title>` followed by a field list:

```markdown
### B-010 Port budget enforcement into the main repo
- repo: caskey
- status: ready
- depends_on: []
- size: M
- tier: build              # optional; judge | build | mechanical, default build
- spec: specs/api/002-health.md   # optional; the spec this item serves
- human_gate: false        # optional; omit when false
- acceptance: <one or more concrete, checkable criteria>
- pr: <url>                # added when opened
- notes: <free text>       # provenance, caveats, follow-ups
```

| Field | Required | Values / meaning |
|---|---|---|
| `repo` | yes | Which repo (or comma-list) the work lands in. |
| `status` | yes | `ready` \| `in_progress` \| `in_review` \| `done` \| `blocked` \| `draft`. The short names `doing` and `review` (as in Lesson 6's template) are accepted aliases of `in_progress` and `in_review`. |
| `spec` | no | Path to the spec the item serves, when one exists (`specs/api/002-health.md`). The loop reads it before starting and grades the result against its acceptance criteria. |
| `depends_on` | yes | List of `B-NNN` ids that must be `done` first. `[]` = none. |
| `size` | yes | `XS` \| `S` \| `M` \| `L`. Rough effort, used only for ordering intuition. |
| `tier` | no | `judge` \| `build` \| `mechanical`. Which model the item deserves: `judge` when the acceptance itself needs a decision (an investigate item), `build` for a checkable acceptance line, `mechanical` when the steps are already enumerated in the item. Default `build`. The expensive model judges; the cheap one types. |
| `human_gate` | no | `true` if a human must act before the loop may start it (irreversible, billable, outward-facing, or a business decision). Default `false`. |
| `acceptance` | yes | Concrete, checkable done-condition(s). Vague criteria produce vague work. |
| `pr` | no | PR URL, added when the item enters `in_review`/`done`. |
| `notes` | no | Provenance ("discovered during B-003"), provisional choices, caveats for the human. |

**Status lifecycle:** `ready → in_progress → in_review → done` (or `ready → doing → review → done` with the short names). `blocked` and
`draft` are holding states the loop skips. An item discovered to be already done
or wrong is set `done` with a `notes:` explanation rather than worked. `done`
means merged, never "the agent said so". The loop changes status only on the
item it holds, except for confirmed-merge reconciliation in the protocol.

### Investigate items

Work you cannot write an acceptance line for yet still gets an item. Give it
`tier: judge` and this acceptance:

```markdown
- acceptance: done when it has produced child items B-NNN, each with a
  checkable acceptance, and it has asked nothing it could have found in the
  repository
- notes: an investigate item; its output is items, not code
```

The loop works it like any other item: the output is new items appended at the
bottom with `status: draft`, no code. Commit these changes to the versioned
backlog and open a PR under the usual merge policy; the parent is `done` when
that PR merges. The children stay `draft` until a human reads them, strikes or
edits, and marks the survivors `ready`. This is how a queue decomposes itself
without a human writing every item by hand.

---

## Loop protocol

> Copy this section verbatim into the head of every generated `BACKLOG.md`,
> with `(schema 1.1.0)` after the heading.

Each iteration:

1. Re-read this file and reconcile merged PRs as in step 4. Pick the **lowest-numbered item with `status: ready` whose `depends_on` are all `done` and whose `human_gate` is not `true`**.
   If none qualifies, stop as in step 6.
   Before starting it, if its `pr` is empty, search the default branch history for its id
   (`git log origin/<default> --oneline --grep=B-NNN`). If the work already merged, set the item
   `done` with a note and pick again. The file is the program; a stale file makes the loop
   redo finished work faithfully.
2. Set it `status: in_progress`, do the work in the item's repo on a branch named `<branch-prefix>/B-NNN` (see project profile), with the project's checks green.
3. Open a PR referencing `B-NNN`. If the project profile authorizes merging and the change is test-green and matches the item's acceptance: merge it, delete the branch, pull the default branch, set `status: done`. Otherwise leave it `in_review` with the PR URL for a human.
4. On a later iteration, if an `in_review` item's PR has been merged, set it `done` (and pull the default branch in that repo).
5. **Never** start an item with `human_gate: true`; surface it to the human and pick the next eligible item instead.
6. Stop when no eligible item remains after reconciliation: no `ready` item has all dependencies `done` and `human_gate` other than `true`. This includes queues waiting on `draft` items or unmet dependencies. Report the standstill and what needs human action (and, if running unattended, send one summary notification).

**Rules:**
- One item per iteration; never batch.
- Don't edit items you aren't working, except status and notes updates for confirmed merged work in steps 1 and 4.
- Respect each repo's `CLAUDE.md` and any hard rules named in the project profile.
- Verify before merging: run the project's checks yourself; a known pre-existing failure listed in the profile is ignorable, anything else is not.
- If an item turns out to be wrong or already done, mark it `done` with a note rather than doing makework.
- When a finished item reveals new work, append it as a new `B-NNN` (don't silently expand the current item's scope).
- Investigate items append children with `status: draft`; commit them to the versioned backlog and open a PR under the project merge policy. Merging closes the parent; only a human promotes its children to `ready`.

---

## Project profile

The block the generator fills by surveying the repo(s). This is the context
that lets the loop run without rediscovering the same facts every iteration;
it's the difference between a loop that works and one that re-greps the test
command 18 times.

```markdown
## Project profile
- repos: <list> (default branch per repo; note any that differ, e.g. sandbox uses `main`)
- branch_prefix: backlog        # branches are <prefix>/B-NNN
- checks: <exact command to run tests/lint, per repo>
- known_failures: <pre-existing failures to ignore, with why> (e.g. "test_integration_real_claude: needs the real CLI, fails on default branch too")
- merge: <authorized | human-only> + merge style (merge commit | squash) + how branch protection behaves
- hard_rules: <repo-specific invariants the loop must not violate> (e.g. IAM-enforced privacy rules, config-pair lockstep, never log secrets)
- spec_source: <where canonical specs live, if any, and whether they're reachable from this machine>
```

The `merge: authorized` line is what lets the loop close items end-to-end; omit
it (or set `human-only`) and every item parks at `in_review`.

---

## Versioning

Schema changes are semver'd in `CHANGELOG.md`. Generated `BACKLOG.md` files pin
the version they were written against (`Loop protocol (schema X.Y.Z)`), so a
loop reading an older file knows which rules apply. Bump **minor** for additive
fields, **major** for changes that break an existing file's meaning.
