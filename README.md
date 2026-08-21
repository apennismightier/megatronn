<div align="center">
MegaTronn

A desktop client for MEGA.nz — link containers, multi-connection downloads, bulk account management, and inactivity protection, without a browser tab open.

MegaTronn is an Electron desktop app for people who work with a lot of MEGA links and a lot of MEGA accounts. It parses link containers, downloads over multiple connections with real resume, manages accounts in bulk, and keeps stored files from being purged for inactivity.

Built on megajs for the MEGA protocol, React for the UI, and packaged as a per-user install that never needs admin rights.

Features
⬇️ Downloads and transfers
Multi-connection chunked downloads with configurable connection count and chunk size
Resume from disk — progress is persisted to a .part file plus a sidecar, so an interrupted transfer restarts from the exact byte it reached, surviving app restarts, network drops and IP changes
Bandwidth-limit handling — when MEGA's transfer quota is hit the transfer pauses, keeps its partial file, and retries on a short interval; change your IP and it resumes from saved progress instead of starting over
429 rate-limit backoff with automatic retry
Integrity checking — optional MAC verification of completed downloads
Duplicate detection, so an already-downloaded file doesn't spend transfer quota twice
Uploads to a chosen folder
Queue control — reorder, pause/resume, retry failed, clear completed, copy failed links, reveal in folder
Sleep prevention while transfers are active
🔗 Link parsing
Extract every MEGA link from arbitrary pasted text
Import DLC, CCF and RSDF containers, including base64-wrapped payloads
Registered as file associations — double-click a container or drop it on the exe
Clipboard monitor — optionally collect MEGA links as you copy them
Folder browsing — expand a folder link and select individual files
My Links — links saved per account
☁️ Cloud Drive

mega.nz embedded in the app with auto-login for the selected account.

👥 Account management
Accounts organised into groups, with search and bulk operations
Passwords encrypted at rest via Electron's safeStorage (OS keychain) — the store file is useless if copied to another machine
Sessions are closed properly using MEGA's sml session logout, so live sessions don't pile up on an account over time
🔄 Import / Export wizard

Bulk account lists arrive in whatever shape someone saved them in. Rather than guessing and silently getting it wrong, the wizard shows how it intends to read your file and lets you fix it first.

Formats: CSV, TXT, TSV, YAML, JSON, four-line blocks, plain email:password
Separator detection — comma, semicolon, tab, colon, pipe, space; records split by line, blank line, semicolon, pipe, or nothing at all
Files with no line breaks are split every N fields. That number is genuinely underivable from the text — a@b.com,p1,a@c.com,p2 is either one 4-field record or two 2-field records — so it is estimated, labelled as an estimate, and left editable
Column mapping — Name / Email / Password / Group / Links / Ignore, each showing a sample value from your own file
Group import toggle for files with no group column
"Last column keeps extra separators" so URLs and passwords containing the separator survive intact
Live preview of exactly how rows will be saved, which rows will be skipped and why, and an explicit approval step before anything is written
Export to CSV or TXT with delimiter choice, optional header and per-column selection; values containing the delimiter, a quote or a newline are quoted automatically, so exports re-import cleanly
♻️ Account Refresher

MEGA deletes the files in an account after roughly three months of inactivity (ToS 37.4) — the account itself survives. The Refresher keeps stored data alive by logging in on a schedule.

Capacity planning — logins/day required vs your cap, with a loud warning when a full sweep would take longer than the inactivity window and the minimum cap that fixes it
At-risk list — accounts the daily cap is about to push past their deadline
Needs-attention list — wrong password or 2FA, paused rather than retried forever
Pacing controls — max logins/day, interval, inter-login gap, safety margin
Jittered scheduling (±7 days/account) so bulk-added accounts don't clump
Error classification by numeric MEGA code, never by string matching — see Architecture
Defers, never preempts — an account open in Cloud Drive or used by a transfer is skipped until free
Per-account Test button — log in right now and see the real result
Off by default
⚕️ Rescue

