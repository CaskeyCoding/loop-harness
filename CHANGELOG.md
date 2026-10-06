# Changelog

Schema versions follow semver. Generated `BACKLOG.md` files pin the version they
were written against, so loops know which rules apply.

## 1.1.0, 2026-10-06

Additive; every 1.0.0 file is still valid.

- `tier` field on items: `judge` | `build` | `mechanical`, default `build`. The expensive model judges, the cheap one types, and the choice is written on the item once.
- Investigate items: an item whose acceptance is to produce child items with checkable acceptance, appended as `draft` for a human to promote. The queue decomposes itself.
- Freshness check in step 1 of the loop protocol: search the default branch history for the item id before starting it; close it if the work already merged. Added after a loop picked three already-shipped items in a row because the file said `ready`.
- `done` means merged; only the iteration holding an item moves its status.
- No em dashes anywhere in the repo (house style).
- README points newcomers at Lesson 6 of the team-of-one series, which uses this schema's field names at five-item scale.

## 1.0.0, 2026-06-09

Initial extraction from the CaskeyCoding backlog run (17 items, 19 merged PRs
across 5 repos). Establishes:

- Item schema: `repo`, `status`, `depends_on`, `size`, `human_gate`, `acceptance`, `pr`, `notes`.
- Six-step loop protocol with verify-before-merge and human-gate skipping.
- The **project profile** block: discovered-once context (checks, default branch, known failures, merge authorization, hard rules, spec source).
- The `backlog` skill: `init` (survey → generate) and `rescan`/`update`.
