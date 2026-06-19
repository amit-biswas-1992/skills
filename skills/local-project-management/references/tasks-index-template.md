# Master Task Index

Single page that shows every numbered task across every plan in the project. Update this any time a sub-task changes status or a new plan is created.

Legend: ⬜ Plan · 🔄 In Progress · ✅ Done · ⛔ Blocked · 🗑 Dropped

## Plans

| Plan | Title | Location | Progress |
| ---- | ----- | -------- | -------- |
| #1   | <Plan title> | [docs/plans/01-<slug>/](plans/01-<slug>/) | 0 / N done |

## Sub-tasks

### Plan #1 — <Plan title>

| #     | Task                | Status | File |
| ----- | ------------------- | ------ | ---- |
| #1.1  | <one-line title>    | ⬜ Plan | _(file when started)_ |

## Live URLs

_(populate when the project has production / staging / dashboards / repos)_

## Sync rule

Whenever a sub-task changes status (Plan → In Progress → Done):

1. The sub-task's own file header (`Status:` line).
2. The parent plan's `00-overview.md` Status table + Plan / In Progress / Done sections.
3. This file.

If you find drift between the three, the per-task file wins (most-recent local truth), then the parent overview, then this index. Reconcile and move on — do not leave drift unresolved.
