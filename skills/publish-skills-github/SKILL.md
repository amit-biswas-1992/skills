---
name: publish-skills-github
description: Use when publishing a Claude Code / Cursor / Codex / OpenCode agent skill to skills.sh — creating the public GitHub repo, pushing the SKILL.md, and registering it on the leaderboard via the first `npx skills add`. Triggers include — "publish my skill", "upload skill to skills.sh", "list my skill on skills.sh", "share this skill publicly", "make this skill discoverable", "put this on the skills leaderboard", "publish-skills-github". Also use when the user has a SKILL.md in `~/.claude/skills/<name>/` they want to make available to other AI agents and users.
---

# Publishing an Agent Skill to skills.sh via GitHub

## TL;DR

skills.sh is GitHub-backed, not npm-backed. To publish a skill:

1. **Create a public GitHub repo** with `SKILL.md` at the root (single-skill repo) or under `skills/<name>/SKILL.md` (mono-repo).
2. **Run `npx skills add <owner>/<repo>` once** — this fires the telemetry that adds your repo to the leaderboard.
3. Subsequent users of `npx skills add <owner>/<repo>` bump your install count.

There is no submission form, approval queue, or registry tarball. Push to GitHub, run the install once, you're listed.

## Decision: dedicated repo vs mono-repo — default to mono-repo

| | Dedicated repo (`<user>/<skill-name>`) | Mono-repo (`<user>/skills`) ← **default** |
|---|---|---|
| **Structure** | `SKILL.md` at root | `skills/<name>/SKILL.md` per skill |
| **Install command** | `npx skills add <user>/<skill-name>` | `npx skills add <user>/skills` (installs all) or `--skill <name>` (one) |
| **skill-detail URL on skills.sh** | `/<user>/<skill>/<skill>` ← name doubled in the breadcrumb | `/<user>/skills/<skill>` ← clean |
| **Adding a second skill later** | Spin up another repo, deal with archive/redirect of the first to migrate | Drop a folder under `skills/`, push, done |
| **Stars / issues** | Per skill (useful only if a skill develops its own community) | Consolidated on the mono-repo |
| **Best when** | The skill genuinely needs its own GitHub presence — code repository, complex codebase, third-party PRs | Anyone with more than one skill, or who expects to publish more later |

**Default to mono-repo `<user>/skills`.** The dedicated-repo pattern forces the repo name to equal the skill name, which makes the skill-detail URL on skills.sh repeat the name (`/<user>/<skill>/<skill>`) and clutters your GitHub profile if you publish a second skill. The mono-repo gives clean URLs and one canonical home for everything you publish.

The dedicated-repo flow below is included for the rare case where a skill needs its own GitHub presence — most users should jump to the **mono-repo flow** further down.

## Workflow (dedicated repo — only when you really need a separate GitHub presence)

Assume the skill already exists at `~/.claude/skills/<name>/SKILL.md` and works locally.

### 1. Set up the repo directory

```bash
SKILL_NAME=<skill-name>                          # the skill folder name in ~/.claude/skills/
GH_USER=$(gh api user --jq .login)               # auto-discover from authenticated gh CLI

REPO_DIR="$HOME/code/${SKILL_NAME}-skill"        # adjust to wherever you keep code repos
mkdir -p "$REPO_DIR"
cd "$REPO_DIR"

# Copy the canonical SKILL.md
cp ~/.claude/skills/$SKILL_NAME/SKILL.md ./SKILL.md
```

The repo folder is named `<skill>-skill` (with `-skill` suffix) so it doesn't collide locally with any companion code repo of the same name. The GitHub repo will be named just `<skill>` (no suffix). Pre-check that `gh auth status` returns OK before relying on `gh api user`.

### 2. Add LICENSE, .gitignore, README

`SKILL.md` is the substance, but a complete skill repo needs:

- **`LICENSE`** — MIT is the convention. Skills are read by AI agents and may be reproduced verbatim, so a permissive license matters.
- **`.gitignore`** — at minimum: `node_modules/`, `*.log`, `.DS_Store`. Skills don't need build output.
- **`README.md`** — short. Describe what the skill does, the install command, what triggers should activate it, and link to any companion package (npm, etc.). Add a skills.sh badge:
  ```markdown
  [![skills.sh](https://skills.sh/b/<user>/<repo>)](https://skills.sh/<user>/<repo>)
  ```

The README is what visitors see on GitHub; the SKILL.md is what AI agents read. They are different audiences.

### 3. SKILL.md sanity check before pushing

The SKILL.md must have YAML frontmatter that skills.sh / agent CLIs read. The minimum:

```yaml
---
name: react-native-latex-math
description: Use when … Triggers include — "phrase 1", "phrase 2", … . Also use when …
---
```

