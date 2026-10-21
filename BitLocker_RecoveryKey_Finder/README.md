# BitLocker Recovery Key Finder

A forensic tool that **recovers BitLocker 48-digit recovery keys that were saved on
a disk** — including keys in **deleted / unallocated space** (carved) and in **live
files** (even hidden ones) — from disk images *and* from physical disks directly.

It does **not** crack, brute-force, or guess anything. It only finds recovery keys
the user actually saved. That is exactly why it is reliable and fast.

> **For authorized forensic examination only.** Use it on media you own or are
> authorized to analyze.

---

## Table of contents

- [What it does](#what-it-does)
- [Why it is reliable (no false positives)](#why-it-is-reliable-no-false-positives)
- [Features](#features)
- [Supported input](#supported-input)
- [Quick start](#quick-start)
- [Usage](#usage)
  - [1. Console (CLI)](#1-console-cli)
  - [2. Local dashboard (disk picker + provenance tree)](#2-local-dashboard-disk-picker--provenance-tree)
  - [3. Shared server (intranet)](#3-shared-server-intranet)
- [Output files](#output-files)
- [How it works](#how-it-works)
- [Administrator / root rights](#administrator--root-rights)
- [Performance & tuning](#performance--tuning)
- [Building the executable](#building-the-executable)
- [Protecting the code (obfuscation & distribution)](#protecting-the-code-obfuscation--distribution)
- [Deployment](#deployment)
- [Limitations](#limitations)
- [Legal](#legal)

---

## What it does

A BitLocker recovery password is 8 groups of 6 digits (48 digits total). When a
user enables BitLocker, Windows asks them to save this key — to a file, a USB
stick, a printout, or their Microsoft account. This tool finds those saved copies
on the media, wherever they ended up:

- **Live files** — e.g. `BitLocker Recovery Key *.txt`, documents, notes — including
  **hidden** files.
- **Deleted / unallocated space** — a saved key file that was deleted usually still
  exists in unallocated space until overwritten; carving recovers it.
- **Any filesystem** (NTFS / FAT / exFAT) and any partition, because carving works
  on the raw bytes.

It then reports each key with its **Recovery Key ID (GUID)** so you can match it to
the locked volume, and shows **where** each key was found (folder + file, or byte
offset for deleted data).

## Why it is reliable (no false positives)

Each 6-digit group of a real BitLocker recovery password equals `word x 11`
(`word` in 0..65535). That divisibility-by-11 rule is a built-in checksum. The tool
validates every candidate against it:

- each group must be divisible by 11, and
- `group / 11` must be <= 65535.

A random 48-digit string passes all 8 groups with probability ~`3e-10`. In a test
of 1,000,000 random strings, **0** passed. So every key the tool reports is a
mathematically valid BitLocker recovery key.

## Features

- **Carves saved + deleted keys** from the whole raw media (allocated + unallocated).
- **Scan disk images or physical disks directly** — no imaging step required.
- **Local web dashboard** with:
  - a **disk picker** listing this machine's physical disks, USB drives and volumes;
    select and scan them **in place (no copy)**.
  - drag & drop of image files/folders.
  - a **provenance tree**: results grouped as **disk -> folder -> file -> key**, with
    **hidden** files tagged and **deleted/unallocated** keys grouped by offset.
  - **collapse / expand** of folders and whole-disk trees.
  - **live download**: a per-row / per-file **Get** button that works *while scanning*
    (downloads keys found so far), plus **Download all (.zip)**.
  - a **resizable** disk list.
- **Shared server mode** for an intranet: colleagues open a browser and drag their
  images; the server carves, **auto-deletes each uploaded image the moment its scan
  finishes**, and isolates each user's view (no login required).
- **Parallel carving**: multiple images run in parallel, and a single large drive is
  split across **all CPU cores** (near-linear speed-up), verified to return the exact
  same keys as a single-core pass (boundary keys and GUIDs included).
- **Self-contained executable** (no Python needed on the target) via PyInstaller,
  with **Nuitka** (native code) or **PyArmor** (obfuscated) builds for code protection.

## Supported input

| Type | Formats |
|------|---------|
| Disk images | `.E01` `.Ex01` `.S01` (EWF), `.dd` `.raw` `.img` `.001`, `.vmdk` `.vhdx` |
| Physical disks (Windows) | `\\.\PhysicalDrive0`, `\\.\PhysicalDrive1`, ... |
| Volumes (Windows) | `\\.\C:`, `\\.\D:`, ... |
| Block devices (Linux) | `/dev/sda`, `/dev/nvme0n1`, ... |

---

## Quick start

**Windows (build once, then run):**

1. Unzip the project into a folder.
2. Double-click **`build.bat`** -> produces a single `BitLockerKeyFinder_CLI.exe`.
3. To scan a disk, run **`Local Scanning_GUI.bat`** (raw disk access needs
   Administrator), open the dashboard, pick a disk, click **Scan selected**.
   To scan an image file, just drag it onto `BitLockerKeyFinder_CLI.exe`.

**Linux (run from source):**

```bash
python3 -m pip install --break-system-packages dissect.target dissect.fve dissect.evidence
python3 blkeyfinder.py --gui                 # local dashboard
python3 blkeyfinder.py /path/to/image.E01    # console
sudo python3 blkeyfinder.py /dev/sdb         # a physical disk (needs root)
```

---

## Usage

### 1. Console (CLI)

```
BitLockerKeyFinder_CLI.exe [images/folders/devices ...] [options]
```

Or drag images onto `BitLockerKeyFinder_CLI.exe` directly.

- Drag one or more images onto the `.exe`, or pass paths as arguments, or a folder.
- A **single** image/disk is carved across **all CPU cores** (= logical processors)
  automatically; set a specific number with `--workers N`.
- **Multiple** images run in parallel; a merged `all_recovery_keys.txt` is also
  written so a key found on one drive can be tried against the other evidence.

Examples:

```bat
:: one image — all cores
BitLockerKeyFinder_CLI.exe E:\evidence\disk.E01

:: a whole folder of images, in parallel
BitLockerKeyFinder_CLI.exe D:\evidence\

:: a physical disk (run the console as Administrator)
BitLockerKeyFinder_CLI.exe \\.\PhysicalDrive2

:: choose the number of parallel workers
BitLockerKeyFinder_CLI.exe --workers 16 E:\evidence\disk.E01
```

Keys are printed live as they are found, and written next to the image (see
[Output files](#output-files)).

### 2. Local dashboard (disk picker + provenance tree)

```
BitLockerKeyFinder_CLI.exe --gui [--workers N] [--out-dir PATH]
```

Launchers:
- **`Local Scanning_GUI.bat`** — opens the dashboard and self-elevates (Administrator)
  so you can scan **physical disks / volumes** from the disk picker.
- **`BitLockerKeyFinder_GUI_DragNDrop.bat`** — opens the same dashboard without
  Administrator, for **dragging & dropping image files**.

A browser opens to `http://127.0.0.1:...` (loopback only).

- **Disks on this machine** — physical disks, USB drives and volumes are listed with
  size, filesystem and a BitLocker tag. Select one or more (or **All physical
  disks**) and click **Scan selected (no copy)** — they are carved in place.
  - A **physical disk** carves its whole allocated + unallocated area (deleted keys
    too); a **volume** scans only that volume.
- **Drag & drop** image files or folders onto the page, or **paste a path** (opened
  off disk with no copy — fastest for huge images).
- **Results tree** — each disk/image is a parent row; underneath, keys are grouped
  by the **folder and file** they were found in (hidden files tagged), with a
  separate **Deleted / unallocated** group for carved fragments. Collapse/expand any
  folder or the whole tree.
- **Download** — the per-row and per-file **Get** buttons produce a `.txt` of those
  keys (works while scanning), or **Download all (.zip)** bundles everything.
- Bind address is `127.0.0.1` only (local).

### 3. Shared server (intranet)

Host the dashboard on a server so colleagues on the LAN open it in a browser and
drag their own images onto it. The **server** does the carving.

```
BitLockerKeyFinder_CLI.exe --gui --server --port 80 --workers 32 --out-dir D:\BLKF_results
```

- Prints a colleague link `http://<server-ip>/?t=<token>` and an admin link
  `http://<server-ip>/?t=<token>&admin=<token>` (use `--no-token` for a clean URL on
  a trusted, physically-secured intranet).
- Each uploaded image is streamed to the server, scanned, and the **uploaded copy is
  deleted the instant its scan finishes** — only the recovery-key files remain.
- **Per-user isolation without login**: each browser sees only its own uploads; the
  admin link sees everyone.
- Colleagues cannot enumerate the server's own disks (a browser can't read a remote
  machine's raw disks); the disk picker is shown only via the admin link.

See **[DEPLOY.md](DEPLOY.md)** for the full server + DNS + systemd setup.

---

## Output files

Written into the results folder (or next to the image):

```
<name>_bitlocker_keys.txt     readable table: KEY + Recovery Key ID + SOURCE + LOCATION
<name>_bitlocker_keys.json    same data as JSON, plus the disk -> folder -> file tree
all_recovery_keys.txt         merged unique keys across several images (when >1)
```

The TXT is an aligned table showing, per key: the 48-digit **KEY**, its **Recovery
Key ID** (GUID), the **SOURCE** (`LIVE` / `HIDDEN` / `DELETED`), and the **LOCATION**
(file path, or byte offset for deleted data). Match the Recovery Key ID to the locked
volume, then enter the key.

---

## How it works

1. **Carve the raw media.** The whole device/image is scanned byte-by-byte with a
   regex for the 48-digit pattern, in both ASCII and UTF-16LE. A cheap `'-'` memchr
   skip jumps over hyphen-free regions (most of a typical disk) without running the
   regex. Every match is validated by the divisibility-by-11 checksum, and the
   nearby **Recovery Key ID (GUID)** is captured. This finds keys regardless of
   filesystem, deletion, or hidden status — if the bytes are on the disk, they are
   found.
2. **Attribute each key to a file (provenance).** The filesystem is then walked and
   each live file (including hidden) is carved, so keys that live in a current file
   get their exact **folder + filename**. Keys that exist only in unallocated space
   (deleted, no file entry) are reported with their **byte offset**.
3. **Match to volumes.** BitLocker (FVE) metadata on the media is parsed to list each
   encrypted volume's Recovery Key ID, so a found key can be matched to the volume it
   unlocks.

Single large drives are split into byte ranges carved in parallel across all cores,
with overlapping range boundaries so a key sitting on a boundary is still recovered
whole.

---

## Administrator / root rights

Reading a **physical disk** or a **volume** raw is an OS-protected operation:

- **Windows:** needs **Administrator**. Launch **`Local Scanning_GUI.bat`** (it
  self-elevates via UAC), or run the console as Administrator. Without elevation you
  can still scan **image files** (no admin needed); physical-disk reads are refused
  with a clear message.
- **Linux:** needs **root** (or membership of the `disk` group).

Listing the disks (names/sizes) does not need admin; only the raw read does.

---

## Performance & tuning

- **Workers (`--workers N`)** default to the CPU core count. Carving is CPU-bound on
  data-dense regions and disk-bound on empty ones.
- **SSD / NVMe:** more workers = faster (near-linear).
- **Single spinning HDD:** many parallel readers cause head thrash and can be
  *slower*; try a lower worker count (e.g. `--workers 4-8`) for one HDD.
- **Provenance** (the folder/file attribution pass) reads allocated file data, so it
  adds time on large, full volumes. It is best-effort and falls back to offsets when
  a filesystem can't be parsed (e.g. a raw physical disk dissect can't open).
- **Rough throughput:** carving is bounded by `min(disk read speed, CPU carve
  speed)`. A 100 TB estimate is dominated by disk read bandwidth — plan by your
  array's MB/s. Measure on a 1 TB sample and extrapolate for your hardware.

---

## Building the executable

All build scripts produce a single self-contained `BitLockerKeyFinder_CLI.exe` (Python,
`page.html`, and the `dissect` stack are bundled inside). Run each from **inside the
project folder**.

| Script | Tool | When to use |
|--------|------|-------------|
| `build.bat` | PyInstaller (`--onefile`) | Default. Fast build. On Python 3.12 the bytecode is already hard to decompile. |
| `build_hardened.bat` | Nuitka (native C) | Strongest anti-reversing (no decompilable bytecode). Needs a C compiler (auto-downloaded). |
| `build_obf.bat` | PyArmor + PyInstaller | Obfuscated single exe without a C compiler (when Nuitka won't build). |

Requirements: 64-bit Windows, Python 3.10-3.13 (3.12 recommended).

On Linux you don't build an exe — run `python3 blkeyfinder.py ...` from source, or
use `build_server_obf.sh` for an obfuscated server deployment.

## Protecting the code (obfuscation & distribution)

To hand the tool to colleagues **without shipping the source**:

- Build with `build_hardened.bat` (Nuitka) or `build_obf.bat` (PyArmor).
- Distribute **only** `BitLockerKeyFinder_CLI.exe` plus the `.bat` launchers you use.
- You may delete everything else — `blfinder/`, `blkeyfinder.py`, `page.html`,
  `build*.bat`, `tests/`, `dist/`, `.pyarmor/`, `*.spec` — none are needed at runtime.
  **Keep a backup of the source elsewhere** so you can rebuild later.

> No client-side protection is absolute. For zero code on colleague machines, use the
> **shared server** model instead (the exe stays on the server; colleagues only use a
> browser).

## Deployment

See **[DEPLOY.md](DEPLOY.md)** for hosting the shared dashboard on a Linux (Proxmox
VM/LXC) or Windows VM: static IP, port 80, a clean DNS name, a systemd/Task-Scheduler
service for auto-start, firewall, and the obfuscated build.

## Limitations

- The tool finds recovery keys **that were saved on the media**. If no copy of the
  key is on the disk, there is nothing to find — it is not a cryptographic attack.
- **Deleted keys** are recovered only while their bytes remain in unallocated space
  (not yet overwritten). Deleted data has no file entry, so only the byte offset is
  shown (not a folder/filename).
- On a **raw physical disk** the folder/file provenance may be unavailable (keys are
  still found, shown under Deleted/offset); use a **volume** or an image file for full
  provenance.
- **HTTP only** for the shared server (fine for a trusted internal LAN).

## Legal

This software is intended for **authorized digital forensic examination** of media you
own or have explicit permission to analyze. You are responsible for complying with all
applicable laws and policies. It recovers only keys that were already saved on the
media and performs no cryptographic attack on BitLocker.
