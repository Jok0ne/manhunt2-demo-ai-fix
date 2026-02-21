# Manhunt 2 Shipped With Demo AI on All Platforms

## Summary

Both Manhunt 1 (2003) and Manhunt 2 (2007) shipped with placeholder AI values intended for demo builds. The Manhunt 1 community has known about these demo values for years. What this project documents is that **Manhunt 2 uses the exact same config file** — byte-for-byte identical — inherited unchanged from MH1, and provides a cross-platform fix.

The demo values result in hunters that:
- **Search a limited area** with reduced thoroughness (35m radius, 85% density, 15% double-check)
- **Hear running as quieter than walking** — acoustic inversion bug in `VOLUME_DISTANCES`

The config also contains `SUSTAIN_ACTION combat 1` and `SUSTAIN_ACTION chase 3`, but binary analysis of both executables confirmed these fields are **never parsed by the engine** — they are dead data from an unfinished system. See the "Engine Analysis" section below.

---

## Evidence

### File: `AITYPED.INI` (AI Type Data)

This file controls all hunter AI behavior: perception, aggression, search patterns, hearing, vision, and combat persistence. It exists as a global config and as per-level copies (16 levels on PSP).

**Locations (verified):**
- PSP: `PSP_GAME/USRDIR/GLOBAL/INIS/AITYPED.INI` + 16 per-level copies
- PS2: `GLOBAL/INIS/AITYPED.INI` (global only, no per-level copies)
- PC: `global/ini/resource6.glg` (zlib-compressed inside NSIS installer)

### The Developer Comments

The file contains two blocks of developer comments, each with **five exclamation marks**, at lines 152-154 and 192-194:

```ini
#################################
#SETTINGS FOR DEMO VERSION!!!!! #
#################################
```

### Demo Values — Block 1: Combat Persistence (Line 148-150)

Located in `RECORD default`, directly above the demo warning:

```ini
RECORD default
    ALERT_LEVEL_DEGRADE     20          # seconds until hunter forgets alert
    SUSTAIN_ACTION          combat      1       # <-- DEMO: 1 SECOND of combat
    SUSTAIN_ACTION          chase       3       # <-- DEMO: 3 SECONDS of chase
    SUSTAIN_ACTION          investigate 8

#################################
#SETTINGS FOR DEMO VERSION!!!!! #
#################################
```

**What the developers intended:**
- `SUSTAIN_ACTION combat 1` — combat duration of 1 second
- `SUSTAIN_ACTION chase 3` — chase duration of 3 seconds

**What actually happens:** Binary analysis confirmed that the engine **does not read** `SUSTAIN_ACTION`, `SET_ALERT_LEVEL`, `ALERT_LEVEL_DEGRADE`, or `ACTION_TRANSITION`. These strings do not exist in either the Manhunt 1 or Manhunt 2 executable. The Manhunt 1 EXE contains debug messages like `TEMP_SetDefaultTransitions() is a temporary function to hardcode transition information` — Rockstar planned a data-driven AI system but never finished it. Combat and chase durations are hardcoded in the game executable.

### Demo Values — Block 2: Search Parameters (Line 191-210)

```ini
RECORD search_parameters
#################################
#SETTINGS FOR DEMO VERSION!!!!! #
#################################
    RADIUS_m            35          # search radius in meters
    DENSITY_%           85          # % of searchable objects checked
    DOUBLE_SEARCH_%     15          # % chance of rechecking same spot
```

These values result in less thorough searching, making it easier for demo players to avoid detection.

### Cross-Platform and Cross-Game Verification

The AI config file is **byte-for-byte identical** across Manhunt 1 and all verified Manhunt 2 platforms:

| Game | Platform | Version | SHA256 | Config Format |
|------|----------|---------|--------|---------------|
| **Manhunt 1** | PC | Razor1911 / PCDVD | `aee092ad...3ed2b83` | XOR 0x7F encrypted in MHPK `.pak` |
| **Manhunt 2** | PSP | ULES-00756 (EU) | `aee092ad...3ed2b83` | Plain text INI |
| **Manhunt 2** | PS2 | PAL (EU) | `aee092ad...3ed2b83` | Plain text INI |
| **Manhunt 2** | PC | EN (RELOADED) | `aee092ad...3ed2b83` | zlib-compressed in `.glg` container |

Full SHA256: `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83` (13,627 bytes)

**This means Rockstar copied `aiTypeData.ini` from Manhunt 1 (2003) to Manhunt 2 (2007) without any modification whatsoever** — not even a single byte was changed across 4 years of development.

Both MH2 console ISOs were mastered on the same day: **PS2 at 08:06, PSP at 17:04 on September 5, 2007.**

The Wii version is expected to be identical but was not verified in this analysis.

---

## Origin: Inherited from Manhunt 1

The demo AI values are **not** a Manhunt 2-specific mistake. They originate from Manhunt 1 (2003), where they have been known to the community for some time. Rockstar copied the entire `aiTypeData.ini` (renamed `AITYPED.INI`) unchanged into Manhunt 2's codebase.

The Manhunt 1 config was extracted from `ManHunt.pak`, which uses Rockstar's MHPK archive format with XOR 0x7F encryption on all file data. After decryption, the file is byte-for-byte identical to every Manhunt 2 version.

### Why Manhunt 2 Never Got Fixed Values

#### The ESRB Crisis Timeline (2007)

