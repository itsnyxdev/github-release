# github-release

Disciplined, confirm-before-acting workflow for cutting GitHub releases. Never invents changelog entries, never assumes version numbers, never fires `git`/`gh` at unconfigured repo.

## Install

```bash
npx skills add itsnyxdev/github-release
```

## Use

Trigger phrases: "release new version", "cut release", "publish vX.Y.Z", "ship to GitHub", "tag release", "update changelog and release", "bump version".

## How it works

9-step workflow in `SKILL.md`: verify env → inspect workflow → confirm scope → semver → changelog (confirmed only) → commit → tag+push (checkpoint) → `gh release create` → verify.

Core rules: never fabricate, inspect before build, verify env first, ask don't assume, confirm before push/tag/release.

## Structure

- `SKILL.md` — workflow entry point
- `references/workflow-patterns.md` — recognize semantic-release, release-please, goreleaser
- `references/changelog-format.md` — Keep a Changelog template

## License

MIT — see `LICENSE`.
