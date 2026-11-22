# Privacy

aiClone is a small program that runs on your own computer. It has no account, no server of its own, no analytics and
no crash reporting. **It does not send any information about you or your computer to the author or to anyone else.**
aiClone itself makes no network requests.

## What it reads and changes on your computer

- The apps it clones (Grok Bot, Cursor, Claude, ChatGPT). On a Mac it copies the app into `~/Applications` to make
  each clone's wrapper; the official app in `/Applications` is never changed.
- Each clone's own data folder, which aiClone creates (`~/Library/Application Support/AiClone` on a Mac,
  `~/.local/share/AiClone` on Linux, `%AppData%\AiClone` on Windows).
- On a Mac, the URL-scheme handler for an app while you use **Receive browser sign-in**. aiClone puts the official app
  back as the handler when that ends.

aiClone does not read the sign-in or credential files of the apps it clones. The official Grok Bot app's
`~/.grokbot` folder is never used by a clone.

## What it stores on your computer

Its catalog of clones (`profiles.json`), window settings (`gui.json`), the sign-in claim list (`url-claims.json`),
lock files and import snapshots, all inside the AiClone folder above. A slot PIN is stored only as a salted hash.
If you turn on **Weekly All backup**, its password is kept in the macOS Keychain, Windows Credential Manager, or (on Linux) a
private file in the AiClone folder.

Exports and backups are written only to the place you pick. They hold the clones' data, including their sign-ins, so
keep them safe; turning on encryption in Settings protects them with a password.

## What the clones send

Each clone is the vendor's own app. When you use it, it talks to that vendor (xAI, Anysphere, Anthropic or OpenAI)
exactly as the official app does, under that vendor's privacy policy.

## Updates and contact

aiClone does not check for updates by itself. New versions are posted at
https://github.com/ai4coo/aiClone/releases. Questions: https://github.com/ai4coo/aiClone/issues.
