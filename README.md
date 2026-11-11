# Settings Backup for OpenChamber

An extension that writes the settings this OpenChamber instance keeps on disk into one portable file, and restores them here or on another install.

Everything happens on the instance the browser is connected to: two declared files are read, one file in your home folder is written, and nothing is sent anywhere.

![Bildschirmfoto 2026-11-11 um 01.29.17](imgs/Bildschirmfoto%202026-11-11%20um%2001.29.17.png)


## What it touches

| Path | Why |
| --- | --- |
| `~/.config/openchamber/settings.json` | Machine facts: projects, terminal, tunnels, tools, and the legacy copy of profile values |
| `~/.config/openchamber/preferences.json` | Profile keys as `{ value, updatedAt }`, including per-surface values |
| `~/openchamber-backup*.json` | Plain backups |
| `~/openchamber-backup*.ocbackup` | Backups sealed with a password |

The manifest declares exactly these four patterns. The host refuses every other path, and the install dialog shows them before you approve.

## Install

**Recommended (enables auto-update):** install from the Git URL so OpenChamber can
check for and apply updates:

```
https://github.com/Bitsonwheels/openchamber-settings-backup.git
```

In OpenChamber: **Settings → Extensions → add Git URL** → paste the URL above →
approve filesystem access.

**Alternative (no auto-update):** clone/download the folder and add the absolute
folder path under **Settings → Extensions → add folder path**, or install the
release ZIP from https://github.com/Bitsonwheels/openchamber-settings-backup/releases.

## Use

1. Install the folder: **Settings → Extensions → add an absolute folder path**, for example `<repo>/packages/sdk/examples/settings-backup`. Approve the filesystem access.
2. Open **Settings Backup** from the context rail (or as a full page).
3. **Export**: pick the keys, optionally set a password, write the file. The path is shown when it is done; the file sits in your home folder.
4. **Import**: type the file name (`~/openchamber-backup-2026-10-22T09-30-00Z.json`), read it, review what it carries, pick the keys, restore.
5. Reload OpenChamber (Cmd/Ctrl+R). The running page keeps the values it loaded before the restore.

Before every restore the extension writes `~/openchamber-backup-before-import-<timestamp>.json` with both current files. That file is the way back if a restore is not what you wanted.

## Updates

Extensions installed from a **Git URL** update themselves: OpenChamber checks
each of them at most once an hour (or immediately via **Check for updates** in
Settings → Extensions). A newer version shows **Update available** with an
**Update** button; permissions, accounts and settings are kept.

Folder and ZIP installs do not auto-update — replace them by hand, or add
`#v0.1.7` (or any tag/branch) to the Git URL to pin a version.

Manual fallback:
1. `cd /path/to/openchamber-settings-backup && git pull`
2. Reload OpenChamber (Cmd/Ctrl+R).

Or download the latest release ZIP:
https://github.com/Bitsonwheels/openchamber-settings-backup/releases/latest

## What is in a backup

- **Profile keys** (appearance, chat, shortcuts, notifications, models, voice, …): the app's `preferences.json` entries, including per-surface values such as the phone's own theme.
- **Machine facts**: projects, terminal shells, tunnel configuration, tool toggles, window state.
- **Secrets** (off by default): the desktop UI password, the managed tunnel token and its presets, and any key this build does not know whose name reads like a credential (`token`, `secret`, `password`, …) — the Electron host writes one such token outside the registry. They are plain text in `settings.json`; a backup with them is a credential file.
- **Unknown keys**: written by a newer OpenChamber than this extension knows. They are carried, marked `unknown`, and announced on import.

Not in a backup, because they live somewhere else:

- Provider API keys and OpenCode credentials (`auth.json`).
- MCP servers, agents, commands, skills, snippets, OpenCode plugins (`/api/config/*`).
- Custom themes, extensions, Git identities, passkeys, remote clients, GitHub/GitLab tokens.
- Device keys such as sidebar width or panel layout: they live in one browser, and the server drops them.

## Security

- The panel runs in a sandboxed iframe. It reaches those four paths through the host's file API and nothing else; it has no network access and cannot call the OpenChamber API.
- Password protection is PBKDF2-SHA256 (600 000 iterations) plus AES-GCM, through WebCrypto. WebCrypto needs a secure context: `https` or `localhost`. On a plain-http LAN address the password option is disabled and the panel says so.
- A forgotten password cannot be recovered. The salt and IV are random per file; the password is not stored anywhere.
- The backup format is plain JSON (or a JSON envelope when encrypted), so a damaged file fails loudly instead of restoring half of it.
- The guest file API writes with the process umask (usually `0644`), while OpenChamber's own writer uses `0600`. On a shared machine, run `chmod 600 ~/.config/openchamber/settings.json ~/.config/openchamber/preferences.json` after a restore. A backup that carries secrets is only as protected as the home folder it sits in; the password option is the portable answer.

## Limitations

- The paths above are fixed. An instance started with `OPENCHAMBER_DATA_DIR` keeps its files elsewhere; change the two declared patterns in `package.json` and rebuild for that setup.
- The file API stops at 2 000 000 characters per file. A backup of the two settings files is far below that.
- There is no download button: a sandboxed iframe may not start one. The file is written into your home folder instead.
- Extension panels run on web and Electron. VS Code and the mobile app do not render them.
- A restore merges. Keys the file does not carry keep their value on this instance; it never clears a document.
- Only version 1 of the backup format is read. A newer format is refused with a message instead of guessed at.

## Development

From the repository root:

```bash
node packages/sdk/examples/settings-backup/tools/generate-keys.mjs   # after settings-registry:generate
bun packages/sdk/examples/settings-backup/tools/build-panel.mjs      # panel/main.ts -> panel/main.js
node --test packages/sdk/examples/settings-backup/panel/backup.test.ts
```

`generate-keys.mjs` copies the key list and scopes from `packages/web/server/lib/opencode/settings-registry.json` into `panel/registryKeys.ts`. A key that is missing there still moves through a backup; it is shown as `unknown`.

### Publishing it on its own

The folder is an ordinary guest package. To leave this repository:

1. Copy `shared.ts` into the package and change the import in `panel/main.ts` from `../../shared.ts` to `./shared.ts`.
2. Replace `"@openchamber/sdk": "workspace:*"` with the published version.
3. Replace `tools/build-panel.mjs` with the SDK's `openchamber-guest-bundle` script, or keep the alias plugin.
4. Commit `panel/main.js` so the folder installs without a build step.
