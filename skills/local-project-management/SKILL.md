---
name: local-project-management
description: |
  File-based task tracking for a project that spans multiple repos or codebases coordinated from a single top-level folder. Use this skill when (1) starting a new project that will have multiple plans / phases / subsystems, (2) the user asks to "set up task tracking", "track tasks across repos", "create a plan", "keep tasks in docs", "make a master index", or anything similar, (3) the user asks to add a new plan or sub-task to an existing project that already uses this layout (`docs/plans/`, `docs/inprogress/`, `docs/completed/`, `docs/tasks-index.md`), (4) work is splitting across `codebase/repo-a/`, `codebase/repo-b/` … and the user wants planning kept outside the code folders. Skill is a directory-level scaffold + ongoing discipline, NOT a single-shot command.
---

# Project Task Tracker

For projects that contain multiple repos / codebases under one top-level folder and need a single source of truth for plans and progress.

The goal: every piece of work has a written plan with a stable identifier, every sub-task can be traced from a master index, and status changes never live only in chat.

## When to invoke

- User asks to "set up task tracking", "plan this", "track work across these repos", "create a master tasks list", "set up the docs structure", or anything in that family.
- User starts a new substantive project that will span multiple plans (more than one feature/migration).
- User asks to add a new plan to an existing project that already has `docs/plans/` and `docs/tasks-index.md`.
- User asks to move an existing plan forward (Plan → In Progress, or sub-task → Done) — you sync the three files per the rule below.
- You're about to start ANY non-trivial implementation in a project that uses this layout — write the plan FIRST.

