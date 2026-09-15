# Changelog format reference

Default to [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) style, **unless the repo's existing `CHANGELOG.md` already uses a different established format** — in that case, match the existing format instead of imposing this one.

## File-level structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.4.0] - 2026-09-15

### Added
- Short, user-facing description of the new capability.

### Changed
- Short description of a change in existing behavior.

### Fixed
- Short description of a bug fix.

## [1.3.0] - 2026-08-01
...
```

New releases are **prepended** — most recent version directly under `## [Unreleased]`, older versions below.

## Category conventions

Only include categories that actually have entries for this release — don't emit empty headers.

| Category   | Use for |
|------------|---------|
| `Added`      | New features or capabilities |
| `Changed`    | Changes to existing functionality |
| `Deprecated` | Features that still work but will be removed |
| `Removed`    | Features that were removed |
| `Fixed`      | Bug fixes |
| `Security`   | Vulnerability fixes — call these out clearly, developers scan for these |

## Entry style

- One line per entry, imperative or past tense (pick whichever the existing file already uses; default to past tense — "Added support for X" — if starting fresh).
- User/consumer-facing framing, not raw commit messages. "Fixed a crash when exporting empty projects" beats "fix: null check in export.ts".
- Link issue/PR numbers if the repo convention does so: `- Added dark mode support ([#142](link))`.
- **Every entry must trace back to something the developer confirmed in Step 2/Step 4 of the skill workflow — do not add an entry because it "seems likely" from a diff.** If a commit's intent is unclear from its message alone, ask rather than paraphrasing a guess.

## Version link footer (optional but common)

If the repo uses comparison links at the bottom:

```markdown
[Unreleased]: https://github.com/OWNER/REPO/compare/v1.4.0...HEAD
[1.4.0]: https://github.com/OWNER/REPO/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/OWNER/REPO/compare/v1.2.0...v1.3.0
```

Only add this if the existing file already does it, or the developer asks for it — it's a nice-to-have, not a requirement.
