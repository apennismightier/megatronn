<div align="center">

<a href="https://github.com/apennismightier/megatronn">
  <img src="https://i.ibb.co/zhn5pDvf/Mega-Tronn.png" alt="MegaTronn" width="400">
</a>

# MegaTronn

**Powered by DeceptiCloud**

[![Version](https://img.shields.io/github/v/release/apennismightier/megatronn?label=latest&color=22c55e)](https://github.com/apennismightier/megatronn/releases)
[![Platform: Windows](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4?logo=windows)](#installation)
[![Built with Electron](https://img.shields.io/badge/Electron-43-47848F?logo=electron)](https://www.electronjs.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript)](https://www.typescriptlang.org/)
[![Vulnerabilities](https://img.shields.io/badge/npm%20audit-0-22c55e)](#privacy--safety)

</div>

---

## What is it?

MegaTronn is an unofficial desktop client for [Mega.nz](https://mega.nz/) that turns the painful workflow of "open browser, paste link, log in, click download" into a one-window experience. Manage hundreds of Mega accounts, parse links out of any messy paste, download in parallel at full bandwidth, and upload through an embedded browser that actually behaves like a real one.

Built because the official MEGAsync client doesn't handle multi-account workflows, link-list ingestion, or batch maintenance — and because the browser webclient stalls out the moment you try to do anything at scale.

> **Disclaimer:** MegaTronn is independent third-party software. It is not affiliated with, endorsed by, or connected to Mega Limited. You are responsible for ensuring your use complies with Mega's Terms of Service.

---

## Features

### Multi-Account Management
- Unlimited accounts organized into named groups (`Personal`, `Archive-A`, etc.)
- Collapsible groups, alphabetical sort, multi-select with bulk move/delete
- Resizable account sidebar (180–520px), state persisted across restarts
- Custom group autocomplete that scales to hundreds of groups (vanilla `<datalist>` broke at 100+)
- Passwords encrypted at rest via OS keychain (`safeStorage`) — copy the store to another machine and it's unreadable blobs
- Sessions closed with Mega's real `sml` session logout, so you don't accumulate dozens of live sessions per account

### Import / Export Wizard
Bulk lists arrive in whatever shape someone saved them in. Rather than guessing and silently getting it wrong, the wizard shows you how it intends to read your file — and lets you fix it before anything is written.
- **Formats**: CSV, TXT, TSV, YAML, JSON, 4-line blocks, `email:password` colon lines — pasted or from file
- **Separator detection**: comma, semicolon, tab, colon, pipe, space; records split by line, blank line, `;`, `|`, or nothing at all
- **Files with no line breaks** are split every N fields. That number is genuinely underivable from the text — `a@b.com,p1,a@c.com,p2` is either one 4-field record or two 2-field records — so it's estimated from `values ÷ emails`, *labelled as an estimate*, and left editable
- **Column mapping**: assign each column to Name / Email / Password / Group / Links / Ignore, with a live sample value from your own file under each
- **Group toggle** for lists that have no group column; **"last column keeps extra separators"** so `email:pass:https://mega.nz/...` doesn't explode on the URL's own colons
- **Live preview table** of exactly how rows will be saved, which rows will be skipped and why, plus an explicit approval tick before Import unlocks
- **Export** to CSV or TXT with delimiter choice, optional header, per-column selection — values containing the delimiter, a quote, or a newline are auto-quoted, so exports re-import cleanly

### Link Parser
- Extracts Mega URLs from arbitrary text — single and double Base64 decoded
- Container file support: drag-and-drop or "Open with..." for **`.dlc`**, **`.ccf`**, **`.rsdf`**, and **`.txt`**
- **Clipboard auto-capture** routes copied Mega links into the parser queue — no surprise downloads, you click when you're ready
- **📂 Browse folder** any `/folder/` link inline: see every file inside, untick what you don't want, download only what you pick

### Parallel Downloader
- **6 connections per file, 20 MB chunks** — matched to MegaBasterd's proven throughput profile (megajs's 1 MB default means 20× the requests and a fraction of the speed)
- 3 simultaneous transfers by default; bump higher if your bandwidth supports it
- Decrypt and reassembly run in a **utility process**, off the main thread — several files at once no longer freeze the UI
- **Resume from disk**: progress persists to a `.part` file plus a sidecar, so an interrupted transfer restarts from the exact byte it reached — across app restarts, network drops, and IP changes
- **Quota wall handling**: hit Mega's transfer limit and the download pauses with its partial file intact, retrying on a short interval — change your IP and it picks up where it left off instead of starting over
- 429 rate-limit backoff; anonymous fallback when an account session expires (`ESID`)
- Optional MAC integrity verification; duplicate detection so a file already on disk never spends quota twice
- Per-transfer pause/resume, bulk pause-all / resume-all / cancel-all, queue reordering
- ▶ **Open** file with default app, 📁 **Show in Explorer** on completed rows

### Embedded Cloud Drive
- mega.nz rendered inline as a fully-functional browser tab — auto-login with stored credentials, persistent session per account
- **Auto-transfer worker hook** eliminates the "Data cannot be cloned, out of memory" crash that kills uploads of large files in stock embedded browsers (Mega's upload code clones every chunk via `postMessage` — we transfer ownership instead, zero allocation)
- 16 GB V8 heap headroom for large file workloads
- 🗑 **Clear Cache** button for one-click session reset when Mega's IndexedDB cache fills up
- File-path privacy: your local upload paths are hidden from mega.nz's page (uploads show only filenames, never `C:\Users\you\Documents\...`)
- Auto-recovery: if an upload fails with memory pressure anyway, the webview silently clears storage and reloads — you keep working

### ↻ Account Refresher
Mega deletes the **files** in an account after ~3 months of inactivity (ToS 37.4) — the account itself survives. The Refresher keeps stored data alive by logging in on a schedule, via the `megajs` API in about a second per account, with a clean `sml` logout afterwards.
- **Capacity planning**: logins/day your account count actually needs vs your configured cap — and a loud warning, with the minimum cap that fixes it, when a full sweep would take longer than the inactivity window
- **At-risk list**: accounts the daily cap is about to push past their deadline, with a one-click "run now, ignore today's cap"
- **Needs-attention list**: wrong password or 2FA — paused rather than retried forever, with per-account retry once you've fixed it
- **Pacing controls**: max logins/day, refresh interval, gap between logins, safety margin. Spacing matters more than raw count
- **±7 day jitter per account**, so a 40-account Creator run doesn't land 40 logins on one future day
- **Errors classified by numeric Mega code, never string matching**: `429`/`-3` mean *we're* throttled and never count against the account; `-16` is the only real "blocked" signal and needs two consecutive hits; anything unrecognised is treated as transient. An account is never killed because parsing failed
- **Defers, never preempts**: an account open in Cloud Drive or in use by a transfer is skipped until free
- **🧪 Test button** on every row — log in right now and see the real result and error code, without waiting weeks for the schedule
- Ships **off by default**

### ⚕ Account Rescue
- Headless browser-based reactivation for dormant or suspended accounts (the manual counterpart to the Refresher — rescuing and refreshing are different jobs)
- Batch queue with configurable delays; storage and cache cleared between accounts
- CAPTCHA escalation to the user when automated solving fails

### Quality of Life
- **In-app auto-update** — top-banner notification when a newer release is published, then download and install without leaving the app; per-user install means it never needs admin rights
- Embedded log viewer with live tail, level filter (`INFO` / `WARN` / `ERROR`), and one-click Desktop export
- Live performance monitor (memory + transfer stats)
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

> The installer is unsigned, so SmartScreen will warn on first run: **More info → Run anyway**. Building from source avoids this.

### From source

```bash
git clone https://github.com/apennismightier/megatronn.git
cd megatronn
npm install
npm run dev        # development with HMR
# or
npm run dist:win   # produces dist/MegaTronn-Setup-<version>.exe
```

Requirements: **Node.js 20.19+ or 22.12+** (electron-vite 5 refuses older 20.x), npm 10+.
Windows users can just run `build.bat`, which self-elevates, clears stale caches, verifies the compiled renderer actually changed, and packages in one step.

---

## Tech Stack

| Layer        | Stack                                                          |
|--------------|----------------------------------------------------------------|
| Runtime      | Electron 43 (Chromium 150, Node 24)                            |
| UI           | React 18 + TypeScript 5 + Tailwind CSS                         |
| Build        | electron-vite 5 + Vite 7 (renderer + main), electron-builder 26 NSIS (installer) |
| State        | Zustand                                                        |
| Persistence  | electron-store (settings, accounts, refresher state), JSON (transfer queue) |
| Logging      | electron-log → `%AppData%\megatronn\logs\main.log`             |
| Mega API     | [`megajs`](https://github.com/qgustavor/mega) (TypeScript)     |
| Tests        | Plain Node type-stripping — no framework, no runner            |

Scheduling and parsing logic lives in **pure modules with zero Electron imports** (`account-format.ts`, `refresher-schedule.ts`, `session-registry.ts`). That's what makes them testable with plain Node, and it's why bugs in them get caught before release rather than after:

```bash
cd test
node --experimental-strip-types --no-warnings account-format.test.ts      # 58 assertions
node --experimental-strip-types --no-warnings refresher-schedule.test.ts  # 118 assertions
node --experimental-strip-types --no-warnings session-registry.test.ts    # 19 assertions
```

---

## Configuration

Everything is in-app under **Settings** (gear icon). Defaults are tuned for Mega's optimal throughput — most users never need to touch them.

- **Downloads**: max concurrent transfers, default destination, incomplete-file directory
- **Performance**: connections per file, chunk size, worker count — auto-floored to sane values if set too low
- **Integrity**: optional MAC verification of completed downloads
- **Paths**: where downloaded files land, where the log lives
- **Rescue**: headless on/off, delay range between attempts
- **Refresher**: max logins/day, interval, inter-login gap, safety margin (in the Refresher tab)

Settings, accounts, refresher state, and the transfer queue live in `%AppData%\megatronn\`. The app **does not** make outbound network requests other than to mega.nz and to GitHub (once per launch, for the update check).

---

## Privacy & Safety

- **No telemetry.** No analytics SDK, no error reporting service, no "usage data" of any kind.
- **No ads.** Nothing is monetized.
- **Local-only state.** Accounts, settings, refresher state, transfer queue, logs — all on your disk, never transmitted.
- **One outbound check** per app launch to `api.github.com` for the version check (anonymous, no identifier sent beyond a `User-Agent: MegaTronn/<version>` header).
- **Account passwords are encrypted at rest** with the OS keychain via `safeStorage`, not stored in plaintext.
- The embedded browser sandboxes each account in its own session partition — accounts cannot see each other's cookies, storage, or upload state.
- Runs on **Electron 43 / Chromium 150**, and `npm audit` reports **0 vulnerabilities**.

> ⚠️ **Account exports contain passwords in plain text.** That's the point of a portable backup, and the app makes you confirm before writing one. Store it securely and delete it when done — or untick the Password column to export a list without credentials.

---

## Roadmap

- [ ] **Smart proxy** — HTTP / SOCKS4 / SOCKS5, per-chunk rotation, configurable ban duration on error (settings scaffolding exists; the transport layer is not wired up yet)
- [ ] **Automatic VPN rotation on quota** — detect the transfer-limit wall and cycle a WireGuard config pool instead of asking you to change IP by hand
- [ ] macOS + Linux builds
- [ ] Bandwidth limiter (throttle download speed)
- [ ] Scheduled / time-windowed transfers
- [ ] Webhook notifications on completion
- [ ] Folder-level upload queueing from local disk

Open an issue if there's something you'd want.

---

## Contributing

Issues and PRs welcome. Please attach a diagnostic log (`Settings → Export Diagnostic`) when reporting bugs — it bundles the main log, transfer queue state, settings, and system info in one text file. For transfer or login problems it's especially worth it: every attempt records Mega's raw numeric error code, which is usually the difference between a diagnosis and a guess.

If you're submitting a PR, please:
- Bump the version in `package.json` (everything else reads `app.getVersion()`)
- Run the test suites above if you touched a pure engine, and add cases for any new branch
- Add a note to [CHANGELOG.md](CHANGELOG.md)

---

## License

© [DeceptiCloud](https://github.com/apennismightier)

mega.nz, Mega, and the Mega logo are trademarks of Mega Limited. This project is not affiliated with or endorsed by Mega Limited.