A separate tool driving an isolated browser session against mega.nz, for accounts needing manual recovery rather than a routine login. Storage is cleared between accounts.

🩺 Diagnostics

Built-in log viewer, one-click diagnostic export, live performance monitor, and an auto-updater that checks GitHub Releases and installs in place.

Install

Grab the latest installer from Releases.

[!WARNING] The installer is unsigned. Windows SmartScreen will warn you — click More info → Run anyway. Building from source avoids this entirely.

Installs per-user to %LocalAppData%\Programs\MegaTronn. No admin rights required, which also means in-app updates never need elevation and pinned taskbar shortcuts survive them.

Build from source

Requirements

Node.js 20.19+ or 22.12+ — electron-vite 5 refuses older 20.x releases
npm 10+
bash
npm install
npm run dist:win

Or run build.bat, which self-elevates, clears stale caches, verifies the compiled renderer actually changed, then packages. The installer lands in dist\.

bash
npm run dev      # hot-reload development
npm run build    # compile only, no packaging
Testing

The scheduling and parsing engines are deliberately pure — no Electron imports, no I/O — so they run under plain Node with no test framework:

bash
cd test
node --experimental-strip-types --no-warnings account-format.test.ts      # 58 assertions
node --experimental-strip-types --no-warnings refresher-schedule.test.ts  # 118 assertions
node --experimental-strip-types --no-warnings session-registry.test.ts    # 19 assertions

Keep them pure. If you add a branch to an engine, add a test. Both engines have shipped bugs that these suites caught before release.

Project structure
src/
  main/                      Electron main process
    transfer-manager.ts      Download engine — chunking, resume, quota + 429 handling
    account-format.ts        PURE: import/export parsing, mapping, serialization
    refresher-schedule.ts    PURE: all refresher scheduling decisions
    refresher.ts             Refresher runtime — megajs ping, batching, persistence
    session-registry.ts      PURE: session leases (refresher defers to user/transfers)
    accounts.ts              Account persistence, safeStorage encryption
    mega-service.ts          megajs Storage session wrapper
    reactivator.ts           Browser automation for the Rescue tab
    link-parser.ts           DLC / CCF / RSDF container parsing
    creator/                 Account creation subsystem
    ipc-handlers.ts          All IPC
  preload/                   contextBridge surface (window.api)
  renderer/                  React UI
test/                        Plain-node test suites
Architecture

A few decisions that are load-bearing and shouldn't be reversed without reading why:

Pure engines. All scheduling and parsing logic lives in modules with zero Electron imports. That's what makes them testable with plain Node, and it's why bugs get caught before they ship rather than after. Date maths belongs in refresher-schedule.ts, not in the runtime around it.

Errors are classified by numeric code, not by string matching. An earlier version regex'd error strings, which made a throttle response and a genuinely blocked account indistinguishable — everything looked "deactivated". Now: 429 and -3 mean we are being throttled and never count against the account; -16 is the only real blocked signal and needs two consecutive hits before an account is flagged; anything unrecognised is treated as transient. An account is never marked dead because parsing failed.

One session lease covers everything. Transfers don't open their own MEGA sessions — they borrow the user's. So a lease taken at login covers Cloud Drive browsing and in-flight transfers alike, and the refresher simply skips anything held. Leases self-expire after 12 hours so a crash can't wedge an account permanently.

Refreshing is not rescuing. The refresher uses the megajs API (~1s per account, no browser fingerprint, clean session teardown). The browser-driving Rescue tool is kept for what it's actually good at: manually recovering an account that needs a real browser session.

Unknown is not the same as fine. Imported accounts carry lastSeenAt: null rather than a fabricated timestamp. Pretending an imported account was "active just now" sets a deadline three months out for an account that might be days from having its files purged.

Contributing

Issues and pull requests welcome. When reporting a transfer or login problem, please attach the diagnostic export (Log tab → export) — it carries the actual numeric error codes, which is usually the difference between a diagnosis and a guess.

If you change a pure engine, run its test suite and add cases for the new branch.
Changelog

See CHANGELOG.md for the full history.
