<div align="center">

<a href="https://github.com/apennismightier/megatronn">
  <img src="https://i.ibb.co/zhn5pDvf/Mega-Tronn.png" alt="MegaTronn" width="400">
</a>

# MegaTronn

**Powered by DeceptiCloud**

[![Version](https://img.shields.io/github/v/release/apennismightier/megatronn?label=latest&color=22c55e)](https://github.com/apennismightier/megatronn/releases)
[![Platform: Windows](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4?logo=windows)](#installation)
[![Built with Electron](https://img.shields.io/badge/Electron-28-47848F?logo=electron)](https://www.electronjs.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript)](https://www.typescriptlang.org/)

</div>

---

## What is it?

MegaTronn is an unofficial desktop client for [Mega.nz](https://mega.nz/) that turns the painful workflow of "open browser, paste link, log in, click download" into a one-window experience. Manage hundreds of Mega accounts, parse links out of any messy paste, download in parallel at full bandwidth, and upload through an embedded browser that actually behaves like a real one.

Built because the official MEGAsync client doesn't handle multi-account workflows, link-list ingestion, or batch reactivation — and because the browser webclient stalls out the moment you try to do anything at scale.

> **Disclaimer:** MegaTronn is independent third-party software. It is not affiliated with, endorsed by, or connected to Mega Limited. You are responsible for ensuring your use complies with Mega's Terms of Service.

---

## Features

### Multi-Account Management
- Unlimited accounts organized into named groups (`Personal`, `Archive-A`, etc.)
- **Bulk import** from YAML lists, plain 4-line blocks, or `email:password` colon format — pasted or from file, with live preview
- Collapsible groups, alphabetical sort, multi-select with bulk move/delete
- Resizable account sidebar (180–520px), state persisted across restarts
- Custom group autocomplete that scales to hundreds of groups (vanilla `<datalist>` broke at 100+)

### Link Parser
- Extracts Mega URLs from arbitrary text — single and double Base64 decoded
- Container file support: drag-and-drop or "Open with..." for **`.dlc`**, **`.ccf`**, **`.rsdf`**, and **`.txt`**
- **Clipboard auto-capture** (800ms polling) routes copied Mega links into the parser queue — no surprise downloads, you click when you're ready
- **📂 Browse folder** any `/folder/` link inline: see every file inside, untick what you don't want, download only what you pick

### Parallel Downloader
- **12 concurrent connections per file**, **16 MB chunks** — auto-tuned to Mega's per-IP throughput sweet spot
- Up to 6 simultaneous transfers by default; bump higher if your bandwidth supports it
- 8 MB disk write buffer so 16 MB chunks don't stall on backpressure
- **Transfer queue persists** across app restarts and version upgrades — in-flight downloads resume from scratch, completed history is preserved
- Anonymous fallback when an account session expires (`ESID` errors) — public links don't need a logged-in account
- Per-transfer pause/resume, bulk pause-all / resume-all / cancel-all
- ▶ **Open** file with default app, 📁 **Show in Explorer/Finder** on completed rows

### Embedded Cloud Drive
- mega.nz rendered inline as a fully-functional browser tab — auto-login with stored credentials, persistent session per account
- **Auto-transfer worker hook** eliminates the "Data cannot be cloned, out of memory" crash that kills uploads of large files in stock embedded browsers (Mega's upload code clones every chunk via `postMessage` — we transfer ownership instead, zero allocation)
- 16 GB V8 heap headroom for large file workloads
- 🗑 **Clear Cache** button for one-click session reset when Mega's IndexedDB cache fills up
- File-path privacy: your local upload paths are hidden from mega.nz's page (uploads show only filenames, never `C:\Users\you\Documents\...`)
- Auto-recovery: if an upload fails with memory pressure anyway, the webview silently clears storage and reloads — you keep working

### Account Reactivator
- Headless browser-based reactivation for dormant accounts
- Batch queue with configurable delays
- CAPTCHA escalation to the user when automated solving fails

### Smart Proxy
- HTTP, SOCKS4, SOCKS5 protocols
- Per-chunk proxy rotation (so a slow/dead proxy doesn't tank a whole transfer)
- Configurable ban duration when a proxy returns errors

### Quality of Life
- **GitHub Releases auto-update checker** — top-banner notification when a newer version is published; click to view release notes, install at your pace
- Embedded log viewer with live tail, level filter (`INFO` / `WARN` / `ERROR`), and one-click Desktop export
- My Links library — per-account saved links with Base64 encoder
- File associations: double-click any `.dlc` / `.ccf` / `.rsdf` and MegaTronn opens it
- System tray + minimize-to-tray
- Single-instance enforcement: opening a second container file routes through the running window instead of spawning duplicates
- Pinned taskbar icon survives version upgrades (locked AppUserModelID + fixed install path)

---

## Installation

### Pre-built (Windows)

1. Download the latest `MegaTronn-Setup-<version>.exe` from the [Releases page](https://github.com/apennismightier/megatronn/releases/latest).
2. Run it. The installer is per-user (no admin rights required) and installs to `%LocalAppData%\Programs\MegaTronn`.
3. Pin to taskbar — your pin will survive future updates.

### From source

```bash
git clone https://github.com/apennismightier/megatronn.git
cd megatronn
npm install
npm run dev          # development with HMR
# or
npm run build:win    # produces dist/MegaTronn-Setup-<version>.exe
```

Requirements: Node.js 18+, npm, and on Windows: Visual Studio Build Tools (for any native deps).

---

## Tech Stack

| Layer        | Stack                                                         |
|--------------|---------------------------------------------------------------|
| Runtime      | Electron 28 (Chromium 120, Node 18)                           |
| UI           | React 18 + TypeScript 5 + Tailwind CSS                        |
| Build        | electron-vite (renderer + main), electron-builder NSIS (installer) |
| State        | Zustand                                                       |
| Persistence  | electron-store (settings + accounts), JSON (transfer queue)   |
| Logging      | electron-log → `%AppData%\megatronn\logs\main.log`            |
| Mega API     | [`megajs`](https://github.com/qgustavor/mega) (TypeScript)    |

---

## Configuration

Everything is in-app under **Settings** (gear icon). Defaults are tuned for Mega's optimal throughput — most users never need to touch them.

- **Downloads**: max concurrent transfers, default destination, incomplete-file directory
- **Performance**: per-file connections, chunk size — auto-floored to optimal values if set too low
- **Paths**: where downloaded files land, where the log lives
- **Proxy**: protocol, server list, rotation rules
- **Reactivator**: headless on/off, delay range between attempts

Settings, accounts, and the transfer queue live in `%AppData%\megatronn\` (or your platform equivalent). The app **does not** make outbound network requests other than to mega.nz and to GitHub (once per launch, for the update check).

---

## Privacy & Safety

- **No telemetry.** No analytics SDK, no error reporting service, no "usage data" of any kind.
- **No ads.** Nothing is monetized.
- **Local-only state.** Accounts, settings, transfer queue, logs — all on your disk, never transmitted.
- **One outbound check** per app launch to `api.github.com` for the version check (anonymous, no identifier sent beyond a `User-Agent: MegaTronn/<version>` header).
- The embedded browser sandboxes each account in its own session partition — accounts cannot see each other's cookies, storage, or upload state.

---

## Roadmap

- [ ] macOS + Linux builds
- [ ] Bandwidth limiter (throttle download speed)
- [ ] Scheduled / time-windowed transfers
- [ ] Auto-update with in-app install (currently link-out to GitHub)
- [ ] Webhook notifications on completion
- [ ] Folder-level upload queueing from local disk

Open an issue if there's something you'd want.

---

## Contributing

Issues and PRs welcome. Please attach a diagnostic log (`Settings → Export Diagnostic`) when reporting bugs — it bundles the main log, transfer queue state, settings, and system info in one text file.

If you're submitting a PR, please:
- Run `npm run lint` before pushing
- Bump the version in `package.json`, `src/main/index.ts`, and `src/renderer/src/components/About.tsx` together (the three are kept in sync)
- Add a brief note to the changelog

---

## License

[MIT](LICENSE) © [DeceptiCloud](https://github.com/apennismightier)

mega.nz, Mega, and the Mega logo are trademarks of Mega Limited. This project is not affiliated with or endorsed by Mega Limited.
