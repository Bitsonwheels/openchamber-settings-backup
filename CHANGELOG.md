# Changelog

All notable changes to the Settings Backup extension are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [0.1.8] - 2026-11-06

### Changed
- README: documented Git-URL install (auto-update) and manual update paths.
- Added this CHANGELOG.

## [0.1.7] - 2026-11-06

### Added
- Key labels in the "Which keys" section now show the user-facing labels from
  the OpenChamber settings UI (localized via the host locale) instead of raw
  variable names; the filter now matches both key and label.
- `Updates` button removed again: OpenChamber core handles extension updates
  for Git-URL installs, so the in-panel button was redundant.

### Fixed
- Label resolver no longer picks `*Aria`/`*Hint`/`*Description` keys (e.g.
  `sttLanguage`, `fontSize`, `followUpBehavior` showed the wrong text).
- Removed `as any` / `Promise<any>` that violated `no-explicit-any` and the
  anti-slop ratchet.
- Moved label resolution into the pure `backup.ts` module with a test.

## [0.1.6] - 2026-11-06

### Changed
- Aligned `package.json` version strings between the repo-root copy and the
  SDK example (was `0.1.0` vs `0.1.5`).

## [0.1.5] - 2026-10-24

### Added
- First published release with ZIP asset.
