# Evidence: Manhunt 1 & 2 Shipped With Demo AI Values

## Cross-Platform and Cross-Game Verification Report

**Date**: 2026-02-07 (MH2), updated 2026-02-21 (MH1 verification)
**Analyst**: Jok0ne aka Zerone
**Method**: Direct extraction from retail ISOs, NSIS installer decompression, and MHPK PAK reverse-engineering

---

## 1. Source Files

### Manhunt 1 PC (Razor1911 scene release)
- **Source**: `archive.org/details/manhunt-razor-1911_202602`
- **Archive**: `ManHunt.pak` (MHPK format, 82 files, XOR 0x7F encrypted)
- **Config Path**: `levels/global/data/aiTypeData.ini` (inside PAK)
- **Config Format**: XOR 0x7F encrypted plain text INI
- **Extraction**: MHPK header parsing (276-byte entries) + XOR 0x7F decryption on all data bytes

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

## 2. AI Config Hash Comparison

| Game | Platform | Extraction Method | SHA256 | Size |
|------|----------|------------------|--------|------|
| **MH1** | PC | XOR 0x7F decrypt from MHPK `.pak` | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |
| **MH2** | PSP (EU) | 7z extract from UMD ISO | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |
| **MH2** | PS2 (EU) | 7z extract from UDF ISO | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |
| **MH2** | PC (EN) | zlib decompress from .glg | `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` | 13,627 bytes |

**Result: Byte-for-byte identical across Manhunt 1 (2003) and all three Manhunt 2 (2007) platforms.**

Rockstar copied `aiTypeData.ini` from Manhunt 1 into Manhunt 2 without modifying a single byte. The file was then distributed unchanged across PSP, PS2, and PC.

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

> **Note (2026-02-21):** Binary analysis confirmed that `SUSTAIN_ACTION`, `SET_ALERT_LEVEL`, and `ALERT_LEVEL_DEGRADE` are **not parsed** by either the MH1 or MH2 engine. These strings do not exist in either executable. The values above are dead data from an unfinished data-driven AI system. Combat and chase durations are hardcoded in the game executables.

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

## 6. Manhunt 1 PAK Format (MHPK)

Manhunt 1 PC stores configs inside `ManHunt.pak` with XOR encryption:

```
Header (12 bytes):
  4 bytes: "MHPK" magic
  4 bytes: Version (0x00020000 = v2.0, little-endian)
  4 bytes: Entry count (82 files)

Entry (276 bytes each):
  256 bytes: Filename (null-padded ASCII)
  4 bytes:   Padding
  4 bytes:   File size
  4 bytes:   Absolute offset into PAK
  4 bytes:   Flags
  4 bytes:   Hash

Data encryption: Every byte XOR'd with 0x7F
  '#' (0x23) stored as '\' (0x5C)
  '\n' (0x0A) stored as 'u' (0x75)
  '\r' (0x0D) stored as 'r' (0x72)
  '\t' (0x09) stored as 'v' (0x76)
  ' ' (0x20) stored as '_' (0x5F)
```

---

## 7. Reproduction Steps

### Manhunt 1 PC
```bash
# Extract ManHunt.pak from installer/disc, then:
python3 -c "
import struct
with open('ManHunt.pak','rb') as f: data=f.read()
n=struct.unpack_from('<I',data,8)[0]
for i in range(n):
    b=12+(i*276)
    name=data[b:b+256].split(b'\x00')[0].decode()
    sz=struct.unpack_from('<I',data,b+260)[0]
    off=struct.unpack_from('<I',data,b+264)[0]
    if 'aiTypeData' in name:
        dec=bytes(x^0x7F for x in data[off:off+sz])
        open('aiTypeData_MH1.ini','wb').write(dec)
        print(f'Extracted: {name} ({sz} bytes)')
"
grep -n 'DEMO VERSION' aiTypeData_MH1.ini
shasum -a 256 aiTypeData_MH1.ini
# Expected: aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83
```

### PSP / PS2 (easiest)
```bash
# Extract with 7z (handles both UMD and UDF formats)
7z e "Manhunt 2.iso" "GLOBAL/INIS/AITYPED.INI" -o/tmp/
# Open in any text editor, search for "DEMO VERSION"
grep -n "DEMO VERSION" /tmp/AITYPED.INI
# Verify: lines ~152 and ~192
```

### MH2 PC
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

## 8. Timeline Context

Both ISOs were mastered on the same day:
- PS2: **2007-09-05 08:06:02** (morning)
- PSP: **2007-09-05 17:04:44** (afternoon)

This is consistent with a final gold master build. The demo values were present in the source tree at the time of mastering.

The ESRB AO-to-M rating crisis occurred June-September 2007, with the final submission happening weeks before the September 5 master date. The development team was focused on content censorship (VHS blur effects, execution cuts) rather than gameplay tuning.

---

## Appendix A: File Identification Table

| File | Description | Size (bytes) | Content |
|------|-------------|------|---------|
| `AITYPED.INI` | AI Type Data config | 13,627 | All AI behavior: perception, combat, search |
| `resource6.glg` | PC zlib container | 4,212 | Compressed AITYPED.INI (decompresses to 13,627) |

---

## Appendix B: Manhunt 1 PAK Contents (82 files)

The MHPK archive contains 9 global config files and 24 levels × 3 files each:

**Global configs**: `aiTypeData.ini`, `WeaponTypeData.ini`, `physicsTypeData.ini`, `ShotTypeData.ini`, `AISounds.ini`, `Communications.ini`, `EntityStateSounds.ini`, `ParticleEffects.INI`, `WEATHER.INI`

**Per-level (24 levels)**: `entityTypeData.ini` + `levelSetup.ini` + `splines.ini` each

---

*Compiled 2026-02-07 (MH2), updated 2026-02-21 (MH1 verification). All hashes computed with SHA-256 via `shasum -a 256`.*
