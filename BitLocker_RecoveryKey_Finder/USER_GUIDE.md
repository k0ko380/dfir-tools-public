# BitLocker Recovery Key Finder — User Guide

Recovers BitLocker 48-digit recovery keys that were **saved** on a disk — from disk
images and from disks directly, including deleted and hidden copies. It does not
crack anything; it only finds keys that were already saved.

> For authorized forensic examination only. No install, no Python needed.

## What's in the package

| File | What it is |
|------|-----------|
| **`BitLockerKeyFinder_CLI.exe`** | The tool itself (console mode). It is also the engine every launcher below uses — keep it in the same folder as the `.bat` files. |
| **`BitLockerKeyFinder_GUI_DragNDrop.bat`** | **GUI** — opens the dashboard in your browser to **drag & drop image files**. No Administrator needed. |
| **`Local Scanning_GUI.bat`** | **GUI** — opens the dashboard to **scan this computer's own disks** (disk picker). Needs Administrator (self-elevates). |
| **`Start Server.bat`** | **Shared server** for a team — colleagues upload images from their own PCs via a browser. Run on one server machine only. |

## How to use

### A. GUI — drag & drop an image file  →  `BitLockerKeyFinder_GUI_DragNDrop.bat`
Double-click it; your browser opens the dashboard (local only). Then:
- **Drag & drop** one or more images (`.E01` / `.dd` / `.raw` / `.img` / `.vmdk` /
  `.vhdx`) or folders onto the page, or click **Choose files**.
- Keys appear live in a tree: **disk → folder → file → key** (deleted keys grouped
  by byte offset). Expand/collapse folders as you like.
- Download with each row's / file's **Get** button (works while scanning), or
  **Download all (.zip)**.

No Administrator needed for image files.

### B. GUI — scan this computer's disks  →  `Local Scanning_GUI.bat`
Double-click it and accept the **Administrator (UAC)** prompt — raw disk access needs
it. The same dashboard opens, now with a **"Disks on this machine"** panel at the
top:
- Select a **physical disk** or a **volume** (or **All physical disks**).
- Click **Scan selected (no copy)** — the disk is carved in place.
- A **physical disk** scans its whole allocated **+ unallocated** area (deleted keys
  too); a **volume** scans just that volume.

If you decline the UAC prompt it still opens, but then only image files can be
scanned (physical disks need admin).

### C. Console (CLI)  →  `BitLockerKeyFinder_CLI.exe`
For scripting or a quick scan without a browser:
- **Drag** an image onto the `.exe`, or run it from a terminal:
  `BitLockerKeyFinder_CLI.exe E:\evidence\disk.E01`
- A folder or several images at once also work. To scan a physical disk, run the
  console **as Administrator**: `BitLockerKeyFinder_CLI.exe \\.\PhysicalDrive2`

### D. Shared server (your team sets this up)  →  `Start Server.bat`
An administrator runs `Start Server.bat` on **one** server machine. Everyone else
just opens the printed link in a browser (e.g. `http://<server>:8700/`) and **drags
their disk image onto the page**. The server scans it and deletes the uploaded copy
as soon as it finishes; you see only your own uploads. (Nothing to install on the
colleagues' PCs.)

## Results

Written to the results folder (default `BLKF_results`, shown and changeable in the
dashboard):

```
<name>_bitlocker_keys.txt    readable: KEY + Recovery Key ID + SOURCE + LOCATION
<name>_bitlocker_keys.json   same data as JSON (plus the disk → folder → file tree)
```

Match the **Recovery Key ID** to your locked volume, then enter the 48-digit **KEY**.

## Notes

- SmartScreen may warn on first run: **More info → Run anyway**.
- Scanning a large physical disk takes time; progress and estimated time are shown.
  On a single HDD, fewer parallel workers can be faster — run with `--workers 8`.
- The build date is shown in the dashboard header (top-left, under the title).
- Use this only on media you are authorized to examine.
