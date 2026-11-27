---
name: fo4-gotchas-build
description: Rules for building FO4 plugins and models on the Deck without losing rounds. Read before writing or running a build script.
applies-to: [fo4-mod]
kind: gotcha
verified: 2026-11-27
---

# Build gotchas (xEdit 4.1.5f, own model scripts, MO2, Steam Deck)

Detail: `fo4-xedit-scripting`, `fo4-deck-workflow`.

## Scripts
- The release binary is the authority: the upstream source clone is newer. Read the script error text.
- `AddNewFile` takes no arguments. Call `AddMasterIfMissing(file, 'Fallout4.esm')` right after it.
- Another plugin's record: write `LoadOrderFormIDtoFileFormID(f, GetLoadOrderFormID(rec))`, never `FormID(rec)`.
- OMOD data is `ElementBySignature(o, 'DATA')`; by name it silently finds nothing.
- Pick list entries by name, never by index.
- Log every written value and stop with `FAIL` when one is wrong.

## Own models (`fo4-nif-models`)
- Rebuilding a model's string table: rewrite every string index. A lighting shader has two (material and root material). Miss one and the game crashes at random.
- A model crashes the game: get a crash log (`fo4-testing`) before swapping parts.

## Running
- MO2 must be closed before `ModOrganizer.ini` is edited, or it writes the old value back. Check from the host (see `fo4-deck-workflow`) and wait to be told, then have the person check **Applying script "<name>"**.
- Editing the contents of the script already pointed at needs no ini change.
- The plugin is written only if the save prompt is accepted, into the game's real `Data` folder. Use a new name for each run. After an aborted run, answer No or delete the partial plugin.
- xEdit cannot see `/tmp`; use home paths (`Z:\home\deck\...`).
- Never start xEdit with `proton run`.
