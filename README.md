# aiClone

Run several separately signed-in copies of **Grok Bot**, Cursor, Claude, and ChatGPT (Codex) on one Mac.

Grok Bot (`/Applications/Grok Bot.app`, bundle id `com.anysphere.sand`) is the primary target. Each clone gets its own data folder, Dock wrapper, tinted icon, a private local-exec daemon (so it cannot steal the official app's computer connection), and a short-lived URL-scheme claim so browser sign-in returns to the clone that started it.

This is a Rust CLI rewrite of [Lots of Agents](https://github.com/carllippert/lots_of_agents) (Swift, MIT, Carl Lippert). Behavior is based on that project's source; this repo does not copy its code. Not affiliated with xAI, Anysphere, Anthropic, or OpenAI.

This repository holds **builds only** (in [`builds/`](builds), one folder per version), plus this README and the license. Source code is kept elsewhere.

The **GUI** (`aiclone-gui`) is a single window with eight Grok Bot clone buttons, Export, and Import. It is built with **egui / eframe** (the same immediate-mode toolkit [aiTrack](https://github.com/genen/aiTrack) uses) so the look — dark grouped chrome, compact rounded buttons, system-like sans — matches that app without pulling in a webview. The CLI (`aiclone`) is unchanged.

---

## Install

Download the build for your OS from [`builds/`](builds) (latest: [`builds/v0.1.1-beta`](builds/v0.1.1-beta)). Each folder has a `SHA256SUMS` file.

| OS | File | Install |
|---|---|---|
| Linux x86_64 (Debian / Ubuntu) | `aiclone_<version>_amd64.deb` | `sudo apt install ./aiclone_<version>_amd64.deb` |
| Linux x86_64 (any) | `aiclone-<version>-linux-x86_64.tar.gz` | `tar xzf aiclone-*.tar.gz` and copy `aiclone`, `aiclone-launcher`, `aiclone-gui` onto your `PATH` (e.g. `~/.local/bin`) |
| macOS | coming soon | |
| Windows | coming soon | |

Linux GUI builds need GTK 3 and X11 at runtime.

Confirm a supported app is installed (Mac):

```bash
ls -d \
  "/Applications/Grok Bot.app" \
  "/Applications/Cursor.app" \
  "/Applications/Claude.app" \
  "/Applications/ChatGPT.app" 2>/dev/null
aiclone apps
```

## GUI

```bash
aiclone-gui
```

The window has:

1. **Eight launch buttons** (slots 1–8). Colors match the Dock tint. Clicking creates `grokcloneN` if needed, then launches it. A green pip means that clone is running. Clicking a running clone does **not** claim URL schemes (use **Receive browser sign-in** on the right-click menu). Right-click a button to rename the label (Work, Wife, …), set a PIN / Touch ID / Windows Hello lock, launch-at-login (optional weekdays only), quit, reset, or claim `sand:` for a pending sign-in.
2. **Export** — All (one zip of every grokclone slot) or **1** (pick a slot). Optional AES-256-GCM in Settings; turning that switch on means each export asks for a password **and confirmation** (empty is rejected).
3. **Import** — All (restore an All zip) or **1** (a single-clone zip into any slot). Confirms before overwrite; refuses zip-slip (`..`, absolute paths, symlinks). Running clones must be quit first (the dialog can quit them). A snapshot is saved before overwrite; if the snapshot cannot be written the import is aborted. **Undo last import** asks for confirmation (and a decrypt password if the snapshot is encrypted).

On/off options are **switches** matching [aiTrack](https://github.com/genen/aiTrack) Settings (capsule track, system blue when on, gray when off, white sliding knob) — not checkboxes. Settings (the small ▸ at the bottom): encrypt exports, open aiClone at login, weekly All backup to a folder, undo last import. Turning **Weekly All backup** on prompts for a backup password (two fields); that password is stored in the OS keychain / a private credential file so scheduled backups run encrypted and unattended. Cancelling the prompt leaves the switch off.

A new PIN is at least 6 characters, hashed with PBKDF2-HMAC-SHA256 (600k iterations, salted); failed attempts back off. Locked slots must be unlocked before launch, PIN changes, export, import, or reset. Dock / CLI launches of a locked clone are refused (Touch ID / Hello can unlock the wrapper).

Clone identity in the catalog stays `grokclone1` … `grokclone8` so the CLI still finds them. Display names live in `gui.json` next to `profiles.json`.

Slot colors: `#FF453A` `#34C759` `#0A84FF` `#FF9F0A` `#BF5AF2` `#64D2FF` `#FFD60A` `#AC8E68`.

### What works on each OS

| | macOS | Windows | Linux |
|---|---|---|---|
| GUI window, Export / Import, PIN, snapshots, scheduled backup | yes | yes | yes |
| Launch Grok Bot clone (wrapper + `SAND_USER_DATA_DIR` + private `SAND_DATA_ROOT`) | yes (Dock wrapper, codesign, pending `sand:` claim, isolated local-exec daemon) | if Grok Bot is installed: spawn with isolated env (no Dock wrapper / unique bundle id / `sand:` handoff) | same as Windows |
| Running-clone detection | process argv + user-data-dir | pidfile of clones this app launched | `/proc` cmdline + `SAND_USER_DATA_DIR` |
| Touch ID / Windows Hello | Touch ID when available | Windows Hello when available | PIN only |
| Launch at login | LaunchAgent | Startup folder `.cmd` | XDG autostart `.desktop` |
| Official-app version prompt / wrapper refresh | yes | n/a (no wrapper) | n/a |

Encrypted exports are an **AES-256-GCM** container (`AIC1` magic + PBKDF2-HMAC-SHA256) around a normal zip — not ZipCrypto. Unencrypted zips stay ordinary `.zip` files.

## Usage

```bash
aiclone new grok Work --color '#FF453A'
aiclone new grok Personal
aiclone new cursor Client
aiclone new chatgpt Home          # isolation is forced to full-home-overlay
aiclone list
aiclone launch Work               # or: grok/Work, Personal, a slug, a UUID
AICLONE_DEBUG=1 aiclone launch Work   # print why a rebuild was skipped or started
aiclone quit Work
aiclone refresh                   # rebuild wrappers after an official app update
aiclone remove Work
```

`new` flags:

| Flag | Meaning |
|---|---|
| `--color '#5B8DEF'` | Dock tint. Cycles a palette if omitted. |
| `--badge WORK` | Optional short badge on the icon |
| `--isolation user-data-dir` \| `full-home-overlay` | ChatGPT always uses the overlay |
| `--shim` | Install `~/.local/bin/<app>-<slug>` |

Dock wrappers appear as e.g. `Grok Bot Work.app` in `~/Applications`. Sign in inside each window with the account you want for that clone.

## How it works

Each clone has a private data folder:

```text
~/Library/Application Support/AiClone/Profiles/<UUID>/   # macOS
~/.local/share/AiClone/Profiles/<UUID>/                  # Linux (XDG; old Library path still used if it already exists)
%AppData%\AiClone\Profiles\<UUID>\                       # Windows
  user-data/      passed as --user-data-dir
  extensions/     passed as --extensions-dir
  home/           full-home overlay (ChatGPT, or --isolation full-home-overlay)
```

Grok Bot ignores `--user-data-dir` (it resets userData before the single-instance lock). Clones set `SAND_USER_DATA_DIR` to the profile folder, which Grok Bot does honor, and set `SAND_DATA_ROOT` to a **private** `<profile>/sand-data` directory. The official app keeps using `~/.grokbot`. See [Local computer connection](#local-computer-connection).

Cursor also gets a short symlink `~/.aiclone/cursor-<slug>` because VS Code puts its IPC socket inside `--user-data-dir` and macOS caps socket paths at 103 characters.

For every built-in recipe (Grok Bot, Cursor, Claude, ChatGPT) aiClone builds a **cloned wrapper** in `~/Applications`:

1. APFS-clone (`cp -c -R`) the official `.app` — near-zero extra disk.
2. Rewrite `Info.plist`: unique bundle id (`works.aiclone.clone.<app>.<slug>`), `CFBundleDisplayName` set to the clone name (e.g. `Grok Bot Work`). **`CFBundleName` stays the official app's name** — Electron looks up `Frameworks/<CFBundleName> Helper.app`. Dock / Cmd-Tab already show the clone via the `.app` folder name. Localized `InfoPlist.strings` are left alone. Tinted `AppIcon.icns`. `CFBundleExecutable` becomes `aiclone-launcher`.
3. Ad-hoc re-sign the **whole clone bundle** (`codesign --force --deep --sign -`) after swapping the launcher, with team-bound entitlements stripped. Leftover `CodeResources` / `_CodeSignature` / stapled tickets from the official copy are deleted first so Info.plist is bound and resources are sealed. The launcher stamp stored in the plist hashes the Mach-O **without** its code-signature blob, so `--deep` signing the embedded copy does not make every launch look like an upgrade.
4. On open, the launcher rebuilds the copy **only if** the official app's version stamp or the launcher stamp changed. A no-op launch leaves `Info.plist` (and the rest of the wrapper) untouched. It restores any expired URL-scheme claims to the official app, then **execve**s the bundle's own binary with the clone's args plus `--use-mock-keychain`. It does **not** claim `sand:` on ordinary launch — use **Receive browser sign-in** in the GUI. `aiclone launch` waits ~3s and warns if the real clone process (not `aiclone-launcher`) is already gone. `aiclone refresh` skips a wrapper whose clone is running and prints `skipped (running, quit it first)`. Set `AICLONE_DEBUG=1` to print why a rebuild was skipped or started.

The official app in `/Applications` is never modified. The user's real default data folder is never used. `~/.grokbot` (the official local-exec daemon) is never used by a clone. No `sudo`.

## Browser sign-in (`sand:`, `cursor:`, `claude:`, `codex:`)

macOS Launch Services allows only one default handler per URL scheme. Older aiClone builds called `LSSetDefaultHandlerForURLScheme` on every launch and left that clone as the handler, so a browser callback started from the official Grok Bot app — or from a different clone — was delivered to whichever wrapper launched last.

A dedicated always-on router `.app` was considered and rejected: it would itself have to remain the default handler, so the official app would never be default.

Instead, aiClone keeps a claim book (`url-claims.json` next to `profiles.json`):

- The **official app** is the default handler whenever no clone sign-in is pending. The previous Launch Services handler is recorded when a claim starts (not assumed from the recipe's first bundle id).
- **Receive browser sign-in** (right-click a slot) claims that recipe's scheme for **that clone only** for 15 minutes. Ordinary launch, autostart, and clicking a running button do not claim. A later launch of a different clone does **not** steal the claim.
- When the claim expires, the clone is quit, or the slot is reset, Launch Services is restored **first**; the claim is dropped only after that apply succeeds. A detached `aiclone-gui --restore-claims` helper (`setsid`, not `fork`) watches until the book is empty. The wrapper also restores expired claims on every start.
- Choosing Receive browser sign-in while another clone still owns the claim shows a short message instead of switching the handler. A corrupt `url-claims.json` restores every official handler and is rewritten empty.

Linux and Windows do not register these schemes; the claim book still records the same policy so the tests cover it.

## Local computer connection

Grok Bot's "this Mac" / local-execution feature is **not** stored in Electron user-data. Public teardowns of 0.18 and 0.47 agree on the layout:

- The local-exec helper is the Grok Bot binary run with `ELECTRON_RUN_AS_NODE=1`, **detached** from the GUI. On an app update it keeps running via an `update-lease` file.
- Its state lives in `$SAND_DATA_ROOT`, which for the official app is `~/.grokbot`: daemon discovery, connection/credential files, supervisor heartbeat, `host.lock`, settings, logs. (Credential files are never read or printed by aiClone.)
- The hello frame that registers the computer uses a persisted machine id when one exists, otherwise `hostname()`, and a label that falls back to the same hostname. That is why a clone sharing `~/.grokbot` took the official entry offline and relabeled it (`My Mac` → `my-mac.local`).
- A separate machine-id blob also lives under Electron user-data (`~/Library/Application Support/Grok Bot` for the official app). Clones already isolate that via `SAND_USER_DATA_DIR`.

**Path taken:** each clone gets its own local-exec identity. Disabling local execution in clones was the fallback and was not needed.

On launch a Grok Bot clone sets:

| Env | Official app | Clone |
|---|---|---|
| `SAND_USER_DATA_DIR` | unset (default Application Support) | `<profile>/user-data` |
| `SAND_DATA_ROOT` | unset → `~/.grokbot` | `<profile>/sand-data` (always, even if the parent env pointed at `~/.grokbot`) |
| `SAND_COMPUTER_ID` | persisted by the app | `aiclone-<profile-uuid>` |
| `SAND_COMPUTER_LABEL` | user-chosen (e.g. `My Mac`) | `Grok Bot grokcloneN` |
| `ELECTRON_RUN_AS_NODE` | unset on the GUI; the daemon child sets `1` | stripped from the clone GUI; the clone's own daemon child still sets `1` |

Because daemon discovery, lease, socket-adjacent state, and computer id are per `SAND_DATA_ROOT`, two signed-in accounts can each keep a working local computer connection at the same time: the official app on `~/.grokbot`, each clone on its own `sand-data`. They do not share `host.lock` or the hello identity, so the official label must not change.

If a grok clone uses full-home overlay, `~/.grokbot` in that overlay is a private empty directory (not a symlink to the real one).

Catalog: `~/Library/Application Support/AiClone/profiles.json`.

ChatGPT keeps login state in `~/.codex` (outside Chromium user-data), so clones get a full-home overlay: `.cursor` / `.codex` (and ChatGPT's Application Support folders) are private empty dirs; everything else in `$HOME` is a symlink back to the real home.

## Caveats

- **Unofficial.** Vendor updates can break `--user-data-dir`, URL schemes, or helper lookup. Run `aiclone refresh` after an app update; if a clone still misbehaves, delete it and create a new one.
- **Gatekeeper.** Wrappers are ad-hoc signed. First launch: right-click → Open, or System Settings → Privacy & Security → Open Anyway. `xattr -dr com.apple.quarantine` on the wrapper if macOS quarantined a copy.
- **Not a security boundary.** Clones isolate *accounts and chats*, not malware. Full-home overlay still sees your real files via symlinks.
- **Sparkle is stripped from the clone only** (Electron Framework does not link it). **Squirrel.framework stays** — Electron Framework loads `@rpath/Squirrel.framework/Squirrel` (also Mantle and ReactiveObjC). Updates are disabled via the `SU*` plist keys and `--use-mock-keychain`. Update the official app; clones pick it up on next launch / `refresh`.
- **Grok: several local agents at once (tentative).** With the official app and clones each on their own data folder (`SAND_DATA_ROOT` / `SAND_USER_DATA_DIR`; the official default is `~/.grokbot`, `%USERPROFILE%\.grokbot` on Windows), all local computer connections stayed up at the same time on a Mac and a Windows PC. Earlier builds saw newest-wins, where the last agent to connect pushed the others offline. Logs then showed server-side connection resets on the clone's daemon, which cleared after restarting the official app's daemon. If it happens again, copy both `local-exec-daemon.log` files before restarting anything. Still open: clones log `device tool status read failed`.
- **Mac-only features.** Bundle copy, `codesign`, `iconutil`, URL-scheme handling, Touch ID, and launching Grok Bot clones work only on a Mac.

## Uninstall

```bash
rm -f "$HOME/.local/bin/aiclone" "$HOME/.local/bin/aiclone-launcher" "$HOME/.local/bin/aiclone-gui"
# or, if installed from the .deb: sudo apt remove aiclone
# optional: clones + wrappers you created
# rm -rf "$HOME/Library/Application Support/AiClone"
# rm -rf "$HOME/.aiclone"
# rm -rf "$HOME/Applications/Grok Bot "*.app   # only wrappers you no longer want
```

Removing aiClone does **not** uninstall Grok Bot / Cursor / Claude / ChatGPT.

## Credit

The clone model — per-profile `--user-data-dir`, APFS-cloned wrappers, team-bound entitlement stripping, URL-scheme claiming, Cursor's short socket path, and ChatGPT's full-home overlay — comes from [Lots of Agents](https://github.com/carllippert/lots_of_agents) by Carl Lippert, MIT License. This project is an independent Rust implementation.

## License

[MIT](LICENSE)
