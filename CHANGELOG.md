# Changelog

All notable changes to the Settings Backup extension are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [0.1.11] - 2026-11-06

### Fixed
- TDZ `ReferenceError` on first mount with non-English locale: locale-change
  handling now runs after the `didMount` guard and uses the module-level
  `refreshLabelsCallback` instead of TDZ-bound `rowViews`/`refreshLabels`.

## [0.1.10] - 2026-11-06

### Fixed
- Standalone manifest missing `~/Downloads` permission (added 2 patterns).
- Re-render on late dictionary load / locale change.

## [0.1.9] - 2026-11-06

### Added
- Extension version (`v0.1.9`) is now shown in the panel footer, plus a hint
  that updates are managed via Settings → Extensions.

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
