# Code signing policy

## macOS

The Mac programs are ad-hoc signed only. They are **not signed with an Apple Developer ID and not notarized yet**, so
macOS blocks them when they are downloaded with a browser. The [README](README.md#mac) shows the `xattr` command that
removes that block. This file will be updated when Developer ID signing starts.

The clone wrappers that aiClone makes in `~/Applications` are always ad-hoc signed on your own Mac, because each one is
a changed copy of the official app. The first time you open one, macOS may ask you to confirm (right-click → Open).

## Windows

Windows releases are not code-signed yet, so Windows SmartScreen may show "Unknown publisher" (choose "Run anyway").
This file will be updated when signing starts.

## Linux

The `.deb` and `.tar.gz` files are not signed. Check them against `SHA256SUMS` (below).

## How a release is made

- Builds are made by the project owner from the project's private source and uploaded by hand to the
  [Releases page](https://github.com/ai4coo/aiClone/releases). No hosted build service is used.
- Every release is approved by the project owner before it is published.
- Each release has a `SHA256SUMS` file. To check a download, put it in the same folder as `SHA256SUMS` and run
  `sha256sum -c SHA256SUMS --ignore-missing` (Linux), `shasum -a 256 -c SHA256SUMS --ignore-missing` (Mac), or
  `Get-FileHash <file>` in PowerShell (Windows) and compare the result with the line for that file.

## Roles

The project owner is the only author, reviewer and approver.

## Privacy

See [PRIVACY.md](PRIVACY.md): aiClone does not send any information to other networked systems.

## Reporting a problem

If you think a release file is wrong or malicious, open an issue at https://github.com/ai4coo/aiClone/issues.
