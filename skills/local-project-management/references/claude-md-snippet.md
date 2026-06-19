<!--
APPEND THIS BLOCK TO THE PROJECT'S CLAUDE.md DURING BOOTSTRAP.

Trim or rename headings to fit existing CLAUDE.md tone. The rules themselves
should not be softened — they exist because skipping them produces drift that
compounds over weeks.
-->

## Plan-First Rule

**No code, scaffolding, config, or destructive action runs without a plan file.** This is non-negotiable.

### Before starting any task

1. If a matching plan file does not exist in `docs/plans/` or `docs/inprogress/`, **stop and write one first**. Even a one-paragraph plan for a small change.
2. Move the plan folder from `docs/plans/<P>-<slug>/` → `docs/inprogress/<P>-<slug>/` the moment work begins.
3. Keep the per-sub-task file open as the working scratchpad for the whole task — paste every link, command, screenshot path, and decision into it as you encounter them. Chat scrollback is lossy; the plan file is canonical.

### Plan + sub-task numbering — `#P.N`

- Plans get a stable integer `P`, monotonic across the project, embedded in the folder name (`01-<slug>`, `02-<slug>`). Never reused, never renumbered.
- Sub-tasks within a plan get `#P.N` (e.g. Plan #1's third sub-task is `#1.3`). Numbers are assigned the moment an item appears in **Plan** and never change as the item moves between sections.
- Per-sub-task filename: `NN-<slug>.md` where `NN` is the two-digit sub-task number.
- Commits, PR titles, follow-up plans all reference `#P.N`.
- Splitting a sub-task: keep the original number, give the new piece the next free number within the plan. Dropping: leave the number burned.

### Required plan-folder structure

```
docs/inprogress/<P>-<slug>/
├── 00-overview.md          ← context, decisions, Status table, Plan/In Progress/Done
├── 01-<slug>.md            ← per-sub-task file, created when sub-task starts
├── 02-<slug>.md
└── …
```

`00-overview.md` skeleton:

```markdown
# Plan #P — <Plan title>

<one-paragraph context>

## Status

| #     | Task              | Status         | File |
| ----- | ----------------- | -------------- | ---- |
| #P.1  | <one-line title>  | ✅ Done        | [01-slug.md](01-slug.md) |
| #P.2  | <one-line title>  | 🔄 In Progress | [02-slug.md](02-slug.md) |
| #P.3  | <one-line title>  | ⬜ Plan        | _(file when started)_ |

Legend: ⬜ Plan · 🔄 In Progress · ✅ Done · ⛔ Blocked · 🗑 Dropped

## Context
## Architecture decisions (locked)
## Reference links

## Plan
- [ ] #P.3 — <outcome, with acceptance criteria>

## In Progress
- [~] #P.2 — <outcome> — see [02-slug.md](02-slug.md)

## Done
- [x] #P.1 — <what shipped> · <follow-up note>

## Out of scope (deliberately)
## Retro / Issues found
```

Per-sub-task file skeleton:

```markdown
# #P.N — <Sub-task title>

Parent plan: [00-overview.md](00-overview.md)
Status: 🔄 In Progress | ✅ Done (<date>)

## Acceptance
## Plan for this task
## Notes / decisions
## Reference links
## What shipped
## Verification
## Issues found
```

## Master Task Index

`docs/tasks-index.md` is the single project-wide view of every plan + sub-task and its current status. **Update it in lockstep** with the per-sub-task file and the parent plan's overview whenever a sub-task changes state.

### The three-place sync rule

Whenever a sub-task changes status (Plan → In Progress → Done, or any → Blocked):

1. **Per-sub-task file** — update the `Status:` line.
2. **Parent plan's `00-overview.md`** — move the bullet between Plan / In Progress / Done AND update the Status table emoji.
3. **`docs/tasks-index.md`** — update the row's emoji + the Plans-table progress fraction.

If you find drift between the three: per-sub-task file wins (most recent local truth) → parent overview → index. Reconcile and continue — never leave drift unresolved.

### After a sub-task ships

- Append **What shipped**, **Verification**, **Issues found** sections to the per-sub-task file.
- Carry follow-ups into a new plan (cross-link them) so the next iteration starts with a better plan.

### After a whole plan ships

- Append a **Retro / Issues found** section to `00-overview.md`.
- Move the folder from `docs/inprogress/<P>-<slug>/` → `docs/completed/<P>-<slug>/`.
- Update `docs/tasks-index.md` Plans-table progress + path.
