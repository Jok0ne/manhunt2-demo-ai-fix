# Manhunt 2 Demo AI Fix

Both Manhunt 1 (2003) and Manhunt 2 (2007) shipped with placeholder AI values intended for demo builds. The community has long known about Manhunt 1's demo AI settings. This repository documents that **Manhunt 2 uses the exact same file** — byte-for-byte identical — and provides a cross-platform fix.

## Key Facts

- Hunters **fight for 1 second** before disengaging (`SUSTAIN_ACTION combat 1`)
- Hunters **chase for 3 seconds** before giving up (`SUSTAIN_ACTION chase 3`)
- Hunters search with **reduced thoroughness** (35m radius, 85% density)
- The developers marked these values with `#SETTINGS FOR DEMO VERSION!!!!!`
- The file `AITYPED.INI` is **byte-for-byte identical** across Manhunt 1 PC, Manhunt 2 PSP, PS2, and PC
- **SHA256** (all versions): `aee092ad6f9de0071472d6e110ad277048ce498ed6fe5576eef4d2b0b3ed2b83`

## Background

The Manhunt 1 community has known for years that the game shipped with demo AI values (`#SETTINGS FOR DEMO VERSION!!!!!`). What was not documented until now:

1. **Manhunt 2 uses the exact same file** — `AITYPED.INI` is byte-for-byte identical (SHA256 match) between Manhunt 1 (2003) and Manhunt 2 (2007). Rockstar copied the config unchanged across 4 years.
2. **Cross-platform verification** — the file is identical across PSP, PS2, and PC versions of Manhunt 2.
3. **No existing fix addresses the core issue** — the [Manhunt 2 Improved AI](https://www.moddb.com/mods/manhunt-2-improved-ai) mod on ModDB adjusts vision and hearing ranges but does not touch the `SUSTAIN_ACTION` values.
4. **PC configs are hidden** — on PC, the config is zlib-compressed inside proprietary `.glg` containers, making it harder to find than on console.

Manhunt 2 received an AO (Adults Only) rating from the ESRB in June 2007. Sony and Nintendo refused to license AO titles, forcing Rockstar to spend months censoring content (VHS blur effects, cut executions) to secure an M rating. Both console ISOs were mastered on September 5, 2007. During this crunch, the inherited demo AI config was never updated.

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

## Manhunt 1 Connection

The demo AI values originate from Manhunt 1 (2003). Rockstar's `aiTypeData.ini` (MH1) and `AITYPED.INI` (MH2) are **byte-for-byte identical** — same 13,627 bytes, same SHA256 hash. This means the demo values were never a Manhunt 2-specific oversight; they were inherited wholesale from MH1's codebase and left unchanged through the MH2 development cycle.

Manhunt 1's config was extracted from `ManHunt.pak` (MHPK format, XOR 0x7F encrypted). See [evidence/VERIFICATION.md](evidence/VERIFICATION.md) for full details.

## Evidence

See [`evidence/`](evidence/) for:
- [Full discovery report](evidence/DISCOVERY.md) — detailed analysis with code excerpts
- [Cross-platform verification](evidence/VERIFICATION.md) — SHA256 hashes across MH1 and MH2, ISO metadata, reproduction steps

## Credits

**Fix & Cross-Platform Analysis**: Jok0ne aka Zerone (February 2026)

**Method**: Systematic analysis of 287+ config files using SQLite FTS5 full-text search and ChromaDB semantic search. Manhunt 1 config extracted via MHPK PAK reverse-engineering (XOR 0x7F decryption). Fix values informed by GTA III/VC/SA source code reference analysis.

**Prior work**: The Manhunt 1 community previously identified the demo AI values. This project adds cross-platform SHA256 verification, the MH1↔MH2 identity proof, and a concrete fix.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt freely with attribution.
