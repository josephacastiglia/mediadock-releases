# MediaDock — Developer Handoff

Updated: 2026-10-22

## Project state

MediaDock is a Windows x64 Electron desktop app for downloading and organizing accessible Instagram, YouTube, and TikTok media. The current production version is **1.3.0**. The app, queue, library, and account windows are one installed product. Users receive updates from **Tools & updates**; they do not manually download or unpack update ZIPs.

The public repository, [josephacastiglia/mediadock-releases](https://github.com/josephacastiglia/mediadock-releases), is a release feed. It contains the public app archive and signed update manifest, not the app's source tree. The latest published build is [MediaDock 1.3.0](https://github.com/josephacastiglia/mediadock-releases/releases/tag/v1.3.0).

## What the app includes

- YouTube, TikTok, and Instagram download workflows, with a TikTok creator search and selectable posts.
- A persistent queue with reorder, cancellation, retry, removal, and per-job reports.
- Batch URL and text-file import, duplicate checking, and creator collections.
- A searchable media library with playback, captions, descriptions, and photo albums.
- Folder and filename templates, disk-space reserve, connection retry, and background tray operation.
- Library repair, privacy-filtered backup and restore, and queued video clips that preserve the original.
- yt-dlp, FFmpeg, FFprobe, and Deno packaged with the Windows app. The app does not require Python, Node, or `yt-dlp` on PATH to run.

See the user's [MediaDock guide](https://github.com/josephacastiglia/mediadock-releases/blob/main/README.md) for operating instructions and platform limitations.

## Source layout

In the private development source tree, `src/main.mjs` coordinates Electron windows, IPC, queue execution, and background services. `src/video.mjs`, `src/instagram.mjs`, and `src/library.mjs` handle media providers and library records. `src/preferences.mjs`, `src/reliability.mjs`, `src/backup.mjs`, `src/repair.mjs`, `src/clips.mjs`, and `src/background.mjs` contain the added feature logic. `src/updates.mjs` and `src/online-updates.mjs` validate and install signed online releases. The renderer lives under `ui/`; release and packaging scripts live under `scripts/`; automated tests live under `tests/` and `vendor/instagram/tests/`.

The public release repository does not contain these sources. Locate the existing private local development workspace before editing or building; do not treat this release repository as the source checkout.

## Build and release

The documented build environment is Windows x64, Node.js 22.12 or later, and the packaged tool binaries under `tools/`. From the source directory:

```powershell
npm ci
node node_modules/electron/install.js
node scripts/setup-tools.mjs
node --test tests/*.test.mjs vendor/instagram/tests/*.test.mjs
npm start
```

To prepare and publish a new build, increment `version` in `package.json` and `package-lock.json` together, run the test command above, then run:

```powershell
npm run release
```

`src/update-channel.json` identifies the public release repository. Publishing uses GitHub CLI (`gh`) with permission to create releases in `josephacastiglia/mediadock-releases`. The release script packages the app, signs the manifest, and uploads the versioned ZIP plus `latest.json`. Keep updates on the same channel so existing installations can discover them. `npm run package` creates a local signed build without publishing the GitHub release.

The public key at `src/update-public.pem` is part of the app. The matching Ed25519 private signing key is held separately in the trusted development environment and is intentionally absent from the source package and public repository. Do not commit, upload, paste into logs, or regenerate that key: existing installations trust the embedded public key. The app update signature is not a Windows Authenticode certificate.

## Verification record for 1.3.0

- 308 automated tests passed before release.
- Packaged Windows smoke checks covered the queue, library and captions, batch input, settings, backup privacy, restore, repair, clips, background operation, retry handling, and paused rate-limit/disk-space states.
- A real public TikTok download was checked with FFprobe for video and audio streams; yt-dlp caption and description sidecars were also verified.
- The hosted release was fetched through MediaDock's unauthenticated updater. Its Ed25519 manifest, archive SHA-256, and all 85 staged file hashes matched. The updater then reported 1.3.0 as current.

## Data and privacy boundaries

The app's user state and media downloads are outside the program installation. The public release feed contains app files and update metadata only. User cookies, sign-in exports, browser sessions, download history, and media files must never be added to a release. Backup and restore intentionally omit sign-in credentials and provider request data.

For user-facing behavior, preserve the single MediaDock installation and in-app signed update flow. Existing users should not need to handle update ZIPs manually.
