# AGENTS.md

Skill-definition repo. No code, build, test, lint, CI, or package manager. Only Markdown.

## Structure

- `SKILL.md` — the skill (frontmatter `name`/`description` + Steps 0–8 release workflow). Entry point. Keep at root.
- `references/workflow-patterns.md` — release-automation recognition guide for Step 1.
- `references/changelog-format.md` — Keep a Changelog template for Step 4.
- `README.md` — human landing page with `npx skills add itsnyxdev/github-release`.
- `LICENSE` — MIT.

## Working here

- Edit Markdown directly; nothing to compile or run.
- Verify with `git diff` / `git status` only. No test suite exists.
- Keep `SKILL.md` step numbering stable — references files point at specific steps.
- `npx skills` discovery verified at root (`npx skills add ./ --list` → 1 skill). Don't move into `skills/` subdir — breaks install path.
- Skill's own rules (confirm-before-push, never invent changelog entries) apply when cutting a release of *other* projects, not when editing these docs. Confirm before push/tag/release here too.
- Remote `origin` is `git@github.com:itsnyxdev/github-release.git` (`main`, shipped `v1.0.0` via tag + `gh release create`).