Skip if:
- The project is single-file or a one-shot script.
- The user has their own established system (respect it; don't impose this one).

## The layout

```
<project-root>/
├── CLAUDE.md                       ← project rules (Plan-First Rule lives here)
├── docs/
│   ├── tasks-index.md              ← master index of every plan + sub-task
│   ├── plans/                      ← plan folders not yet started
│   ├── inprogress/                 ← plan folders currently active
│   │   └── NN-plan-slug/
│   │       ├── 00-overview.md      ← context, decisions, Status table, Plan/In Progress/Done
│   │       ├── 01-first-task.md    ← per-sub-task file
│   │       ├── 02-second-task.md
│   │       └── …
│   └── completed/                  ← plan folders fully shipped
└── codebase/                       ← one folder per repo this project spans
    ├── repo-a/
    └── repo-b/
```

## Numbering scheme — `#P.N`

- Plans get a stable integer `P`, monotonically assigned across the project, embedded in the folder name (`01-web-showcase`, `02-…`, `03-…`). Never reused, never renumbered.
- Sub-tasks within a plan get `#P.N` — e.g. Plan #1's third sub-task is `#1.3`. Numbers are assigned the moment an item appears in the plan's **Plan** section and never change as the item moves between sections.
- Per-sub-task filename: `NN-slug.md` where `NN` is the two-digit sub-task number (`03-releases-lib.md`).
- Commits, PR titles, follow-up references all use `#P.N`. Example commit: `#1.5: add /pricing route`. Example follow-up: "carries over from #1.7".
- Splitting a sub-task: keep the original number, give the new piece the next free number within the plan. Dropping: leave the number burned, never reuse.

## Per-plan file structure

Every `00-overview.md` has these sections, in this order:

```markdown
# Plan #P — <Plan title>

<one-paragraph context>

## Status

| #     | Task                    | Status         | File |
| ----- | ----------------------- | -------------- | ---- |
| #P.1  | <one-line title>        | ✅ Done        | [01-slug.md](01-slug.md) |
| #P.2  | <one-line title>        | 🔄 In Progress | [02-slug.md](02-slug.md) |
| #P.3  | <one-line title>        | ⬜ Plan        | _(file when started)_ |

Legend: ⬜ Plan · 🔄 In Progress · ✅ Done · ⛔ Blocked · 🗑 Dropped

## Context
## Architecture decisions (locked)
## Reference links

## Plan
- [ ] #P.3 — <outcome, with acceptance criteria>

## In Progress
- [~] #P.2 — <outcome> — see [02-slug.md](02-slug.md)

## Done
- [x] #P.1 — <what shipped> · <follow-up note, if any>

## Out of scope (deliberately)
## Retro / Issues found
```

Per-sub-task file structure:

```markdown
# #P.N — <Sub-task title>

Parent plan: [00-overview.md](00-overview.md)
Status: 🔄 In Progress | ✅ Done (<date>)

## Acceptance
<concrete criteria — the things that must be true to mark this done>

## Plan for this task
<numbered list of steps>

## Notes / decisions
<inline reasoning, refinements vs. parent plan, trade-offs>

## Reference links
<populated as work happens — every link, command, screenshot path, URL>

## What shipped
<after completion — concrete list>

## Verification
<after completion — how you proved acceptance>

## Issues found
<after completion — surprises, follow-ups, anything the next plan should know>
```

## Master index — `docs/tasks-index.md`

The single project-wide view. Every plan + every sub-task appears with its current status. Must include:

- A "Plans" table (one row per plan, with a progress fraction).
- A per-plan "Sub-tasks" table with `#P.N`, title, status emoji, link to the per-sub-task file.
- A "Live URLs" section if the project has any (production, dashboards, repos).
- The sync rule (below) reproduced at the bottom of the file as a reminder.

## The three-place sync rule

Whenever a sub-task changes status (Plan → In Progress → Done, or any → Blocked):

1. **Per-sub-task file** — update the `Status:` line in the header.
2. **Parent plan's `00-overview.md`** — move the bullet between **Plan / In Progress / Done** sections AND update the row in the Status table.
3. **`docs/tasks-index.md`** — update the row in the per-plan sub-tasks table and the "Progress" fraction in the Plans table.

If you find drift between the three: per-sub-task file wins (most-recent local truth) → parent overview → index. Reconcile and continue — never leave drift unresolved.

## Bootstrap recipe

When starting fresh on a project that doesn't have this layout — **one paste each**:

1. `mkdir -p docs/plans docs/inprogress docs/completed`
2. Copy `references/tasks-index-template.md` from this skill folder to `docs/tasks-index.md` and fill in the first plan row.
3. Append the contents of `references/claude-md-snippet.md` to the project's `CLAUDE.md` (create it if missing). The snippet contains the Plan-First Rule, plan-folder structure spec, master-index pointer, and three-place sync rule — verbatim, generic.
4. Create the first plan folder: `docs/plans/01-<slug>/00-overview.md` using the `00-overview.md` skeleton from the snippet.
5. Move it to `docs/inprogress/01-<slug>/` the moment the first sub-task starts.

Both reference files live in this skill's `references/` directory — read them once, paste them in, and the project is bootstrapped.

## New-plan recipe

For a new plan in an already-set-up project:

1. Find the next free plan number `P` by listing `docs/plans/` and `docs/inprogress/` and `docs/completed/`.
2. Create `docs/plans/<P>-<slug>/00-overview.md` using the structure above. Number sub-tasks `#P.1, #P.2, …`.
3. Add a row to `docs/tasks-index.md`'s Plans table and a new sub-tasks table beneath it.
4. When work begins, move the folder to `docs/inprogress/`.

## New-sub-task recipe (mid-plan)

1. Pick the next free `#P.N` within the plan (don't reuse dropped numbers).
2. Add a `[ ] #P.N — <outcome + acceptance>` line under the plan's **Plan** section.
3. Add a row in the plan's **Status** table.
4. Add a row in `docs/tasks-index.md`'s per-plan table.
5. Create `docs/inprogress/<P>-<slug>/NN-<sub-slug>.md` ONLY when the sub-task moves to In Progress — not before. Plan items without their own file are fine; the per-sub-task file is created on activation.

## Mark-done recipe

When a sub-task hits its acceptance criteria:

1. Per-sub-task file: change `Status: 🔄 In Progress` → `Status: ✅ Done (<date>)`. Append **What shipped**, **Verification**, **Issues found** sections.
2. Parent `00-overview.md`: move the item from **In Progress** → **Done** (newest first in Done); update Status table emoji.
3. `docs/tasks-index.md`: update the row's emoji + bump the Plans-table progress fraction.

## Mark-plan-done recipe

When every sub-task in a plan is ✅:

1. Append a **Retro / Issues found** section to `00-overview.md` capturing surprises, follow-ups, decisions.
2. Move the entire plan folder from `docs/inprogress/<P>-<slug>/` → `docs/completed/<P>-<slug>/`.
3. Update `docs/tasks-index.md` Plans-table progress to `N/N done` and the link path.
4. Spin out any follow-ups as a new plan (next free `P`) with cross-links to the completed plan.

## Multi-repo nuance

Plans live at the PROJECT root in `docs/`, never inside `codebase/<repo>/`. The point of this layout is one coordinated planning surface for work that touches multiple repos.

A single sub-task is free to touch multiple repos — write the per-sub-task file in `docs/inprogress/<P>-<slug>/NN-….md` and reference the affected files in `codebase/repo-a/...` and `codebase/repo-b/...` in the **What shipped** section.

If a sub-task is genuinely scoped to one repo's internals and would not benefit from project-level visibility, fine — but the plan that birthed it still lives in project-level docs/.

## Anti-patterns

- **Skipping the plan file because "this is simple."** No. Even a five-line sub-task gets a one-paragraph plan file. Every implementation decision has compounding context the next session needs.
- **Updating only one of the three sync places.** Causes drift that compounds. Always update all three.
- **Renumbering after dropping a sub-task.** Breaks every external reference (commits, PRs, follow-ups). Leave the number burned.
- **Stuffing many sub-tasks into one file because they're "small."** Batching IS allowed when items genuinely ship together (e.g., four similar legal pages) — but the parent plan's Status table still shows each `#P.N` as its own row, even if they share a file.
- **Putting links and decisions in chat instead of the per-sub-task file.** Chat scrollback is lossy. The plan file is canonical.

## Example

Real example of this skill applied to a multi-repo product is in the Solstice project: see `docs/tasks-index.md` at the project root and `docs/inprogress/01-web-showcase/` for the per-plan + per-sub-task files. The CLAUDE.md there has the Plan-First Rule section that goes alongside this skill.
