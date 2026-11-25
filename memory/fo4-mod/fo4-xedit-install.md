---
name: fo4-xedit-install
description: Downloading, installing and adding FO4Edit to MO2 on the Steam Deck; startup quirks.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-23
---
Facts marked *(upstream)* have not been run here.

## FO4Edit (xEdit)

**Source:** the official `TES5Edit/TES5Edit` GitHub releases page (xEdit is one project; FO4Edit is its Fallout build). No Nexus account needed. Latest release on 2026-11-23 was **xEdit 4.1.5f** (2024-04-28).

How it was fetched and installed (2026-11-23):

```bash
# list releases
curl -s https://api.github.com/repos/TES5Edit/TES5Edit/releases?per_page=4
# download into a scratch dir, then hash
curl -sSL -o xEdit.4.1.5f.7z https://github.com/TES5Edit/TES5Edit/releases/download/xedit-4.1.5f/xEdit.4.1.5f.7z
sha256sum xEdit.4.1.5f.7z
# extract into a new empty folder (bsdtar reads .7z; no 7z needed)
mkdir -p ~/Games/mo2-lint/tools/xEdit-4.1.5f
bsdtar -xf xEdit.4.1.5f.7z -C ~/Games/mo2-lint/tools/xEdit-4.1.5f
# FO4 build: copy the 64-bit Fallout exe under a name xEdit recognises
cd ~/Games/mo2-lint/tools/xEdit-4.1.5f && cp xFOEdit64.exe FO4Edit64.exe
```

- Size 31,345,110 bytes (matches the GitHub release metadata); SHA-256 `54c014da621f83f06a64fd92ddb8e32ed3082d1c65f543dc1c4e432130dced08`. GitHub published no digest to compare against, so the hash is a record, not a verification.
- The archive holds `xFOEdit64.exe`/`xFOEdit.exe` (Fallout), plus TES/SF/Dump builds, BSArch and `Edit Scripts/`. There is no file called FO4Edit in it.
- **Game selection:** xEdit takes the game from a `-FO4` command-line switch or from an exe name containing `fo4` (read from `xEdit/xeInit.pas` on current master, not from the 4.1.5f binary). A copy named `FO4Edit64.exe` needs no arguments. Alternative: keep `xFOEdit64.exe` and pass `-FO4`.

**Observed 2026-11-23 (Proton 11.0, MO2 2.5.2, xEdit 4.1.5f):**
- **Use the 32-bit `FO4Edit.exe`.** `FO4Edit64.exe` showed the "64-bit is slower" prompt, then died on startup with `Runtime error 216` (an access violation inside the exe; the Proton log showed a chain of faults at `FO4Edit64.exe+0x1C8EC4`). Cause not found. The 32-bit copy (`cp xFOEdit.exe FO4Edit.exe`) starts fine.
- **Click OK on the "Not implemented." box.** It appears right after "Loading active plugin list" and is non-fatal; xEdit carries on and opens. Expect a few splash screens (support/donation nags) too.
- **Pick the right exe in MO2.** The first attempt used `xFOEdit64.exe` (no `fo4` in the name, no `-FO4` switch), so xEdit couldn't tell the game. Use `FO4Edit.exe`.
- **Loading works.** The Module Selection list appears with every plugin ticked. For vanilla lookups, right-click → **Select None**, then tick only `Fallout4.esm` and the 6 `DLC*.esm`, and OK. Loading finished ("Background Loader: finished" in the message pane) with a stream of tip/donation screens to close along the way. Entries struck through in the list are in `Plugins.txt` but missing from `Data`; harmless.
- xEdit creates `FO4Edit Cache` and `FO4Edit Backups` folders inside the game's `Data` folder. Normal.
- To debug a crash, set the game's launch options to `PROTON_LOG=1 %command%`; the log lands at `~/steam-377160.log` and is overwritten each launch. Remove it afterwards (it is large and slows things).

Add it to MO2 (a GUI step; don't edit `ModOrganizer.ini` while MO2 is running): gear icon (Edit executables) → **+ → Add from file…** → `~/Games/mo2-lint/tools/xEdit-4.1.5f/FO4Edit.exe` (the 32-bit copy). Title `FO4Edit`. Run it from MO2 so it sees the virtual `Data` folder. Output files land in MO2's `overwrite` folder.
