<div align="center">

[![MegaTronn](https://i.ibb.co/zhn5pDvf/Mega-Tronn.png)](https://github.com/apennismightier/megatronn)

# MegaTronn

**Powered by DeceptiCloud**

An open-source Mega.nz desktop client for Windows — built for speed, bulk account management, and automation.

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-purple.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-blue.svg)]()
[![Version](https://img.shields.io/badge/Version-1.0.1-brightgreen.svg)]()

</div>

---

## What is MegaTronn?

MegaTronn is an open-source Mega.nz desktop client for Windows, built for people who manage large numbers of accounts and need fast, parallel downloads. It was inspired by MegaBasterd and built from the ground up in Python with a focus on throughput, multi-account management, and automation.

---

## Features

### Downloads
- **Parallel chunked downloads** — up to 32 simultaneous worker threads per file, each with its own independent Mega session and HTTP keep-alive connection
- **8 MB chunks by default** (configurable 1–64 MB) — dramatically reduces round-trip overhead compared to Mega's internal 1 MB scheme
- **Public folder support** — paste a folder link and every file inside is queued automatically, with the correct share context passed to each chunk request
- **Chunk-level resume** — downloads interrupted by pausing, closing the app, or a crash resume from exactly where they left off
- **Persistent queue** — the download list survives restarts; running downloads become queued and pick up automatically
- **Pause / Resume / Cancel** per transfer, or globally with Pause All / Resume All / Cancel All
- **DLC / link container import** — supports `.dlc`, `.ccf`, `.rsdf`, and `.txt` formats

### Link Parser
- **Automatic clipboard monitoring** — detects Mega links the instant you copy them, no clicking required
- **Base64 decoding** — automatically decodes links encoded up to 2 layers deep
- **Pre-download review** — inspect links before queuing, remove individual files

### Account Manager
- **Multi-account support** — unlimited accounts, fast switching
- **Account Groups** — organise accounts into collapsible named groups
- **Sort** by name, email, or date added — ascending or descending
- **Live search** — filters accounts and groups simultaneously as you type
- **Rename** accounts or groups in-place (F2 or right-click)
- **Edit Credentials** — change display name, email, and password for any account
- **Move between groups** — single or bulk selection
- **YAML bulk import** — import hundreds of accounts at once with optional group assignment
- **Ctrl+A** to select all, then bulk delete
- **Double-click to login** — automatically switches to the Cloud Drive tab on success

### Account Reactivator
- Opens a **headless Edge or Chrome browser** (invisible) and logs into mega.nz for accounts the API won't accept
- **Triggered automatically** when a login fails — prompts you to attempt browser-based reactivation
- Handles **CAPTCHA** by reopening with a visible browser so you can solve it manually
- **Batch mode** — reactivate multiple accounts sequentially with a 3-second delay between each to avoid rate limiting
- Live progress dialog showing per-account status: Reactivated / Wrong Password / Not Found / Rate Limited / Suspended
- Uses Selenium 4.6+ Selenium Manager to **auto-download the correct Edge/Chrome driver** — no manual setup

### Cloud Drive
- Browse your Mega cloud drive after logging in
- Download files or entire folders with a single click
- **Drag and drop upload** — drag files from Explorer onto the Cloud Drive panel
- Upload progress bar with live speed readout
- New Folder creation

### Smart Proxy
- Auto-detects system/VPN proxy from environment variables and Windows registry (enabled by default)
- Optional proxy rotation list (HTTP/SOCKS4/SOCKS5) for sustained downloads
- Per-chunk rotation, configurable ban duration for failing proxies

### Logging
- Rotating log file at `%USERPROFILE%\.megatronn\logs\megatronn.log`
- Live log viewer — **Tools → View / Export Log** — with level filtering and auto-refresh
- One-click export to Desktop as a timestamped `.txt` file for diagnostics

---

## Installation

### Requirements
- Windows 10 or 11 (64-bit)
- Microsoft Edge or Google Chrome *(for Account Reactivator — Edge is built into every Windows 10/11 install)*

### Install
1. Download **MegaTronn_Setup_v1.0.1.exe** from the [Releases](https://github.com/apennismightier/megatronn/releases) section
2. Run the installer and follow the prompts
3. Launch MegaTronn from the Start Menu or Desktop shortcut

> No Python, no command line, no additional setup required.

---

## YAML Account Import Format

Import hundreds of accounts at once via **Import YAML** in the sidebar:

```yaml
accounts:
  - name: "Show Name (14 GB)"
    email: "user@example.com"
    password: "yourpassword"
    group: "Show Name"          # auto-creates the group in the sidebar
    links:
      - "https://mega.nz/folder/XXXXXXXX#YYYYYYYY"

  - name: "Standalone Account"
    email: "solo@example.com"
    password: "pass"
    # no group: = added to Ungrouped
```

Accepted aliases: `user` or `username` for email, `pass` or `pwd` for password, `files` for links.

---

## Performance

MegaTronn's parallel download engine was designed to address the bottlenecks in standard mega.py:

| Setting | Default | Notes |
|---|---|---|
| Workers per download | 8 | Each with its own Mega session + HTTP keep-alive (configurable 1–32) |
| Chunk size | 8 MB | vs mega.py's 1 MB scheme — 8× fewer round-trips (configurable 1–64 MB) |
| Max simultaneous downloads | 3 | Configurable in Settings |
| Smart Proxy | On | Auto-detects system/VPN proxy from Windows registry and environment variables |

---

## Known Limitations

- **Account reactivation** — works for accounts that Mega has soft-suspended (last active flag). Accounts permanently deleted by Mega will show "Not Found" and cannot be recovered.
- **Upload speed** — uploads are single-threaded (Mega's API does not support parallel upload streams). Progress is shown live.
- **Folder downloads** — requires the folder share key to be present in the link. Private folders require being logged in to the owning account.

---

## Legal

Released under the **GPL-3.0 License**. See [LICENSE](LICENSE).

MegaTronn is not affiliated with, endorsed by, or connected to Mega Limited. Use responsibly and in accordance with [Mega's Terms of Service](https://mega.io/terms).

Inspired by [MegaBasterd](https://github.com/tonikelope/megabasterd) by tonikelope (GPL-3.0).

---

<div align="center"><i>"Decepticons, DOWNLOAD!"</i></div>
