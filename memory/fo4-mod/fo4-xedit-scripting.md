---
name: fo4-xedit-scripting
description: Writing xEdit Pascal build scripts - condition indices and flags, script API, masters, cross-plugin FormIDs.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-26
---
xEdit source facts here were checked against `TES5Edit/TES5Edit` at commit `73f919a` (2026-11-23), mainly `Core/wbDefinitionsFO4.pas`, `Core/wbDefinitionsCommon.pas` and `xEdit/JvI/xejviScriptAdapter*.pas`; reread the source after an xEdit update. Dated entries say what was observed on the Deck.

## Condition functions (CTDA `Function` index)

| Function | Index | Params |
|---|---|---|
| GetItemCount | 47 | base object |
| GetStageDone | 59 | quest, quest stage |
| GetInFaction | 71 | faction |
| GetEquipped | 182 | base object |
| IsSneaking | 286 | none |
| HasPerk | 448 | perk |
| GetPlayerTeammate | 453 | none (no `IsPlayerTeammate` exists in FO4) |
| HasKeyword | 560 | keyword |
| WornHasKeyword | 682 | keyword |

## Condition `Type` byte

- Low bits are flags: `1` Or, `2` Use Aliases, `4` Use Global, `8` Use Packdata, `16` Swap Subject and Target.
- Comparison is `Type and 224`: `0` equal, `32` not equal (by elimination; not read directly), `64` greater, `96` greater-or-equal, `128` less, `160` less-or-equal.
- So `>= 1` is Type `96` and `== 1` is Type `0`. The OR flag sits on the condition that is OR'd with the one after it.

## Condition `Run On`

`0` Subject, `1` Target, `2` Reference, `3` Combat Target, `4` Linked Reference, `5` Quest Alias, `6` Package Data, `7` Event Data, `8` Command Target, `9` Event Camera Ref, `10` My Killer.

Note `Target` is `1`, not `2`. Using `2` selects *Reference*.

## xEdit Pascal script host

- `AddNewFile` exists and, in the 4.1.5f release binary, takes **no arguments** (it shows a "Filename without extension" box and creates a `.esp`). `AddNewFileName(name, isLight, isMedium)` and the light-flag arguments exist only in the newer `dev-4.1.6` source, not in 4.1.5f (`Undeclared Identifier` / `Type mismatch`, verified 2026-11-23). `CreateNewFile` does not exist.
- **The source clone is newer than the release binary.** Anything read from `~/Repos/upstream/TES5Edit` may be missing from FO4Edit 4.1.5f; the script error text is authoritative.
- `AddMasterIfMissing(file, masterName)` is built in. Redefining it in a script as a wrapper that calls itself recurses forever. A copy made with `wbCopyElementToFile(..., True, True)` does **not** add masters: without it the saved plugin had no `MAST` and vanilla references came out as `<Error: Could not be resolved>` with an extra `01` in the FormID (observed 2026-11-23; calling `AddMasterIfMissing(file, 'Fallout4.esm')` right after `AddNewFile` fixed it, confirmed the same day by the saved header and zero unresolved references).
- A plugin created from a script is saved into the game's real `Data` folder (the `S:` path), not into MO2's `overwrite`.
- `wbCopyElementToFile(element, file, asNew, asDeepCopy)` exists.
- `AddNewRecord` was not found in the script adapters; add records with `Add(file, 'SIG', True)` (container `Add`).
- `ElementAssign(container, HighInteger, nil, False)` is the append pattern; `HighInteger` is a defined constant.
- Script entry points run per selected record (`Process`). One-time setup belongs in `Initialize`.

## Plugin script and cross-plugin gotchas

- **Plugin script gotchas (2026-11-24):** `wbCopyElementToFile(src, file, True, True)` makes a new record; with `False` as the third argument it makes an **override** of the same FormID (used to extend a vanilla naming record). `ElementAssign(container, HighInteger, srcElement, False)` copies an existing element into a list (used to duplicate a perk entry and a naming rule). `ElementByPath(rec, 'A\B')` works. After adding a rule to an array, set the array's count field (`VNAM - Count`) yourself. Removing a property entry: `Remove(p)`, then set `Property Count`.
- **Cross-plugin references in a new file (verified 2026-11-24, xEdit 4.1.5f):** `FormID(e)` returns the ID in the numbering of the file the record **lives in**, not its load-order ID. For a record of the file you are building that is exactly right (write it as is). For a record of **another plugin** it is wrong: writing it into your new file makes it read as one of *your* records (an Eye keyword became the file's own magic effect, silently). Convert instead: `raw := LoadOrderFormIDtoFileFormID(f, GetLoadOrderFormID(rec))`, after `AddMasterIfMissing(f, 'OtherPlugin.esp')`, and decide by `GetFileName(GetFile(rec)) = GetFileName(f)`. Passing `FormID(rec)` into `LoadOrderFormIDtoFileFormID` raises "Load order FileID [nn] can not be mapped". For records from `Fallout4.esm` (file index 0) both give the same number, which is why vanilla references always worked. **Always log the parameter you wrote** (`GetElementEditValues(ctda, 'Parameter #1')`): the wrong record shows up by name.
- **A failed script run can still leave a partial plugin:** if you accept the save prompt after an abort, a few hundred bytes of an empty plugin land in `Data`; answer No, or delete it before rerunning (the filename box refuses an existing name).

## More script facts
- A condition list cannot be created on a spell effect with `Add`; put the condition on the perk entry. *(2026-11-25)*
- A helper that converts another plugin's FormIDs: `Ref` in `fo4-regalia-of-mystery/build/xedit/BuildVoice3.pas`.
- `scripts/esp/esp.py` in Rhodes reads any plugin directly, including ones xEdit did not load.

## A light (ESL-flagged) plugin from a script *(4.1.5f, 2026-11-26)*
- `header := ElementByIndex(f, 0)`. **Before creating any record**, `SetElementNativeValues(header, 'HEDR\Next Object ID', $800)`; a new file otherwise numbers records from `000001`. Check that the first record got `000800`.
- At the end set the flag: `SetElementNativeValues(header, 'Record Header\Record Flags', GetElementNativeValues(header, 'Record Header\Record Flags') or $200)`. The file stays `.esp`.
- Read the saved file back: flag set, all own object IDs within `800` to `FFF`. The flag is bit `0x200` of the 4 bytes at offset 8, so it can be cleared on the saved file for a comparison test.
- In game it loaded as a light plugin; overrides of exterior and interior cells, new placed references and a `DLCNukaWorld.esm` master all worked. *(in game)*
- Working script: `fo4-regalia-of-mystery/build/xedit/BuildRegalia.pas`.

## Known unknowns

- FormIDs for other vanilla records (templates and so on) are not verified here; add them to a request file and run `Lookup.pas`.
- Whether a particular enchantment condition can see the wielder (Subject vs Reference) in-game is untested.
