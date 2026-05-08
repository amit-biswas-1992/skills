# skills

[![skills.sh](https://skills.sh/b/amit-biswas-1992/skills)](https://skills.sh/amit-biswas-1992/skills)

A collection of reusable AI agent skills by Amit Biswas. Compatible with Claude Code, Cursor, Codex, OpenCode, Windsurf, and any other agent that consumes [skills.sh](https://skills.sh).

## Install all

```bash
npx skills add amit-biswas-1992/skills
```

## Install one

```bash
npx skills add amit-biswas-1992/skills --skill <skill-name>
```

## What's in here

| Skill | What it does | Trigger phrases |
|---|---|---|
| [`react-native-latex-math`](./skills/react-native-latex-math) | Render inline LaTeX math (`$...$`) inside React Native / Expo `<Text>` content — single-WebView KaTeX with a fast-path bypass for plain text. Includes the script-ordering bug fix that quietly kills KaTeX rendering. | "math is showing as raw text", "render KaTeX in React Native", "DEVELOPER_ERROR with $...$ in my exam app" |
| [`publish-skills-github`](./skills/publish-skills-github) | Walks through publishing your own SKILL.md to skills.sh via GitHub — repo creation, non-interactive `npx skills add` registration, leaderboard verification, personal-info pre-flight grep. | "publish my skill", "upload skill to skills.sh", "make this skill discoverable" |

Each skill has its own `SKILL.md` with the agent-readable frontmatter (name + description with trigger phrases) and full procedural content.

## Adding a new skill

1. Create `skills/<new-skill-name>/SKILL.md` with proper frontmatter (`name:` matching the folder, `description:` packed with trigger phrases).
2. Add a row to the table above.
3. `git push` — that's it. Existing installs pick up the new skill on `npx skills update`.

The companion skill [`publish-skills-github`](./skills/publish-skills-github) documents the full workflow including the personal-info pre-flight grep before pushing.

## License

MIT — see [LICENSE](./LICENSE).
