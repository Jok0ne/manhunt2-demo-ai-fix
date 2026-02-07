# Evidence: Manhunt 2 Shipped With Demo AI Values

## Cross-Platform Verification Report

**Date**: 2026-02-07
**Analyst**: Jok0ne aka Zerone
**Method**: Direct extraction from retail ISOs + NSIS installer decompression

---

## 1. Source ISOs

### PSP (Europe) — ULES-00756
- **File**: `Manhunt 2 (Europe) (En,Fr,De,Es,It).iso`
- **ISO SHA256**: `d16de4ae70512dd382c2afa3505d663a4abe5b846a8d59fc2230a705db102e49`
- **ISO Size**: 1,305,051,136 bytes (1.22 GB)
- **ISO Metadata**:
  - System: PSP GAME
  - Volume: MANHUNT2
  - Publisher: ROCKSTAR
  - Preparer: ROCKSTAR
  - Created: 2007-09-05 17:04:44
- **Config Path**: `PSP_GAME/USRDIR/GLOBAL/INIS/AITYPED.INI`
- **Config Format**: Plain text INI (directly accessible)

### PS2 (Europe)
- **File**: `Manhunt 2 (Europe) (En,Fr,De,Es,It).iso`
- **ISO SHA256**: `94c6b1b5b0d0aa3f497e551ee2258d6493d2427ddefa32c7064eb2c6327472e1`
- **ISO Size**: 2,658,795,520 bytes (2.48 GB)
- **ISO Metadata**:
  - System: PLAYSTATION
  - Volume: MANHUNT2
  - Publisher: ROCKSTAR GAMES
  - Preparer: ROCKSTAR LONDON
  - Created: 2007-09-05 08:06:02
- **Config Path**: `GLOBAL/INIS/AITYPED.INI`
- **Config Format**: Plain text INI (directly accessible)

### PC (RELOADED release)
- **File**: `rld-mh2.iso` (inside `Manhunt-2_Win_EN_ISO-Version.zip`)
- **ISO Created**: 2009-11-08 16:07:35 (repackage date)
- **Installer**: NSIS-2 (`setup.exe`, 171 MB)
- **Config Path**: `global/ini/resource6.glg`
- **Config Format**: Z2HM container with zlib compression (decompresses to identical plain text)
- **Decompression Method**: Skip 8-byte header (`Z2HM` + 4 bytes), decompress remainder with zlib

---

## 2. AITYPED.INI Hash Comparison

| Platform | Extraction Method | SHA256 | Size |
|----------|------------------|--------|------|
| PSP (EU) | 7z extract from UMD ISO | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |
| PS2 (EU) | 7z extract from UDF ISO | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |
| PC (EN)  | zlib decompress from .glg | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |

**Result: Byte-for-byte identical across all three platforms.**

---

## 3. Per-Level Copies (PSP)

The PSP version contains 16 per-level copies of AITYPED.INI in addition to the global one.
All copies are byte-for-byte identical (verified via SHA256):

```
PSP_GAME/USRDIR/GLOBAL/INIS/AITYPED.INI          -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A01_ES/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A02_TH/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A03_NE/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A04_SM/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A06_CI/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A07_TO/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A07_2T/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A09_BU/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A10_BR/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A11_ME/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A12_PL/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A14_SU/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A15_CE/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A16_TV/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A17_CR/AITYPED.INI        -> aee092ad...
PSP_GAME/USRDIR/LEVELS/A18_MA/AITYPED.INI        -> aee092ad...
```

The PS2 version has only the global copy. The PC version packs configs per-level in separate `.glg` containers.

---

## 4. Affected Code Sections

### Block 1: Combat and Chase Persistence (Lines 148-153)

Verbatim from the extracted file (identical across all platforms):

```
RECORD default
	ALERT_LEVEL_DEGRADE	20

	SET_ALERT_LEVEL		combat		very_high
	SET_ALERT_LEVEL		chase		high
	SET_ALERT_LEVEL		investigate	low

	SUSTAIN_ACTION		combat		1
	SUSTAIN_ACTION		chase		3
	SUSTAIN_ACTION		investigate	8

#################################
#SETTINGS FOR DEMO VERSION!!!!! #
#################################
```

- `SUSTAIN_ACTION combat 1` — Hunters fight for **1 second** before disengaging
- `SUSTAIN_ACTION chase 3` — Hunters chase for **3 seconds** before giving up
- `investigate 8` appears to be a normal value

### Block 2: Search Parameters (Lines 191-196)

```
RECORD search_parameters
#################################
#SETTINGS FOR DEMO VERSION!!!!! #
#################################
	RADIUS_m		35
	...
	DENSITY_%		85
	...
	DOUBLE_SEARCH_%		15
```

---

## 5. PC Config Architecture Note

The PC version uses a proprietary packaging format for config files:

- **Container format**: `.glg` files with `Z2HM` 4-byte magic header
- **Compression**: zlib (raw deflate), starts at byte offset 8
- **Location**: `global/ini/resource6.glg` contains AITYPED.INI
- **Discovery method**: Decompressed all 10 .glg files in `global/ini/`, searched for "DEMO VERSION" string
- The EXE references the filename `AITYPED.INI` as a string, confirming runtime loading

This means PC modders cannot simply edit a text file — they need to decompress the .glg, modify, recompress, and repack. This may explain why the demo values were never discovered on PC.

---

## 6. Reproduction Steps

### PSP / PS2 (easiest)
```bash
# Extract with 7z (handles both UMD and UDF formats)
7z e "Manhunt 2.iso" "GLOBAL/INIS/AITYPED.INI" -o/tmp/
# Open in any text editor, search for "DEMO VERSION"
grep -n "DEMO VERSION" /tmp/AITYPED.INI
# Verify: lines ~152 and ~192
```

### PC
```bash
# 1. Extract setup.exe from ISO
7z e rld-mh2.iso setup.exe
# 2. Extract .glg files from NSIS installer
7z e setup.exe "global/ini/*" -oglobal_ini/
# 3. Decompress resource6.glg (Python)
python3 -c "
import zlib
with open('global_ini/resource6.glg', 'rb') as f:
    data = f.read()
with open('AITYPED_PC.INI', 'wb') as f:
    f.write(zlib.decompress(data[8:]))
"
# 4. Search for DEMO VERSION
grep -n "DEMO VERSION" AITYPED_PC.INI
```

---

## 7. Timeline Context

Both ISOs were mastered on the same day:
- PS2: **2007-09-05 08:06:02** (morning)
- PSP: **2007-09-05 17:04:44** (afternoon)

This is consistent with a final gold master build. The demo values were present in the source tree at the time of mastering.

The ESRB AO-to-M rating crisis occurred June-September 2007, with the final submission happening weeks before the September 5 master date. The development team was focused on content censorship (VHS blur effects, execution cuts) rather than gameplay tuning.

---

## Appendix: File Identification Table

| File | Description | Size (bytes) | Content |
|------|-------------|------|---------|
| `AITYPED.INI` | AI Type Data config | 13,627 | All AI behavior: perception, combat, search |
| `resource6.glg` | PC zlib container | 4,212 | Compressed AITYPED.INI (decompresses to 13,627) |

---

*Compiled 2026-02-07. All hashes computed with SHA-256 via `shasum -a 256`.*
