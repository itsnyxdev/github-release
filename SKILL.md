---
name: github-release
description: >-
  Use this skill whenever the user wants to cut, ship, publish, or prepare a
  new release/version of a project on GitHub — phrases like "release a new
  version", "cut a release", "publish vX.Y.Z", "ship this to GitHub", "tag a
  release", "update the changelog and release", or "bump the version". Also
  trigger if the user asks to set up or fix GitHub release automation (a
  release workflow, semantic-release, release-please, goreleaser). This
  skill enforces a safe, confirm-before-acting release process. It inspects
  any existing release workflow before touching it, always asks the
  developer exactly what should go in the release, verifies git/gh setup
  before running any git or gh commands, and never invents changelog
  entries: every entry must come from the developer or from commit history
  the developer has explicitly confirmed.
---

# GitHub Release

A disciplined, confirm-before-acting workflow for releasing a new version of a project on GitHub. The point of this skill is less "how do releases work" (routine) and more "never surprise the developer" — no invented changelog entries, no assumed version numbers, no git/gh commands fired at an unconfigured repo, no silently rewriting an existing release pipeline.

## Core principles (apply throughout, not just at the step where they're mentioned)

1. **Never fabricate.** Don't invent changelog entries, version numbers, release names, or scope. If you derived a candidate list from `git log`, label it clearly as a draft and get explicit confirmation before it goes in the changelog or release notes.
2. **Inspect before you build.** If release automation already exists in `.github/workflows/`, read it fully before deciding whether to use it, extend it, or (last resort) replace it.
3. **Verify the environment before any `git`/`gh` command that mutates state.** See Step 0 — this is not optional and comes before anything else.
4. **Ask, don't assume.** If anything below is ambiguous — which branch, what counts as "done" for this release, whether a change is breaking — ask the developer. Use short, specific questions, not open-ended ones, and prefer offering the choices you found (e.g. "I see commits for X, Y, Z since v1.3.0 — which of these should be in this release?") over vague prompts.
5. **Confirm before anything irreversible or remote-visible.** Pushing a commit, pushing a tag, and publishing a GitHub release are all checkpoints requiring explicit developer go-ahead — even if they approved the plan in principle earlier. Local commits/tags that haven't been pushed are cheap to undo; treat the push and the `gh release create`/`publish` as the real commit points.

---

## Step 0 — Verify the environment (always first, no exceptions)

Before running any other `git` or `gh` command, confirm:

```bash
git rev-parse --is-inside-work-tree      # is this a git repo at all?
git status --porcelain -b                 # branch, clean/dirty working tree
git remote -v                             # is there a GitHub remote configured?
gh auth status                            # is the gh CLI installed and logged in?
```

Handle failures explicitly, don't route around them:

- **Not a git repo** → stop, tell the developer, ask if they want you to `git init` (never do this silently).
- **No remote, or remote isn't GitHub** → stop and ask for the intended GitHub repo (or ask them to add the remote themselves).
- **`gh` not installed** → tell the developer and ask how they'd like to proceed (install it, or you fall back to git tag + a manually-crafted release they publish via the web UI).
- **`gh auth status` shows not logged in** → stop and ask the developer to run `gh auth login`, or confirm they want to proceed git-only (tag/push, but you cannot create the GitHub Release object itself without auth).
- **Dirty working tree** → surface what's uncommitted and ask whether it belongs in this release before doing anything else.

Do not proceed past this step on assumptions. If any check fails, that failure — and only that — is what you report back; don't guess at fixes.

---

## Step 1 — Inspect any existing release workflow

Check for existing automation before designing anything new:

```bash
ls -la .github/workflows/ 2>/dev/null
grep -rlE "release|tag" .github/workflows/ 2>/dev/null
```

For each workflow file that looks release-related, **read it in full** — don't skim the filename. For each one, determine:

- What triggers it (`push` to a tag pattern? manual `workflow_dispatch`? merge to `main`?)
- What it does (run tests, build artifacts, bump version, generate changelog, publish to a registry, create the GitHub Release, or some subset)
- What tool/convention it relies on, if any — see `references/workflow-patterns.md` for how to recognize semantic-release, release-please, goreleaser, or a hand-rolled `gh release create` step, and what each implies about who owns the version number and changelog.

Then decide, and say which you're doing and why:

