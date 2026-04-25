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

<div align="center">

*"Decepticons, DOWNLOAD!"*

</div>