- **`name`** must match the directory/repo name exactly (kebab-case).
- **`description`** must be a single line, but pack it with **trigger phrases**. AI agents match on these — vague descriptions never get invoked. Include the exact phrases users would say (in quotes), the failure-mode symptoms, and any synonyms for the technology.

If the description was written hastily ("Use when working with X"), rewrite it before publishing. The leaderboard ranks by installs, but discovery starts with description matches.

### 4. Init git and push to GitHub

```bash
cd "$REPO_DIR"
git init -b main
git add -A
# Uses your already-configured global git user.name / user.email.
# Verify with `git config --global user.name` and `git config --global user.email`.
git commit -m "Initial: ${SKILL_NAME} skill"

# Create the public GitHub repo and push in one shot.
# Omitting `${GH_USER}/` defaults to your authenticated user.
gh repo create "${SKILL_NAME}" \
  --public \
  --description "AI agent skill: <one-line summary>. Install via 'npx skills add ${GH_USER}/${SKILL_NAME}'." \
  --source . \
  --remote origin \
  --push
```

You'll get back the GitHub URL: `https://github.com/<user>/<skill-name>`. Add `--homepage <url>` if there's a companion package, docs site, or demo.

### 5. Register on skills.sh (the install)

The leaderboard only knows about repos that have been installed at least once. Run the install yourself to seed it:

```bash
cd /tmp   # don't pollute any project dir
npx --yes skills add "${GH_USER}/${SKILL_NAME}" \
  --agent "claude-code" \
  --global \
  --yes
```

Flags that matter:
- `--agent claude-code` — installs only for Claude Code (otherwise the CLI prompts for which agents to install across, interactive). Adjust to your primary agent. `cursor`, `codex`, `opencode`, `windsurf` etc. are also valid.
- `--global` — installs to your user-level skill dir, not project-level. Avoids tying it to whatever cwd you happened to be in.
- `--yes` — skip confirmation prompts so the command can run non-interactively.

You'll see output like:
```
✓ react-native-latex-math (copied)
  → ~/.claude/skills/react-native-latex-math
```

Note: this overwrites any existing local copy of the skill in `~/.claude/skills/<name>/`. If you've been editing the local copy in parallel with the GitHub one, sync them first or you'll lose changes.

### 6. Verify the listing

```bash
sleep 30                                                     # let telemetry ingest
curl -sI "https://skills.sh/<user>/<skill-name>" | head -1   # should be HTTP 200
```

Or open the URL directly: `https://skills.sh/<user>/<skill-name>`. You'll see:

- The fully-qualified name `<user>/<skill-name>`
- Skill count (1 for a dedicated repo)
- Install count (1, after your seed install)
- The install command with copy button
- Link out to the GitHub repo

## Updating a published skill

Just edit and push:

```bash
cd ~/Work/My\ Work/Personal/<skill-name>-skill
$EDITOR SKILL.md
git add SKILL.md
git commit -m "Improve <section>: <what changed>"
git push
```

Consumers run `npx skills update` (in their project) or `npx skills update -g` (globally) to pull your new version.

You typically don't need to bump a version number — there's no semver in skills.sh. Each commit on `main` is the new "version".

## Mono-repo flow (the default — and the right choice for almost everyone)

The repo name is literally `skills`. Every skill is a subfolder. The structure scales — first skill, fifth skill, fiftieth skill all use the same layout.

```
<user>/skills/
├── README.md          ← lists what's inside, install commands, links to each skill
├── LICENSE            ← MIT
├── .gitignore         ← node_modules/, *.log, .DS_Store
└── skills/
    ├── publish-skills-github/SKILL.md
    ├── react-native-latex-math/SKILL.md
    └── npm-publish/SKILL.md
```

### Setup (first time only)

```bash
GH_USER=$(gh api user --jq .login)
REPO_DIR="$HOME/code/skills"               # adjust to where you keep code repos
mkdir -p "$REPO_DIR/skills"
cd "$REPO_DIR"

# Top-level files
$EDITOR README.md   # describe the collection — see template below
$EDITOR LICENSE     # MIT, your name + year
echo -e "node_modules/\n*.log\n.DS_Store" > .gitignore

# Add the first skill
SKILL_NAME=<skill-name>
mkdir -p skills/$SKILL_NAME
cp ~/.claude/skills/$SKILL_NAME/SKILL.md skills/$SKILL_NAME/SKILL.md

# Init git, create the public repo, push
git init -b main
git add -A
git commit -m "Initial: ${SKILL_NAME}"
gh repo create skills \
  --public \
  --description "Reusable AI agent skills (Claude Code, Cursor, Codex, ...). Install all via 'npx skills add ${GH_USER}/skills' or one via '--skill <name>'." \
  --source . --remote origin --push

# Register on the leaderboard — install each skill once
cd /tmp
npx --yes skills add ${GH_USER}/skills --skill '*' --agent claude-code --global --yes
```