- **Suitable as-is** → use it. Your job becomes: get the developer's confirmed content into whatever the workflow expects as input (a conventional-commit history, a tag, a `CHANGELOG.md` entry, etc.), then trigger it per its own trigger condition.
- **Close but needs a tweak** (e.g. it publishes but doesn't generate a changelog the developer wants, or it's pinned to an old action version) → propose the specific change to the developer before editing the workflow file. Don't silently rewrite someone's CI.
- **Nothing relevant exists** → proceed with the manual workflow in Steps 2–7 below, and optionally offer (don't assume) to set up a lightweight workflow for next time.

If you're unsure whether an existing workflow is "suitable," that uncertainty itself is a reason to ask the developer rather than to guess.

---

## Step 2 — Find out exactly what's going in this release

This is a hard requirement, not a nicety: **ask the developer what they want included** before drafting a version number or changelog. Don't infer scope purely from a branch diff and proceed.

Useful prep before asking (so your question is concrete, not open-ended):

```bash
git describe --tags --abbrev=0                       # last release tag, if any
git log <last_tag>..HEAD --oneline                    # commits since then
git diff <last_tag>..HEAD --stat                      # what changed, at a glance
```

Bring this to the developer as a starting point, e.g.:

> "Since v1.3.0 there are 9 commits touching auth, the export CLI, and a dependency bump. Which of these should be part of this release — all of them, or a subset? Anything not reflected in commit messages that should be called out?"

Also confirm, if not already obvious from context:
- Which branch is being released
- Whether this is a full release or a pre-release (`-beta.1`, `-rc.1`, etc.)
- Any breaking changes (these drive the version bump — see Step 3)

Do not move to changelog drafting until you have this.

---

## Step 3 — Determine the version number

Default convention is [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`), unless the project's existing tags show a different scheme (check `git tag --list`) or the developer says otherwise.

- Breaking change confirmed in Step 2 → MAJOR bump (or `0.x` minor bump pre-1.0 — ask, since pre-1.0 semver conventions vary by team)
- New backward-compatible functionality → MINOR
- Fixes only → PATCH

If it's genuinely ambiguous from what the developer told you (e.g. they said "some improvements" without detail), ask directly: "Does this include any breaking changes, or is it new features / fixes only?" Don't pick a bump level and move on silently — state the version you're proposing and get a nod before it goes into any file, tag, or commit message.

Find where the version lives in-repo so you update the right place(s), e.g.:

```bash
grep -rlE '"version"\s*:' package.json 2>/dev/null
grep -l "^version" pyproject.toml setup.cfg Cargo.toml 2>/dev/null
find . -maxdepth 2 -iname "VERSION" 2>/dev/null
```

If there are multiple version-bearing files (e.g. a workspace with several packages) and it's not obvious which should move, ask.

---

## Step 4 — Build the changelog (confirmed content only)

**Never write a changelog entry the developer hasn't confirmed.** You may draft candidate entries from the confirmed commit list in Step 2 to save them typing, but present them as a draft and get explicit sign-off (or edits) before writing to `CHANGELOG.md` or release notes.

Format: use [Keep a Changelog](https://keepachangelog.com/) style unless the repo already has an established different format (check the existing `CHANGELOG.md` first and match it). See `references/changelog-format.md` for the template and category conventions (Added / Changed / Deprecated / Removed / Fixed / Security).

Workflow:
1. Draft entries from the confirmed scope (Step 2), grouped by category.
2. Show the draft to the developer verbatim: "Here's the changelog section I'd add — anything to change, add, or cut?"
3. Only after confirmation, prepend the new version section to `CHANGELOG.md` (create the file with a standard header if it doesn't exist yet).

---

## Step 5 — Version bump commit

Update the version string(s) identified in Step 3 and the changelog file, then:

```bash
git add <changed files>
git commit -m "chore(release): vX.Y.Z"
```

Show the developer the diff before committing if there's any doubt about scope creep (e.g. unrelated files got staged).

---

## Step 6 — Tag and push (confirmation checkpoint)

```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z"
```

**Stop and confirm with the developer before pushing anything.** Pushing the commit and/or tag is what makes this visible/triggers CI — treat it as the real point of no return, distinct from having approved the plan earlier:

```bash
git push origin <branch>
git push origin vX.Y.Z
```

If Step 1 found a workflow that triggers on tag push, pushing the tag is likely enough to kick off the rest — say so, and move to Step 8 to monitor it rather than duplicating its work manually.

---

## Step 7 — Create the GitHub Release (if not handled by an existing workflow)

Only do this manually if Step 1 concluded there's no workflow that already creates the Release object.

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z" \
  --notes-file <path-to-this-release's-changelog-section> \
  [--prerelease]   # if this was flagged as a pre-release in Step 2
```

Use the exact, developer-confirmed changelog section from Step 4 as the notes — don't regenerate or embellish it here.

---

## Step 8 — Post-release verification

```bash
gh release view vX.Y.Z
gh run list --limit 5          # if a workflow was triggered, confirm it's green
```

Report back: release URL, whether CI passed, and (if the project publishes to a package registry) a reminder to verify the package installs cleanly. Don't mark the release "done" to the developer until any triggered CI has actually finished — offer to check back or to watch it (`gh run watch <run-id>`) rather than assuming success from the trigger alone.

---

## Reference files

- `references/workflow-patterns.md` — how to recognize and evaluate common existing release-automation setups (semantic-release, release-please, goreleaser, manual tag-triggered workflows) during Step 1.
- `references/changelog-format.md` — Keep a Changelog template and category conventions for Step 4.