1. **June 2007** — Manhunt 2 receives AO (Adults Only) rating from ESRB
2. **June 2007** — Sony and Nintendo refuse to license AO games; release cancelled
3. **July-September 2007** — Rockstar scrambles to censor content (VHS blur effects, cut executions)
4. **October 2007** — Resubmitted, receives M rating, rushed to release

The inherited demo config was never updated during this crunch. The development team was focused on censoring violent content to satisfy the ESRB. The AI configuration was likely deprioritized or simply assumed to be correct since it was carried over from the previous game.

---

## Impact on Gameplay

The demo AI values explain a long-standing community observation: **Manhunt 2's hunters feel passive and "stupid" compared to Manhunt 1.**

With `combat 1`:
- Melee fights feel disconnected — enemies swing once and back off
- Group encounters lack sustained pressure
- The combat system's depth (combos, environmental kills, weapon variety) is underutilized

With `chase 3`:
- Stealth failures have minimal consequences — just run for 3 seconds
- The tension of being hunted is almost nonexistent
- The game's stealth mechanics become optional rather than necessary

### Community Evidence

Multiple reviews and player discussions from 2007-2025 note that Manhunt 2's AI feels "dumbed down" compared to the original:

> "The hunters in MH2 are braindead compared to MH1" — common sentiment on r/manhunt and GTA Forums

The [Manhunt 2 Improved AI mod](https://www.moddb.com/mods/manhunt-2-improved-ai) on ModDB adjusts vision and hearing ranges — fields the engine does read. That mod's approach was on the right track for INI-level changes, though it did not address the search parameters or acoustic inversion bug.

---

## Engine Analysis

Binary analysis of both Manhunt 1 (`MANHUNT.EXE`, 6.3 MB) and Manhunt 2 (`Manhunt2.exe`, 3.1 MB) PC executables confirmed which `AITYPED.INI` sections the engine actually reads:

### Fields the engine reads (INI-fixable)

| Section | What it controls |
|---------|-----------------|
| `VOLUME_DISTANCES` | Hearing distances per sound level (float values) |
| `BSP_VOLUME_FILTERING` | Sound dampening through walls |
| `INTERESTING_SOUNDS` | Which sounds trigger AI interest |
| `search_parameters` | `RADIUS_m`, `DENSITY_%`, `DOUBLE_SEARCH_%` |

### Fields the engine ignores (dead data)

| Field | Evidence |
|-------|----------|
| `SUSTAIN_ACTION` | String not present in either executable |
| `ALERT_LEVEL_DEGRADE` | String not present in either executable |
| `SET_ALERT_LEVEL` | String not present in either executable |
| `ACTION_TRANSITION` | String not present in either executable |

Manhunt 1's executable contains debug messages confirming the developers intended to replace hardcoded AI transitions with config-driven logic: `TEMP_SetDefaultTransitions() is a temporary function to hardcode transition information`. This replacement was never completed in either game.

---

## Recommended Fix

Based on engine analysis, only changes to fields the engine actually reads have an effect:

```ini
RECORD search_parameters
    RADIUS_m            40          # was 35 — wider search area
    DENSITY_%           90          # was 85 — more thorough searching
    DOUBLE_SEARCH_%     25          # was 15 — more re-checking

RECORD VOLUME_DISTANCES
    # Fix acoustic inversion: Running must be louder than Walking
    # Original has Running (0.37) quieter than Walking (0.40)
```

Fixing combat duration, chase persistence, and alert degradation would require **patching the game executable** where these values are hardcoded.

These values must be changed in the global config **and** all 16 per-level copies (PSP) or the equivalent per-level files on other platforms.

---

## Verification

Anyone can verify this discovery in under 2 minutes:

### PSP or PS2 (easiest)
```bash
7z e "Manhunt 2.iso" "GLOBAL/INIS/AITYPED.INI" -o/tmp/
grep -n "DEMO VERSION" /tmp/AITYPED.INI
# Expected output: lines ~152 and ~192
shasum -a 256 /tmp/AITYPED.INI
# Expected: aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83
```

### PC
```bash
# Extract NSIS installer from ISO, then decompress .glg container
7z e rld-mh2.iso setup.exe && 7z e setup.exe "global/ini/resource6.glg"
python3 -c "
import zlib
with open('resource6.glg','rb') as f: d=f.read()
with open('AITYPED.INI','wb') as f: f.write(zlib.decompress(d[8:]))
"
grep -n "DEMO VERSION" AITYPED.INI
```

The file is plain text (or trivially decompressible), unencrypted, and has been accessible since 2007.

---

## Discovery & Analysis Details

- **Analysis by:** Jok0ne aka Zerone, February 2026
- **Method:** Systematic analysis of 287+ config files using a custom SQLite FTS5 index (146,586 entries) + ChromaDB semantic search (11,019 vectors). Manhunt 1 PAK extraction via MHPK reverse-engineering (XOR 0x7F decryption). Fix values informed by GTA III/VC/SA reversed source code analysis.
- **MH2 platforms verified:** PSP (ULES-00756), PS2 (PAL), PC (EN) — all byte-identical
- **MH1 verification:** PC (Razor1911 scene release) — byte-identical to all MH2 versions
- **Prior knowledge:** The Manhunt 1 community has known about the demo AI values. This project adds: (1) SHA256 proof that MH1 and MH2 share the exact same file, (2) cross-platform MH2 verification, (3) binary analysis proving which fields work, (4) a targeted fix for the fields the engine actually reads.
- **Evidence file:** See [VERIFICATION.md](VERIFICATION.md) for full SHA256 hashes, ISO metadata, and reproduction steps

---

## License

This document is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You are free to share and adapt this information with attribution.
