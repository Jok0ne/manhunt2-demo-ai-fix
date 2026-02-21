# Manhunt 2 Demo AI Fix

Both Manhunt 1 (2003) and Manhunt 2 (2007) shipped with placeholder AI values intended for demo builds. The community has long known about Manhunt 1's demo AI settings. This repository documents that **Manhunt 2 uses the exact same file** — byte-for-byte identical — and provides a cross-platform fix.

## Key Facts

- Hunters search with **reduced thoroughness** (35m radius, 85% density, 15% double-check rate)
- **Acoustic inversion bug**: Running (0.37) is quieter than Walking (0.40) — the AI is *less* likely to hear you sprint than walk
- The config contains `SUSTAIN_ACTION combat 1` and `chase 3` marked as `#SETTINGS FOR DEMO VERSION!!!!!` — however, binary analysis confirmed the engine **never reads** these fields (see below)
- The file `AITYPED.INI` is **byte-for-byte identical** across Manhunt 1 PC, Manhunt 2 PSP, PS2, and PC
- **SHA256** (all versions): `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83`

## Background

The Manhunt 1 community has known for years that the game shipped with demo AI values (`#SETTINGS FOR DEMO VERSION!!!!!`). What was not documented until now:

1. **Manhunt 2 uses the exact same file** — `AITYPED.INI` is byte-for-byte identical (SHA256 match) between Manhunt 1 (2003) and Manhunt 2 (2007). Rockstar copied the config unchanged across 4 years.
2. **Cross-platform verification** — the file is identical across PSP, PS2, and PC versions of Manhunt 2.
3. **Existing mods miss the full picture** — the [Manhunt 2 Improved AI](https://www.moddb.com/mods/manhunt-2-improved-ai) mod on ModDB adjusts vision and hearing ranges (which the engine does read) but does not address the search parameters or acoustic bug.
4. **PC configs are hidden** — on PC, the config is zlib-compressed inside proprietary `.glg` containers, making it harder to find than on console.
5. **Binary analysis** confirmed which `AITYPED.INI` fields the engine actually reads and which are dead data from an unfinished system.

Manhunt 2 received an AO (Adults Only) rating from the ESRB in June 2007. Sony and Nintendo refused to license AO titles, forcing Rockstar to spend months censoring content (VHS blur effects, cut executions) to secure an M rating. Both console ISOs were mastered on September 5, 2007. During this crunch, the inherited demo AI config was never updated.

## What the Engine Actually Reads

Binary analysis of both Manhunt 1 and Manhunt 2 PC executables confirmed that the engine only parses four sections from `AITYPED.INI`: `VOLUME_DISTANCES`, `BSP_VOLUME_FILTERING`, `INTERESTING_SOUNDS`, and `search_parameters`.

The `SUSTAIN_ACTION`, `SET_ALERT_LEVEL`, `ALERT_LEVEL_DEGRADE`, and `ACTION_TRANSITION` fields are **never parsed** — the strings don't exist in either binary. These are remnants of an unfinished data-driven AI system. Manhunt 1's executable contains debug messages like `TEMP_SetDefaultTransitions() is a temporary function to hardcode transition information` — the developers planned to replace hardcoded transitions with config-driven logic, but it was never completed. Manhunt 2 removed the debug messages but still never added a parser.

This means AI combat duration, chase persistence, and alert degradation are **hardcoded in the executable** and cannot be changed via INI modding alone.

## The Fix

### Changes that work (engine reads these)

| Parameter | Original (Demo) | Fixed | Effect |
|-----------|-----------------|-------|--------|
| `RADIUS_m` | 35 | 40 | Wider search area |
| `DENSITY_%` | 85 | 90 | More thorough searching |
| `DOUBLE_SEARCH_%` | 15 | 25 | More re-checking of spots |

Additionally, `VOLUME_DISTANCES` contains an **acoustic inversion bug**: Running (0.37) is quieter than Walking (0.40). The engine loads these values — running really is quieter than walking in the game's hearing model. A fix for this is included.

### Changes that have no effect (engine ignores these fields)

| Parameter | Original | Why no effect |
|-----------|----------|---------------|
| `SUSTAIN_ACTION combat` | 1 | String not in binary — dead data |
| `SUSTAIN_ACTION chase` | 3 | String not in binary — dead data |
| `ALERT_LEVEL_DEGRADE` | 20 | String not in binary — dead data |

These values are still present in the config file with the `#SETTINGS FOR DEMO VERSION!!!!!` comment. They document what Rockstar intended but never implemented. Fixing combat persistence and chase duration would require patching the game executable.

## Download

See [Releases](../../releases) for the multi-platform fix package (PSP, PS2, PC).

## Installation

**PSP**: Extract ISO, replace `PSP_GAME/USRDIR/GLOBAL/INIS/AITYPED.INI` and all 16 per-level copies. Load in PPSSPP or rebuild ISO.

**PS2**: Extract ISO, replace `GLOBAL/INIS/AITYPED.INI`. Load in PCSX2 or rebuild ISO.

**PC**: Navigate to your install folder, replace `global/ini/resource6.glg`. Backup original first.

Full instructions in the release ZIP's `README.txt`.

## Verification

Anyone can verify this in under 2 minutes:

```bash
# PSP or PS2
7z e "Manhunt 2.iso" "GLOBAL/INIS/AITYPED.INI" -o/tmp/
grep -n "DEMO VERSION" /tmp/AITYPED.INI

# PC (config is zlib-compressed)
7z e setup.exe "global/ini/resource6.glg"
python3 -c "
import zlib
with open('resource6.glg','rb') as f: d=f.read()
with open('AITYPED.INI','wb') as f: f.write(zlib.decompress(d[8:]))
"
grep -n "DEMO VERSION" AITYPED.INI
```

## Manhunt 1 Connection

The demo AI values originate from Manhunt 1 (2003). Rockstar's `aiTypeData.ini` (MH1) and `AITYPED.INI` (MH2) are **byte-for-byte identical** — same 13,627 bytes, same SHA256 hash. This means the demo values were never a Manhunt 2-specific oversight; they were inherited wholesale from MH1's codebase and left unchanged through the MH2 development cycle.

Neither game ever implemented parsing for the behavior fields (`SUSTAIN_ACTION`, `ALERT_LEVEL_DEGRADE`, etc.). The entire config section was designed for a data-driven AI system that remained unfinished across both titles.

Manhunt 1's config was extracted from `ManHunt.pak` (MHPK format, XOR 0x7F encrypted). See [evidence/VERIFICATION.md](evidence/VERIFICATION.md) for full details.

## Evidence

See [`evidence/`](evidence/) for:
- [Full discovery report](evidence/DISCOVERY.md) — detailed analysis with code excerpts
- [Cross-platform verification](evidence/VERIFICATION.md) — SHA256 hashes across MH1 and MH2, ISO metadata, reproduction steps

## Credits

**Fix & Cross-Platform Analysis**: Jok0ne aka Zerone (February 2026)

**Method**: Systematic analysis of 287+ config files using SQLite FTS5 full-text search and ChromaDB semantic search. Manhunt 1 config extracted via MHPK PAK reverse-engineering (XOR 0x7F decryption). Binary analysis of both MH1 and MH2 PC executables to determine which config fields the engine actually reads. Fix values informed by GTA III/VC/SA source code reference analysis.

**Prior work**: The Manhunt 1 community previously identified the demo AI values. The [Manhunt 2 Improved AI mod](https://www.moddb.com/mods/manhunt-2-improved-ai) on ModDB adjusts vision and hearing ranges. This project adds: cross-platform SHA256 verification, the MH1-MH2 identity proof, binary analysis proving which fields work, and a targeted fix for the fields the engine actually reads.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt freely with attribution.
