---
name: fo4-deck-workflow
description: The round-by-round build and lookup loop with MO2 - running scripts, editing MO2 ini, saving plugins, adding loose-file mods.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-25
---
Generic tools live in Rhodes `scripts/xedit/`; a project's build scripts live in its repo.

## Building a plugin with xEdit scripts (the Claude workflow)

Claude can build plugin records by writing an xEdit Pascal script; you click Run in MO2 and paste back the message pane (a screenshot works). Each round takes a minute or two. Prototype first, fix from the error text, commit when it proves out. The project's own scripts live in its repo (e.g. `fo4-regalia-of-mystery/build/xedit/BuildBlade.pas`); generic tools live in Rhodes `scripts/xedit/`.

Lessons (2026-11-23, xEdit 4.1.5f under Proton 11.0):
- **The binary is the authority, not the source.** The `TES5Edit` clone in `~/Repos/upstream` is the newer, unreleased `dev-4.1.6`; 4.1.5f lacks some functions it has. `AddNewFileName(name, light, medium)` is source-only: the binary says `Undeclared Identifier`. In 4.1.5f, `AddNewFile` takes no arguments and shows a "Filename without extension" box (type the name; it creates a `.esp`). An EJvInterpreterError in the message pane names the line and the problem ("Type mismatch", "Undeclared Identifier"); read that line first.
- **A new file has no masters.** Copying a record "as new" does not add masters, so references to vanilla records save against a master that does not exist (they show as `<Error: Could not be resolved>` and FormIDs come out with a stray `01`). Call `AddMasterIfMissing(file, 'Fallout4.esm')` right after creating the file. *Confirmed 2026-11-23: with the master added, every reference resolved and the saved plugin header carried the `MAST`.*
- **The plugin is only written if you accept xEdit's save prompt** when you close the window after "You can close this application now." If you close without saving, nothing is written. Don't re-run if the file is already there: the filename box then says "a file of that name already exists" (check `Data/` first; the file can appear a moment after you accept).
- **Check the saved file, not just the message pane.** A small Python read of the plugin header (`MAST` records) showed the missing master at once.
- **The new plugin is written into the game's real `Data` folder**, not MO2's `overwrite` (xEdit uses the `S:` drive path, which MO2's virtual folder doesn't intercept). Move it into an MO2 mod folder afterwards; delete a stale copy from `Data` before re-running a build script.
- **The ESL flag is a later step.** The prototype is a plain `.esp`.
- **Log every step** in the script and write a log file to a home path; one failing step then shows where it stopped.
- **An assistant can't see xEdit, MO2 or the game running.** `pgrep` from the VS Code Flatpak shell doesn't see processes inside Proton, so "nothing found" is not evidence that they're closed (2026-11-23: a plugin was judged "not saved" when xEdit was still open and saved a moment later). Ask the person whether they've closed everything, and check files by timestamp after they say so.

## Editing MO2 settings by file

MO2 rewrites `ModOrganizer.ini` when it exits, so edit it only with MO2 **closed** (back it up first). Executables are the numbered entries under `[customExecutables]`; bump `size=` when adding one. Typing into MO2's Edit Executables fields is error-prone (autocorrect once put a stray `v` in front of `-autoload`, and a mistyped `FO4Edit.exe` gave "not found"), so for long Arguments lines let Claude write the entry, then reopen MO2.

## The round-by-round workflow (settled 2026-11-24)

How a build or lookup round runs on the Deck, so each round costs the person as few steps as possible:

1. **Claude writes the script or the request file, then asks for MO2 to be closed** only when an `ModOrganizer.ini` edit is needed. Claude backs the ini up (`ModOrganizer.ini.bak-<date><letter>`), changes the one line (`5\arguments=` is the BUILD entry, 7 is the lookup entry) with `sed`, and shows the diff.
2. **Lookups:** Claude edits `our-scripts/requests.txt` (keep the old one in `old/`). The person runs the lookup entry, clicks OK on the module list (ticking whatever the request needs: DLC, `cc*.esl`) and the "Not implemented" box, and says "done". Claude checks `out.txt`'s timestamp against the request time, then reads it.
3. **Builds:** the person runs the BUILD entry, types the plugin name, closes xEdit and accepts the save prompt. Run once per plugin name (the filename box refuses an existing one); a new test is a new name. Claude checks `build.log` and the saved `.esp` (timestamps, master, record names) before reading the log's values.
4. **In game:** Claude gives console commands in fenced code blocks, one per paste target (the console accepts a paste and the editor shows a copy button). The person reports numbers or screenshots.
5. **Commit as you go:** results into the project spec, a short checkpoint in Rhodes, general mechanics into the matching Alexandria `memory/fo4-mod/` doc; PR and merge each, then fast-forward the working branch.

**Learned 2026-11-25:**
- **MO2 left open during an ini edit writes its old value back**, and the next BUILD silently runs the previous script. `pgrep`/`ps` cannot see MO2, so wait for the person to say it is closed, and have them check the xEdit line **Applying script "<name>"**.
- Editing the **contents** of the script the BUILD entry already points at needs no ini change and no MO2 restart.
- Creation Club content downloaded through MO2 lands in the instance's `overwrite` folder; xEdit loaded only the `cc*` files in the real `Data` folder. `scripts/esp/esp.py` reads any plugin directly.

## Adding loose files (textures, materials) as an MO2 mod (verified 2026-11-24)

- Make a folder under `mods/` (e.g. `mods/Blade of Bastet Art/` with `Textures/` and `Materials/` inside, plus a tiny `meta.ini`), and add `+<mod name>` to the profile's `modlist.txt` (CRLF line endings) with MO2 **closed**. Back both files up first.
- Loose files need, in the profile's `fallout4custom.ini` (profile ini files must be on: `LocalSettings=true` in `settings.ini`): `[Archive]`, `bInvalidateOlderFiles=1`, `sResourceDataDirsFinal=` (empty). Vanilla has `sResourceDataDirsFinal=STRINGS\`, which stops loose textures and materials loading. This worked in game on 2026-11-24.
- Never put a generated file under a vanilla path; use your own folder name so nothing vanilla is overwritten.
- Art derived from Bethesda's textures stays out of the (public) repos; commit the script that makes it. Every repo has a `.gitignore` for assets (Creation Kit licence: modding use only; a public repo allows broader use; see Rhodes `docs/decisions.md`, 2026-11-24).
- **Sequence mistake to avoid (2026-11-24):** if Claude has to re-point the BUILD entry, it needs MO2 closed first; running the BUILD entry before the re-point runs the *previous* script (it produced a mislabeled duplicate plugin once). Claude says "do not run it until I tell you it is re-pointed".
