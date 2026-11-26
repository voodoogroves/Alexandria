---
name: fo4-archives
description: Offline Creation Kit wiki (check first for Papyrus and record questions); reading vanilla BA2 archives and plugin records on the Deck without xEdit - baunpack, esmdump, esmrec.py, esp.py; packing .ba2 archives with BSArch.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-26
---
**Tool: `baunpack` from [fo76utils](https://github.com/fo76utils/fo76utils)** (MIT, C++, native Linux). Verified 2026-11-24: lists and extracts from `Fallout4 - Materials.ba2` and `Fallout4 - Meshes.ba2`; lists `Fallout4 - Textures*.ba2` (texture extraction not tried).

Build and install (once): `~/Repos/Rhodes/scripts/fo76utils/build-baunpack.sh`. It clones to `~/Repos/upstream/fo76utils` if missing and puts `baunpack` in `~/.local/bin` (not on PATH in an assistant's shell; use the full path). Built against upstream commit `c52da18`; no scons needed.

```
D=~/.local/share/Steam/steamapps/common/"Fallout 4"/Data
baunpack "$D/Fallout4 - Materials.ba2" --list chineseofficer          # list by name pattern
baunpack "$D/Fallout4 - Meshes.ba2" -- chineseofficersword/           # extract into the current directory
```

- Extraction writes the archive paths under the **current directory**; run it from a fresh empty directory, never from the game folder.
- Patterns are substrings, matched case-insensitively against the archive path. Archives listed first win on duplicates.
- Materials (`.bgsm`/`.bgem`) are small binaries; `strings -n 4 file.bgsm` shows the texture paths, cubemap and template they use. Meshes: `strings` shows shader and material paths too. fo76utils also has `bgsmfile.cpp`, `nif_info.cpp` and `esmdump.cpp` (not built) if real parsing is needed.
- The vanilla archives hold *only vanilla*. Creation Club content is in `Data/cc*` and DLC `.ba2`s; list them the same way.

## Reading records (ESM/ESP) without xEdit's GUI

- **`esmdump`** (also fo76utils): build with `~/Repos/Rhodes/scripts/fo76utils/build-esmdump.sh` (about 12 seconds). Lists records as text, e.g. `esmdump "$D/Fallout4.esm" "$D/Fallout4 - Interface.ba2" -i BOOK -edid -n -o book.txt` (`-i TYPE` filters, `-edid` resolves form IDs to names, `-n` drops the stats; the `.ba2` argument supplies localized strings). **It prints only the fields it knows**: BOOK's `DATA`/`DNAM` and `MODS` are missing, for example. Use `-v` for a few more.
- **`esmrec.py`** (ours, `scripts/fo76utils/esmrec.py`, standard-library Python): prints *every* raw subrecord of chosen records by form ID or EditorID, e.g. `python3 -I esmrec.py "$D/Fallout4.esm" BOOK 8E74A` or `... STAT HighPoly_Grognak10`. Walks all of `Fallout4.esm` in about 1.6 seconds. Fields come out as hex, so layouts must be decoded by hand (see `fo4-books-and-magazines` for a decoded example). Localized `FULL`/`DESC` show as 4-byte string IDs.
- Resolving a form ID to a name: dump the type with `esmdump -i TYPE -edid` and grep the ID.
- Both read only; they never write the plugin.
- Script behaviour: `baunpack "$D/DLCRobot - Main.ba2" -- <script name>` extracts `.pex` files; `strings` shows property and function names, not the logic.
- Localized names: extract `fallout4_en.strings` from `Fallout4 - Interface.ba2`. Layout: u32 count, u32 data size, (id, offset) pairs, then NUL-terminated text at those offsets. `esmdump` left `OMOD` names as IDs (2026-11-25).

## Packing `.ba2` archives (no MO2 needed)
xEdit's `BSArch.exe` runs from the VS Code sandbox through Proton's Wine on the host. Working script: `fo4-regalia-of-mystery/build/package.py`. *(2026-11-26)*

```
P="$HOME/.local/share/Steam/steamapps/common/Proton 11.0/files/bin/wine"
flatpak-spawn --host env WINEPREFIX="$HOME/Games/mo2-lint/tools/bsarch-prefix" WINEDEBUG=-all "$P" "$HOME/Games/mo2-lint/tools/xEdit-4.1.5f/BSArch.exe" \
  pack 'Z:\home\deck\...\stage-materials' 'Z:\home\deck\...\MyMod - Main.ba2' -fo4 -z
```

- Two archives, named after the plugin: `<plugin> - Main.ba2` (`-fo4`, materials) and `<plugin> - Textures.ba2` (`-fo4dds`). Each staging folder holds the `Materials\` or `Textures\` folder itself. Use home paths.
- Check by unpacking with `baunpack <archive> -- /` into an empty folder and comparing with the sources (for textures, compare the pixel data after the DDS header).
- The result (format v1, `GNRL` and `DX10`) loaded in game 1.11.240 from one MO2 mod beside a light `.esp`, with the loose copies turned off. *(in game)*

## Offline Creation Kit wiki
Check it before searching the web for Papyrus functions, record fields or perk entry points: the live wiki refused automated reads (2026-11-25).
`reference/fo4-ck-wiki/FO4CKWiki_210520/fallout4/*.html` in Alexandria (gitignored, local to this Deck; the source `.7z` sits in `reference/`). Pages are named like `AddPerkPoints_-_Game.html`; search with `grep`. A 2021 snapshot: later game and F4SE changes are missing.

## Other tools

- Other prior art found, not used: B.A.E. and Archive2 (Windows); a Linux C rewrite of BSArch at github.com/sableeyed/BSArch.

- `~/Repos/Rhodes/scripts/esp/esp.py` also reads any plugin directly.