### Adding a second / Nth skill

```bash
cd $REPO_DIR
SKILL_NAME=<new-skill-name>
mkdir -p skills/$SKILL_NAME
cp ~/.claude/skills/$SKILL_NAME/SKILL.md skills/$SKILL_NAME/SKILL.md
# Update README.md to add a row for the new skill
git add .
git commit -m "Add skill: $SKILL_NAME"
git push

# Register the new skill on the leaderboard
cd /tmp
npx --yes skills add ${GH_USER}/skills --skill $SKILL_NAME --agent claude-code --global --yes
```

That's the entire flow for adding skills going forward — drop a folder, push, run one install. No new repos, no new GitHub URLs.

### Top-level README template

```markdown
# skills

[![skills.sh](https://skills.sh/b/<user>/skills)](https://skills.sh/<user>/skills)

A collection of reusable AI agent skills. Compatible with Claude Code, Cursor, Codex, OpenCode, Windsurf, and any other agent that consumes [skills.sh](https://skills.sh).

## Install all
\`\`\`bash
npx skills add <user>/skills
\`\`\`

## Install one
\`\`\`bash
npx skills add <user>/skills --skill <skill-name>
\`\`\`

## What's in here

| Skill | What it does | Trigger phrases |
|---|---|---|
| [\`skill-a\`](./skills/skill-a) | one-line summary | "trigger 1", "trigger 2" |
| [\`skill-b\`](./skills/skill-b) | one-line summary | "trigger 1", "trigger 2" |
```

### Migrating from dedicated repos to a mono-repo

If you've already published one or more skills as dedicated repos and want to consolidate (likely because the doubled `/<user>/<skill>/<skill>` URL bothers you, or because publishing the second skill made the redundancy obvious):

1. Create the mono-repo as above with all your existing SKILL.md files moved into `skills/<name>/`.
2. `npx skills add <user>/skills --skill '*' --agent claude-code --global --yes` — registers each skill.
3. For each old dedicated repo:
   - Replace its README with a "moved" notice pointing to the mono-repo install command (so anyone visiting the GitHub page knows where to go)
   - Push the README change
   - `gh repo archive <user>/<old-repo> --yes` — marks the repo read-only, signals deprecation, but keeps URLs working so existing `npx skills add <old-repo>` doesn't 404
4. The old repos' install counts on skills.sh stay frozen at their pre-migration values; new installs accrue on the mono-repo. You don't get to merge the counts.

The leaderboard shows individual skills under the mono-repo namespace (e.g. `react-native-latex-math` appears as a separate row inside the `<user>/skills` listing — clicking lands on `/<user>/skills/<skill>`).

## Common pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| Listing page 404s after install | Telemetry not yet ingested | Wait 30–60s, retry. Persistent 404 means the install didn't actually run — check the CLI output for an error. |
| `npx skills add` hangs at "Additional agents" prompt | Missing `--agent <x> --yes` flags | Re-run with `--agent claude-code --global --yes` (or your primary agent). |
| Description in SKILL.md is short / unspecific | Frontmatter description was a quick draft | Rewrite to include 3–5 trigger phrases in quotes + failure-mode symptoms. Description quality is the gating factor for discovery. |
| Listing shows 0 installs but you ran the install | The CLI may have errored silently — install only counts if it completed successfully | Re-run with `--yes`, watch for the "✓ <name> (copied)" confirmation. |
| Want to delete a stale skill | skills.sh has no admin UI to delete a listing — but archiving/deleting the GitHub repo will eventually de-index it | Archive the repo on GitHub. New `npx skills add` to that path will fail. |
| Skill name collision with someone else's listing | The leaderboard shows multiple skills with the same `<name>` (e.g. three different `npm-publish` skills) | This is expected — listings are namespaced by `<user>/<repo>`. Pick a clear name; description quality wins discovery. |

## What NOT to put in the public repo

- **Personal API tokens** of any kind. The skill is text — it should never contain `npm_…`, `gh_…`, `op://…` URIs that resolve to private items, etc.
- **Internal company workflows or stack details** unless you intend them as public reference.
- **Code that depends on a specific filesystem layout** (e.g. `/Users/hello/...`) — generalize before publishing.
- **License-incompatible content** (don't paste copyrighted prose into SKILL.md).

A practical check before pushing — adjust the regex with your own personal-info markers (your name, email-username, GitHub-username, employer, etc.):

```bash
grep -iE "(token|api[_-]?key|secret|password|/Users/[^/]+/|<your-name>|<your-gh-handle>)" SKILL.md README.md
```

If it returns anything that isn't documentation about NOT including such things, scrub before commit.
