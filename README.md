<div align="center">

<img src="assets/MegaTronn.png" width="320" alt="MegaTronn" />

# MegaTronn

**Powered by DeceptiCloud**

A high-performance, open-source Mega.nz desktop client for Windows.

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-purple.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()

</div>

---

## What is MegaTronn?

MegaTronn is a feature-rich desktop client for [Mega.nz](https://mega.nz) inspired by [MegaBasterd](https://github.com/tonikelope/megabasterd). It was built from the ground up in Python with a focus on:

- **Throughput** — parallel chunked downloads with per-worker HTTP keep-alive connections
- **Flexibility** — multi-account management with groups, YAML import, and per-account link libraries  
- **Automation** — live clipboard monitoring that detects Mega links (plain or Base64-encoded) the instant you copy them
- **Transparency** — every download is visible, stoppable, and persists across restarts

---

## Screenshots

> *Add screenshots here once the app is running on your machine.*

---

## Features

### Downloads
- **Parallel chunked downloads** — up to 32 simultaneous worker threads per file, each with its own independent Mega session and HTTP keep-alive connection
- **8 MB chunks** by default (configurable 1–64 MB) — dramatically reduces round-trip overhead vs. Mega's internal 1 MB scheme
- **Live progress** — real filename, bytes transferred, speed (MB/s), and per-worker status
- **Pause / Resume / Stop** per transfer, or **Pause All / Resume All / Cancel All** globally
- **Persistent queue** — downloads survive closing and restarting the app
- **DLC / link container import** — `.dlc`, `.ccf`, `.rsdf`, `.txt` file formats supported
- **Folder link support** — fetches file list from public folder links and queues each file

### Link Parser
- **Automatic clipboard monitoring** — detects Mega links the instant you copy them, no clicking required
- **Base64 decoding** — automatically decodes links encoded up to 2 layers deep
- **Pre-download review** — expand any link to see its contents; remove individual files before confirming
- **Single "Download" button** — queues everything in one click

### Account Manager
- **Multi-account support** — unlimited accounts with fast switching
- **Account Groups** — organise accounts into collapsible groups (e.g. by show name or genre)
- **Sort** by name, email, or date added — ascending or descending
- **Search** — live filter across account names, emails, and group names simultaneously
- **Rename** accounts or groups in-place (F2 or right-click)
- **Move between groups** — single accounts or bulk selection
- **YAML import** — import hundreds of accounts at once from a structured YAML file, with optional `group:` field for automatic group assignment
- **Per-account link library** — store and manage links associated with each account
- **Base64 encoder** — encode any stored link to Base64 ×1 or ×2 with one-click copy

### Smart Proxy (optional)
- Rotate across a list of HTTP/SOCKS4/SOCKS5 proxies
- Automatically ban slow or failing proxies (configurable ban duration)
- Auto-refresh proxy list from a URL
- Per-chunk rotation for maximum anonymity
- Falls back to system proxies / VPN when Smart Proxy is disabled

### Cloud Drive
- Browse your Mega.nz cloud drive when logged in
- Download files directly from the file tree
- Upload files to your account

---

## Installation

### Requirements

- **Python 3.9 or later** — [python.org](https://python.org/downloads/)
- **pip** (bundled with Python)

### Quick start

```bash
git clone https://github.com/your-username/megatronn.git
cd megatronn

pip install -r requirements.txt

python main.py
```

### One-click Windows build

Drop `build_windows.bat` into the project folder and double-click it. It will:

1. Install all dependencies automatically
2. Fix known `mega.py` compatibility issues (`pathlib` backport, `tenacity` async patch)
3. Build a standalone `MegaTronn.exe` with the app icon embedded

Output: `dist\MegaTronn\MegaTronn.exe`  
To share: zip the entire `dist\MegaTronn\` folder — no Python required on the recipient's machine.

---

## Account YAML Format

MegaTronn can import accounts in bulk from a YAML file. Each account can optionally be assigned to a group, which will be auto-created in the sidebar.

```yaml
accounts:
  - name: "Ancient Aliens (14.14 GB)"
    email: "user@example.com"
    password: "yourpassword"
    group: "Ancient Aliens"          # creates/assigns to this group
    links:
      - "https://mega.nz/folder/XXXXXXXX#YYYYYYYY"

  - name: "Bar Rescue S01"
    email: "other@example.com"
    password: "password"
    group: "Bar Rescue"

  - name: "Standalone Account"
    email: "solo@example.com"
    password: "pass"
    # no group: field = ungrouped
```

Accepted field aliases: `user` / `username` for email, `pass` / `pwd` for password, `files` for links. If a `group:` value references a group that doesn't exist yet, MegaTronn creates it automatically.

---

## Architecture

```
megatronn/
├── main.py                          Splash screen + app entry point
├── requirements.txt
├── build_windows.bat                One-click Windows build script
├── _fix_deps.py                     mega.py compatibility fixer (called by bat)
├── assets/
│   ├── MegaTronn.png               Splash screen artwork
│   ├── MegaTronnIcon.png           Source icon
│   ├── icon.ico                    Multi-resolution Windows icon (16–256px)
│   └── icon.png                    Linux icon
├── core/
│   ├── mega_wrapper.py             Parallel downloader, SmartProxyManager, queue
│   ├── account_manager.py          Account + group CRUD, YAML import, sorting
│   ├── link_parser.py              Mega URL extraction + Base64 decoding
│   ├── dlc_parser.py               DLC / CCF / RSDF / TXT container parser
│   └── settings.py                 Persistent JSON settings
└── ui/
    ├── splash.py                   Animated splash screen
    ├── main_window.py              Root window, menu bar, layout
    ├── account_panel.py            Sidebar: groups, accounts, search, rename
    ├── link_panel.py               Link Parser tab + clipboard monitor
    ├── account_links_panel.py      Per-account link library + Base64 encoder
    ├── download_panel.py           Download queue with live progress rows
    ├── file_browser.py             Cloud Drive tab
    ├── settings_panel.py           Preferences window (5 tabs)
    └── theme.py                    Dark theme palette + ttk style map
```

### Parallel Download Architecture

Modelled on MegaBasterd's `ChunkDownloader` / `ChunkWriterManager` pattern:

```
Download task (e.g. 5.4 GB file)
│
├── [Worker 0]  own Mega session + requests.Session()
│   └── fetches chunks 0, 8, 16 …  (8 MB each, HTTP Range requests)
├── [Worker 1]  own Mega session + requests.Session()
│   └── fetches chunks 1, 9, 17 …
├── [Worker 2]  own Mega session + requests.Session()
│   └── fetches chunks 2, 10, 18 …
│   …
└── [Worker 7]  own Mega session + requests.Session()
    └── fetches chunks 7, 15, 23 …
         │
         ▼
    ChunkWriter (single thread)
    ├── receives decoded chunks out-of-order
    ├── buffers until next sequential chunk is ready
    └── writes to disk in strict order → final file
```

Each worker has its **own independent Mega session** — the key difference from naive implementations that share one session and serialise all API calls. With 8 workers and 8 MB chunks, a 5 GB file requires ~625 API round-trips split across 8 connections rather than 5,000+ sequential 1 MB requests.

---

## Settings

All settings are stored in `~/.megatronn/settings.json` and editable via **Settings** in the title bar.

| Setting | Default | Description |
|---|---|---|
| Workers per download | 8 | Parallel chunk download threads (1–32) |
| Chunk size per worker | 8 MB | HTTP byte-range size per request (1–64 MB) |
| Max simultaneous downloads | 3 | Concurrent download tasks |
| Complete files folder | `~/Downloads/MegaTronn` | Where finished files land |
| Incomplete files folder | `~/Downloads/MegaTronn/.incomplete` | Staging area during download |
| Smart proxy | Off | Enable / configure proxy rotation |
| Auto-clear completed | Off | Remove done transfers automatically |

---

## Building from Source

### Windows — standalone `.exe`

```bat
build_windows.bat
```

### Any platform — Python script

```bash
python build.py                   # one-folder build
python build.py --onefile         # single binary
python build.py --clean           # wipe previous build first
```

### Linux — system install + `.deb`

```bash
# System install (adds megaTronn command + app launcher icon)
sudo bash install_linux.sh

# Build a .deb package
python3 build.py --deb

# Uninstall
sudo bash install_linux.sh --remove
```

### macOS — `.app` bundle + `.dmg`

```bash
python3 build.py
open dist/MegaTronn.app

# Create a distributable .dmg
hdiutil create -volname "MegaTronn" \
  -srcfolder dist/MegaTronn.app \
  -ov -format UDZO \
  dist/MegaTronn.dmg
```

---

## Known Limitations

| Limitation | Notes |
|---|---|
| Public folder downloads | Fetches file list and queues individual files. Requires login for large private folders. |
| Resume from partial | Downloads restart from zero if interrupted. Partial files are cleaned up automatically. |
| Upload progress | Upload progress bar is indeterminate (mega.py limitation). |
| Python 3.14 + pathlib | `build_windows.bat` automatically removes the conflicting `pathlib` backport before building. |

---

## Dependencies

| Package | Purpose |
|---|---|
| `mega.py` | Mega.nz API (authentication, file metadata, CDN URL negotiation) |
| `requests` | HTTP chunk downloads with keep-alive sessions |
| `pyyaml` | YAML account import/export |
| `cryptography` | AES-CTR chunk decryption |
| `pyperclip` | Clipboard access for link detection |
| `Pillow` *(optional)* | Splash screen image display |

---

## Contributing

Pull requests welcome. Before submitting:

```bash
# Run the test suite (no external dependencies needed)
python -m pytest tests/ -v          # if you add tests
python -c "import ast, os; [ast.parse(open(p).read()) for r,d,fs in os.walk('.') for f in fs if f.endswith('.py') for p in [os.path.join(r,f)]]"
```

Areas where contributions are especially welcome:
- Resume-from-partial (chunked download checkpointing)
- macOS native notifications
- Proper MEGA SDK integration for authenticated folder downloads
- Dark/light theme toggle

---

## Legal

MegaTronn is released under the **GPL-3.0 License**. See [LICENSE](LICENSE).

MegaTronn is not affiliated with, endorsed by, or connected to Mega Limited. Use responsibly and in accordance with [Mega's Terms of Service](https://mega.io/terms).

Inspired by [MegaBasterd](https://github.com/tonikelope/megabasterd) by tonikelope (GPL-3.0).

---

<div align="center">

*"Decepticons, DOWNLOAD!"*

<img src="assets/MegaTronnIcon.png" width="80" alt="MegaTronn Icon" />

</div>
