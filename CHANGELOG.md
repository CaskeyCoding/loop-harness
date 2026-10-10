# Changelog

Schema versions follow semver. Generated `BACKLOG.md` files pin the version they
were written against, so loops know which rules apply.

## 1.2.0, 2026-10-10

- New `drain` skill (`drain/SKILL.md`): one iteration of the loop as a skill. Reads the file, runs the freshness check, picks by the protocol, works the item in its own repo, verifies, sets `in_review` (or `review`), and stops with a four-part closing message. Investigate items append draft children instead of code. Blocks with one question and two answers rather than guessing. Proven on a four-repo scratch project: a build item drained and set to review; a run with nothing eligible said so and stopped; the next item drained after the human merged the first.
- README Quickstart gains the watched one-item step (`/drain`) before the self-paced loop (`/loop /drain`).

## 1.1.1, 2026-10-10

Additive. Lines up the schema with the Lesson 6 template so a reader can grow from the five-item file into this repo without renaming anything.

- `doing` and `review` accepted as aliases of `in_progress` and `in_review`.
- Optional `spec` field: the path of the spec an item serves.
- Repo homepage now points at Lesson 6.

## 1.1.0, 2026-10-06

Additive; every 1.0.0 file is still valid.

- `tier` field on items: `judge` | `build` | `mechanical`, default `build`. The expensive model judges, the cheap one types, and the choice is written on the item once.
- Investigate items: an item whose acceptance is to produce child items with checkable acceptance, appended as `draft` for a human to promote. The queue decomposes itself.
- Freshness check in step 1 of the loop protocol: search the default branch history for the item id before starting it; close it if the work already merged. Added after a loop picked three already-shipped items in a row because the file said `ready`.
- `done` means merged, including the backlog PR for an investigate item; its children remain `draft` until human promotion. The loop changes status only on the item it holds, except confirmed-merge reconciliation.
- Reconcile merged PRs before selecting an item, and stop when no eligible item remains, including queues waiting on drafts or unmet dependencies.
- No em dashes anywhere in the repo (house style).
- README points newcomers at Lesson 6 of the team-of-one series and explains the status, size, and heading changes needed to adopt the full schema.

## 1.0.0, 2026-06-09

Initial extraction from the CaskeyCoding backlog run (17 items, 19 merged PRs
across 5 repos). Establishes:

- Item schema: `repo`, `status`, `depends_on`, `size`, `human_gate`, `acceptance`, `pr`, `notes`.
- Six-step loop protocol with verify-before-merge and human-gate skipping.
- The **project profile** block: discovered-once context (checks, default branch, known failures, merge authorization, hard rules, spec source).
- The `backlog` skill: `init` (survey → generate) and `rescan`/`update`.
