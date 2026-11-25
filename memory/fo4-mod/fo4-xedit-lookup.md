---
name: fo4-xedit-lookup
description: Looking up vanilla records - running Lookup.pas from MO2 without clicking, request file, lookup gotchas.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
xEdit source facts here were checked against `TES5Edit/TES5Edit` at commit `73f919a` (2026-11-23), mainly `Core/wbDefinitionsFO4.pas`, `Core/wbDefinitionsCommon.pas` and `xEdit/JvI/xejviScriptAdapter*.pas`; reread the source after an xEdit update. Dated entries say what was observed on the Deck.

## Running xEdit scripts without clicking (verified 2026-11-23, Proton 11.0, xEdit 4.1.5f)

- Flags (read from `xEdit/xeInit.pas`): `-script:"file.pas"` (game from the exe name or a `-FO4` switch), `-autoload`, `-autoexit`, `-R:"log"`. Run via MO2's executable Arguments. `-autoload` did not skip the module-selection dialog here (the dialog still needs one OK click); the script still ran after loading. Script work was under a second.
- **xEdit under Proton cannot see `/tmp`.** Put scripts, request files and output under a normal home path and refer to it as `Z:\home\deck\...`. An assistant's shell in the VS Code Flatpak also has a private `/tmp`.
- Launching Proton directly from a shell (`proton run`) fails (no display auth; and it rewrites runtime files in the game's prefix). Don't.
- The tool lives in Rhodes: `scripts/xedit/Lookup.pas`, driven by a plain-text request file.
- Script API facts used: `FileByIndex`, `GroupBySignature`, `MainRecordByEditorID(group, edid)`, `RecordByFormID(file, id, True)`, `ElementCount`/`ElementByIndex` (groups too), `Name`, `GetEditValue`, `EditorID`, `FormID`, `IntToHex`. There is no `HexFormID`.

- **Lookup tool gotchas (2026-11-24):** `file <n>` indexes xEdit's own list, which has one more file than the usual load order before the DLCs (`DLCUltraHighResolution`): with the base game and six DLC ticked Nuka-World is **7** (index 6 was `DLCworkshop03`). `id` resolves a FormID **inside the current file**, so a Nuka-World record (`06xxxxxx`) needs `file 7` first; under another file it returns NOT FOUND, quietly. The script prints the file name it switched to: check it. `grepall` needs the relevant DLC and `cc*.esl` ticked in the module list, and matches text anywhere in a record's dump (so "Disciples" also matches guns). Light (`.esl`) plugins load fine. A weapon sold only in a Creation Club pack that is not installed cannot be read at all.

## Setting up the lookup entry in MO2

**Running xEdit scripts without clicking (verified 2026-11-23).** xEdit can apply a script on startup from MO2's executable Arguments. The reusable tool is `scripts/xedit/Lookup.pas` in Rhodes: edit `~/Games/mo2-lint/tools/our-scripts/requests.txt` (`id`, `edid`, `list`, `file` and `npcstats` lines; see the script header), set the `FO4EDIT` executable's Arguments to:

```
-autoload -autoexit -script:"Z:\home\deck\Repos\Rhodes\scripts\xedit\Lookup.pas" -R:"Z:\home\deck\Games\mo2-lint\tools\our-scripts\xe.log" Fallout4.esm
```

Run it from MO2, click OK on the module list (it still appears) and the "Not implemented" box, and read `our-scripts/out.txt`. It takes seconds after loading.

- **Use home paths (`Z:\home\deck\...`) for everything.** xEdit runs inside Proton and cannot see `/tmp`; it will say "Could not open script" for a `/tmp` path. An assistant's shell in the VS Code Flatpak has a private `/tmp` too.
- After editing Arguments, re-open the gear to confirm it saved; a stale old line cost a retry.
- **Keep two executable entries:** `FO4EDIT` with empty Arguments for normal GUI use, and a second one (e.g. `FO4EDIT lookup`) with the script Arguments. If the script Arguments are left on the normal entry, opening FO4Edit runs the script and exits.
- **Do not launch xEdit by running `proton run` from a shell.** It fails (no display authorisation) and rewrites Proton runtime files in the game's prefix.
- Reference ids are in Alexandria `memory/fo4-mod/fo4-vanilla-records.md`.
