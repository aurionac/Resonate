# Resonate

A local Spotify-style music player: build playlists, play with shuffle/loop, browse YouTube, and download MP3s — reusing files you’ve already grabbed.

Runs as a **web app** (dev browser) or a **desktop app** (Electron) for Windows, macOS, and Linux.

## Requirements

### Development

- Node.js 18+
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) and [ffmpeg](https://ffmpeg.org/) on your `PATH` (needed for YouTube search/download)

```bash
brew install yt-dlp ffmpeg    # macOS
# Windows: winget install yt-dlp.yt-dlp Gyan.FFmpeg
# Linux: use your package manager
```

### Packaged desktop app users

- No Node.js install required
- Still need `yt-dlp` and `ffmpeg` available on the system `PATH` (or set `YT_DLP_PATH` to a yt-dlp binary)

## Setup

```bash
cd ~/Coding/musicthingy
npm install
npm install --prefix client
```

## Development

### Browser (Vite + Express)

```bash
npm run dev
```

Open http://localhost:5173

### Desktop window (Electron + Vite HMR)

```bash
npm run electron:dev
```

This builds the Electron main process, starts Vite, starts the Express API inside Electron (user-data paths), and opens a native window pointed at the Vite dev server.

## Production desktop builds

Build installers/packages for your current OS:

```bash
npm run make
```

Outputs land in:

```text
out/make/
```

Typical artifacts:

| Platform | Command | Output (under `out/make/`) |
|----------|---------|----------------------------|
| macOS | `npm run make` or `npm run make:mac` | `zip/darwin/…`, `dmg/…` |
| Windows | `npm run make` or `npm run make:win` | `squirrel.windows/…`, `zip/win32/…` |
| Linux | `npm run make` or `npm run make:linux` | `deb/…`, `rpm/…`, `zip/linux/…` |

Package without installers (unpacked app only):

```bash
npm run package
```

Unpacked app:

```text
out/Resonate-<platform>-<arch>/
```

Cross-compiling installers (e.g. Windows from macOS) often needs extra tooling; prefer building on each target OS — or use the GitHub Actions workflow below.

### GitHub Actions (Windows + Linux)

A workflow at [`.github/workflows/build-desktop.yml`](.github/workflows/build-desktop.yml) builds:

| Job | Runner | Artifacts |
|-----|--------|-----------|
| **Windows x64** | `windows-latest` | Squirrel installer (`.exe`) + ZIP |
| **Linux x64** | `ubuntu-latest` | `.deb`, `.rpm`, ZIP |

macOS DMGs are still built locally on your Mac (`npm run make`).

#### Trigger the workflow

1. Push this repo to GitHub (if it isn’t already).
2. Open the repo on GitHub → **Actions** → **Build Desktop**.
3. Click **Run workflow** (manual `workflow_dispatch`), pick the branch, then **Run workflow**.

It also runs automatically when you push a version tag:

```bash
git tag v1.0.0
git push origin v1.0.0
```

#### Download the installer

1. Open the completed workflow run.
2. Scroll to **Artifacts** (requires a successful Windows job with upload).
3. Download **`resonate-windows-x64`**.
4. Unzip it. You should see Squirrel.Windows outputs like:

```text
squirrel.windows/x64/Resonate-1.0.0 Setup.exe   # main installer
squirrel.windows/x64/*-full.nupkg               # update package
squirrel.windows/x64/RELEASES                   # update metadata
zip/win32/x64/*.zip                             # optional portable zip
```

Run the `* Setup.exe` to install. Artifacts are kept for **30 days**.

If a run has **no Artifacts section**, the upload step did not publish (often a failed `make`, or missing `actions: write` permission on the workflow). Re-run after pulling the latest workflow file.

#### What the CI build includes

Same as a local `npm run make` on that OS:

- Electron shell + compiled Express backend (`dist/server`)
- React UI (`client/dist`)
- Production Node dependencies only — **users do not need Node.js**
- User library/data still lives in the OS app data folder (`%APPDATA%\Resonate` on Windows, `~/.config/Resonate` on Linux)

### What gets packaged

- Electron shell
- Compiled server (`dist/server`)
- React UI (`client/dist`)
- Production Node dependencies (Express, etc.) — end users do **not** need Node.js

## Data locations

| Mode | Where files live |
|------|------------------|
| `npm run dev` / `npm run electron:dev` | Project folders: `data/`, `library/`, `thumbnails/` |
| Packaged desktop app | OS user-data directory (see below) |

Packaged app user-data:

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/Resonate/` |
| Windows | `%APPDATA%\Resonate\` |
| Linux | `~/.config/Resonate/` |

Inside that folder: `data/store.json`, `library/` (MP3s), `thumbnails/`.

On first packaged launch, if user-data is empty, Resonate copies an existing project library from `~/Coding/musicthingy` (or `RESONATE_LEGACY_ROOT`) when present.

## Other scripts

| Script | Purpose |
|--------|---------|
| `npm run build:app` | Build React UI + compile Electron/server TypeScript |
| `npm run start` | Electron Forge start (production-like, serves built UI) |
| `npm run start:server` | Express only (serves `client/dist` after build) |

## Features

- Library + playlist management
- Delete songs from disk (trash icon) — removes the MP3, thumbnail, and playlist entries
- Background / concurrent YouTube downloads
- Player: play/pause, next/prev, seek, volume, **shuffle**, **loop all / loop one**
- YouTube search browse
- Paste a YouTube link → pick a playlist → download as MP3
- Duplicate detection by YouTube video ID (skips re-download, reuses the existing file)
