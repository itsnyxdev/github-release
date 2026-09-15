# AGENTS.md

Skill-definition repo. No code, build, test, lint, CI, or package manager. Only Markdown.

## Structure

- `SKILL.md` — the skill (frontmatter `name`/`description` + 9-step release workflow). Entry point.
- `references/workflow-patterns.md` — release-automation recognition guide for Step 1.
- `references/changelog-format.md` — Keep a Changelog template for Step 4.

## Working here

- Edit Markdown directly; nothing to compile or run.
- Verify with `git diff` / `git status` only. No test suite exists.
- Keep `SKILL.md` step numbering stable — references files point at specific steps.
- Skill's own rules (confirm-before-push, never invent changelog entries) apply when cutting a release of *other* projects, not when editing these docs.
- This repo itself is uncommitted (`main`, no commits yet, no remote). Don't assume push target exists.
