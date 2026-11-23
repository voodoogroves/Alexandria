# FO4 / xEdit scripting reference

Verified against the official xEdit source (`TES5Edit/TES5Edit`, shallow clone at commit `73f919a`, 2026-11-23), mainly `Core/wbDefinitionsFO4.pas`, `Core/wbDefinitionsCommon.pas` and `xEdit/JvI/xejviScriptAdapter*.pas`. Reread the source before trusting this after an xEdit update. Not tested in-game.

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

## Armor Addon (ARMA) models

- `MOD2` is the **Male** biped model and `MOD3` is the **Female** one (first person: `MOD4` male, `MOD5` female). The Gemini brainstorm and its script have these swapped.
- Either model may be left out in the record definition; what a character of the other gender sees in that case is untested here.

## Biped slots (first-person flags)

`46` Headband, `47` Eyes, `48` Beard, `49` Mouth, `50` **Neck** (`0x00100000`), `51` **Ring** (`0x00200000`), `52` Scalp, `54` Unnamed. The Gemini brainstorm calls 52 the Ring slot; it is Scalp.

## Vanilla asset paths (checked against `Fallout4 - Meshes.ba2` / `Materials.ba2`, Steam install, 2026-11-22)

- **Slinky (red) dress:** worn meshes `Meshes\Clothes\SlinkyDress\DressF.nif` and `DressM.nif` (a male version exists); material `Materials\Clothes\SlinkyDress\SlinkyDress.BGSM`. There is no `Clothes\RedDress\`.
- **Tuxedo:** `Meshes\Clothes\Tuxedo\SuitF.nif` / `SuitM.nif`; material `Materials\Clothes\Tuxedo\Tuxedo.BGSM`.
- **Maxson cape:** worn meshes `Meshes\Armor\KnightUnderArmor\FCapeMaxon.nif` / `MCapeMaxon.nif` (ground object `CapeGO.nif`); one material, `Materials\Armor\KnightUnderArmor\CapeMaxon.BGSM`. Textures are `MaxonCape_d/n/s.DDS`. There is no `MaxonCape.nif`, `MaxonCape.bgsm` or `MaxonCapeClasp.bgsm`.
- **Pool table:** only `Materials\SetDressing\PoolTable\PoolTable01.BGSM` (whole table, not felt-only). There is no `Props\PoolTable\PoolTableFelt.bgsm` and no `Props\Trophy\` materials.
- Method: the BA2 name tables, plus reading the `.bgsm` strings inside each `.nif`. A material swap only works on a material the mesh actually references.

## Vanilla records for the Blade (Fallout4.esm, Steam install, 2026-11-22)

- **Chinese Officer Sword:** `WEAP` `00147BE4` (`ChineseOfficerSword`). The Gemini script's `$000330B8` is wrong.
- **Armor penetration chain used by the vanilla legendary:** OMOD `001F4426` `mod_Legendary_Weapon_ArmorPenetrating` -> ENCH `001F4425` `enchModArmorPenetration` -> MGEF `001F4423` `ModArmorPenetrationAddPerkEffect` -> PERK `001F4166` `ModArmorPenetrationPerk`, whose single entry point carries actor value `ArmorPenetration` (AVIF `00097341`) and a small float. Decoded in xEdit 2026-11-23: entry point Mod Target Damage Resistance, function "Multiply 1 + Actor Value Mult", float `-0.01`, so the target's resistance is multiplied by `1 + (ArmorPenetration x -0.01)`; a weapon stat of 30 gives x0.70. The OMOD supplies the stat as an `ActorValues` property (`ArmorPenetration` ADD `30.0`) and the enchantment as an `Enchantments` property (`001F4425`).
- Other armor-ignoring records exist for armor linings (`mod_armor_*_IgnoreArmor`, `Mod_IgnoreArmor_AV` `00245BA4`).
- A weapon's own enchantment is its base `EITM`, separate from the legendary OMOD slot (`ap_Legendary`).

## Vanilla armor penetration (from the game's own strings and records)

- Actor value `ArmorPenetration` (AVIF `00097341`); its description reads "For weapons with Armor Penetration. 0-100 (treat as % armor ignored)". It is a weapon stat, a percentage of the target's armor that is ignored.
- Legendary "Penetrating" (OMOD `001F4426`) sets that weapon property to `30.0`, i.e. ignores 30% of the target's armor.
- Rifleman perk text: ranks ignore 15%, 20%, 25% and 30% of a target's armor (non-automatic rifles).
- Armor-lining mods for melee/unarmed: "Unarmed and Melee attacks ignore some of target's armor" (`Mod_IgnoreArmor_AV`, `00245BA4`).
- Anchors for tuning: 15% (Rifleman rank 1) and 30% (legendary Penetrating).

## Vanilla records read in xEdit (`Fallout4.esm`, FO4Edit 4.1.5f on the Steam Deck, 2026-11-23)

- **Chinese Officer Sword `WEAP 00147BE4`:** `DNAM` `Damage - Base` = 16; speed 1.0; reach 0.8; AP cost 35; crit damage mult 2.0; stagger Small; animation `OneHandSword`; value 50; weight 3.0. Default template includes only `mod_melee_ChineseOfficerSword_Null` (`OMOD 00176DF8`, no properties). The record's `Damage - Secondary` (about `-1.07e8`) is an unused field.
- **Components `CMPO` (31 total):** Steel `0001FABD`, Silver `0001FABB`, Gold `0001FAA6`, Crystal `0001FA9F`, Adhesive `0001FAA5`; also Circuitry `0001FA9B`, Aluminum `0001FA91`, Copper `0001FA9C`, Lead `0001FAAD`, Oil `0001FAB4`, Rubber `0001FAB9`, Fiberglass `0001FAA1`, Screws `0003D294`, Wood `0001FAC2`, Cloth `001223C7`.
- **Perk `Blacksmith02` `PERK 0004B26A`:** "You gain access to Rank 2 melee weapon mods." Its own prerequisite is a Strength check.
- **Keywords:** `WorkbenchChemlab` `KYWD 00102158`; `RecipeUtility` `KYWD 0006980C` (display name "UTILITY", type Recipe Filter).
- **Enchantment `ENCH 001F4425` `enchModArmorPenetration`:** Constant Effect, target Self, effect `MGEF 001F4423`. That MGEF is constant, self, hidden in the UI, archetype shown as "Unknown 36" (the Add Perk archetype in the definitions).

## Running xEdit scripts without clicking (verified 2026-11-23, Proton 11.0, xEdit 4.1.5f)

- Flags (read from `xEdit/xeInit.pas`): `-script:"file.pas"` (game from the exe name or a `-FO4` switch), `-autoload`, `-autoexit`, `-R:"log"`. Run via MO2's executable Arguments. `-autoload` did not skip the module-selection dialog here (the dialog still needs one OK click); the script still ran after loading. Script work was under a second.
- **xEdit under Proton cannot see `/tmp`.** Put scripts, request files and output under a normal home path and refer to it as `Z:\home\deck\...`. An assistant's shell in the VS Code Flatpak also has a private `/tmp`.
- Launching Proton directly from a shell (`proton run`) fails (no display auth; and it rewrites runtime files in the game's prefix). Don't.
- The tool lives in Rhodes: `scripts/xedit/Lookup.pas`, driven by a plain-text request file.
- Script API facts used: `FileByIndex`, `GroupBySignature`, `MainRecordByEditorID(group, edid)`, `RecordByFormID(file, id, True)`, `ElementCount`/`ElementByIndex` (groups too), `Name`, `GetEditValue`, `EditorID`, `FormID`, `IntToHex`. There is no `HexFormID`.

## Known unknowns

- FormIDs for other vanilla records (templates and so on) are not verified here; add them to a request file and run `Lookup.pas`.
- Whether a particular enchantment condition can see the wielder (Subject vs Reference) in-game is untested.
