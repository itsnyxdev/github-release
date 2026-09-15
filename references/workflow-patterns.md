# Recognizing existing release workflow patterns

When inspecting `.github/workflows/*.yml` in Step 1, match what you find against these common patterns to figure out what it already owns (so you don't duplicate or fight it) and what it still expects from you.

## 1. semantic-release

**Signs:** a step running `npx semantic-release` or `semantic-release` in the job; a `.releaserc`, `.releaserc.json`, or `release` key in `package.json`; devDependency on `semantic-release` and plugins like `@semantic-release/changelog`, `@semantic-release/git`.

**What it owns:** version number (derived automatically from Conventional Commits since the last release), changelog generation, git tag creation, GitHub Release creation, and often package publish — all in one run, usually triggered on push/merge to the release branch.

**What this means for you:** you generally should NOT hand-bump the version or hand-tag. Your job shifts to making sure the *commit messages* going into the release branch are Conventional-Commits-correct (`feat:`, `fix:`, `feat!:`/`BREAKING CHANGE:` footer) and reflect what the developer confirmed in Step 2. Ask the developer whether they want you to just get the commits in and let the workflow run, or whether they want a preview of what it would generate.

## 2. release-please

**Signs:** a step using `googleapis/release-please-action`; a `release-please-config.json` or `.release-please-manifest.json` in the repo root.

**What it owns:** release-please opens/maintains a standing "release PR" that accumulates a changelog from Conventional Commits and bumps the version file(s) it's configured to track. Merging that PR is what triggers tag + GitHub Release creation.

**What this means for you:** check if a release PR already exists (`gh pr list --search "release-please"`) — if so, your job is often to review/edit that PR's changelog content with the developer (per Step 4's confirmation rule) rather than writing a changelog from scratch, then get their go-ahead to merge it. Don't hand-create a tag; merging the PR does that.

## 3. goreleaser

**Signs:** a step running `goreleaser/goreleaser-action`; a `.goreleaser.yml`/`.goreleaser.yaml` in the repo root; typically Go projects.

**What it owns:** building cross-platform binaries/archives, generating a changelog from commits (configurable, often just a commit list — check the `changelog` section of `.goreleaser.yml` for filters), and creating the GitHub Release with those artifacts attached. Usually triggered on tag push.

**What this means for you:** your job is normally still to confirm scope (Step 2), pick/confirm the version (Step 3), and push the tag (Step 6) — goreleaser takes it from there. Read the `changelog` section of the config before assuming it'll produce developer-approved wording; if it's just dumping raw commit messages, flag that to the developer since it bypasses your Step 4 confirmation step.

## 4. Hand-rolled `gh release create` / `actions/create-release`

**Signs:** a workflow step directly invoking `gh release create` or the older `actions/create-release`, usually triggered on `push: tags: - 'v*'` or `workflow_dispatch`.

**What it owns:** just the Release object creation (and possibly artifact upload). Version number and changelog are usually still your responsibility.

**What this means for you:** check what it uses for `--notes`/`body` — if it's `git log` output or a `CHANGELOG.md` slice, make sure the confirmed Step 4 changelog ends up wherever that step reads from before you push the tag.

## 5. No workflow / CI does tests only

**Signs:** workflows exist but only run on `pull_request`/`push` to run tests, lint, build — nothing release-shaped.

**What this means for you:** proceed with the full manual workflow (Steps 2–7 in SKILL.md). Consider asking the developer, after a successful manual release, whether they'd like a lightweight workflow set up for next time — but only build one if they say yes; don't add automation as a surprise side effect of a release request.

## General inspection checklist

Whichever pattern you find, before deciding it's "suitable as-is," check:

- [ ] What triggers it — does it match how the developer wants to release (push to tag vs. merge to main vs. manual dispatch)?
- [ ] Does it publish anywhere (npm, PyPI, crates.io, Docker Hub, etc.)? If so, confirm the developer actually wants a publish this time, not just a GitHub Release.
- [ ] Is it currently passing? (`gh run list --workflow=<file>`) A broken release workflow is not "suitable as-is" even if the logic looks right on paper.
- [ ] Does its changelog/notes generation respect the "developer-confirmed only" rule, or does it auto-generate from raw commits with no review step? If the latter, surface this explicitly — some teams are fine with it, some aren't.
