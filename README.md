# Manhunt 2 Demo AI Fix

Manhunt 2 (Rockstar Games, 2007) shipped on **all platforms** with placeholder AI values that were never reverted from a demo build. This repository documents the discovery and provides a fix.

## Key Facts

- Hunters **fight for 1 second** before disengaging (`SUSTAIN_ACTION combat 1`)
- Hunters **chase for 3 seconds** before giving up (`SUSTAIN_ACTION chase 3`)
- Hunters search with **reduced thoroughness** (35m radius, 85% density)
- The developers marked these values with `#SETTINGS FOR DEMO VERSION!!!!!`
- The file `AITYPED.INI` is **byte-for-byte identical** across PSP, PS2, and PC
- **SHA256** (all platforms): `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83`

## Why This Exists

Manhunt 2 received an AO (Adults Only) rating from the ESRB in June 2007. Sony and Nintendo refused to license AO titles, forcing Rockstar to spend months censoring content (VHS blur effects, cut executions) to secure an M rating. Both console ISOs were mastered on September 5, 2007. During this crunch, the AI config — which controls how aggressively enemies fight — was left at demo values.

This explains 18 years of community complaints that Manhunt 2's hunters feel "braindead" compared to the original. The existing [Manhunt 2 Improved AI](https://www.moddb.com/mods/manhunt-2-improved-ai) mod on ModDB adjusts vision and hearing ranges but **does not address the SUSTAIN_ACTION values** — the core issue documented here. On PC, the config files are stored compressed inside proprietary `.glg` containers, which may explain why they were never found.

## The Fix

| Parameter | Original (Demo) | Fixed |
|-----------|-----------------|-------|
| `ALERT_LEVEL_DEGRADE` | 20 | 25 |
| `SUSTAIN_ACTION combat` | **1** | **3** |
| `SUSTAIN_ACTION chase` | **3** | **5** |
| `RADIUS_m` | 35 | 40 |
| `DENSITY_%` | 85 | 90 |
| `DOUBLE_SEARCH_%` | 15 | 25 |

The `investigate` (8s) value appears to be final and is left unchanged.

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

## Evidence

See [`evidence/`](evidence/) for:
- [Full discovery report](evidence/DISCOVERY.md) — detailed analysis with code excerpts
- [Cross-platform verification](evidence/VERIFICATION.md) — SHA256 hashes, ISO metadata, reproduction steps

## Credits

**Discovery & Fix**: Jok0ne aka Zerone (February 2026)

**Method**: Systematic analysis of 287 config files using SQLite FTS5 full-text search and ChromaDB semantic search.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt freely with attribution.
