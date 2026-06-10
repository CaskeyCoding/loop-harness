# Changelog

Schema versions follow semver. Generated `BACKLOG.md` files pin the version they
were written against, so loops know which rules apply.

## 1.0.0 — 2026-06-09

Initial extraction from the CaskeyCoding backlog run (17 items, 19 merged PRs
across 5 repos). Establishes:

- Item schema: `repo`, `status`, `depends_on`, `size`, `human_gate`, `acceptance`, `pr`, `notes`.
- Six-step loop protocol with verify-before-merge and human-gate skipping.
- The **project profile** block — discovered-once context (checks, default branch, known failures, merge authorization, hard rules, spec source).
- The `backlog` skill: `init` (survey → generate) and `rescan`/`update`.
