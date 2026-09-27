# TZVerify — International Edition

Timezone verification for forensic images and extracted file systems.

TZVerify reads the configured timezone of a device, compares it with your
reference ("home") timezone and, if they differ, gives a reasoned verdict with a
confidence score and the evidence used. This edition has no fixed home zone —
set the one that fits your jurisdiction.

## Supported platforms
- iOS
- Android
- Windows
- macOS

## Supported inputs

| Input | Supported |
|-------|-----------|
| UFDR report package | Yes |
| File-system archive (.zip) | Yes — for very large archives, extract to a folder first |
| E01 / Ex01 | Yes |
| Raw image (.img, .dd, .raw, .001) | Yes |
| Extracted file-system folder | Yes — fastest |
| Single .plist / .db file | No |

When using a folder, drop the folder that directly contains `private/` (iOS) or
`data/` and `system/` (Android).

## Usage — Windows

Download `TZVerify.exe` from [Releases](../../releases). No installation needed.

# **Drag & drop:** drag an image or folder onto `TZVerify.exe`.

**Command line (cmd):**
```
TZVerify.exe <image_or_folder>
TZVerify.exe <image_or_folder> --home America/New_York
```
**PowerShell:** prefix with `.\`
```
.\TZVerify.exe <image_or_folder> --home Asia/Seoul
```

## Usage — Linux / macOS
The current release is a Windows binary. Native Linux/macOS builds are planned.
Until then, run it on a Windows analysis workstation.

## Options

| Option | Description |
|--------|-------------|
| `--home ZONE` | Reference timezone (IANA name, e.g. `Europe/London`) |
| `--fast` | Timezone artifacts only, no full scan (default for E01/IMG) |
| `--full` | Also scans photos/carrier data for richer reasoning (default for folders/UFDR) |
| `--out DIR` | Output folder for reports |

## Setting the home zone permanently
Place a file `tzverify.ini` next to `TZVerify.exe`:
```
home = America/New_York
```
Order of precedence: `--home` > `tzverify.ini` > `TZVERIFY_HOME` env variable > system timezone > `Europe/Berlin`.

## Output
Two reports are written next to the tool: `TZVerify_<name>_<time>.txt` and `.html`.
All times are shown in the home zone (e.g. `2026-09-27 13:04:01 EDT (UTC-04:00)`).

## Verdicts

| Verdict | Meaning |
|---------|---------|
| `MATCH_HOME` | Device timezone equals the home zone |
| `GENUINE_FOREIGN` | Evidence indicates real use in the other region |
| `STALE_SETTING` | Device likely dormant; old setting retained |
| `MANUAL_OVERRIDE` | Automatic timezone off; value likely set manually |
| `INCONCLUSIVE` | Not enough evidence — the report lists what to check |

## Limitations
- In FAST mode without acquisition metadata, the analysis time is used as the acquisition time.
- Location hints use coarse bounding boxes.

## Disclaimer
Triage aid only. Verify all conclusions against primary artifacts. Provided as-is,
for lawful use by authorized examiners.
