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

## Test enemies: armor and health (`Fallout4.esm` `NPC_` records, FO4Edit 4.1.5f, 2026-11-23)

Read from each record's `PRPS` properties. "Armor" is the `DamageResist` actor value. These are the record's base values, **not checked in game**: many NPCs inherit stats from a template (`Use Template Actors` -> Stats) and leveled actors can scale, so confirm in the console on a selected target (`getav damageresist`, `getav health`; general knowledge, not run on this setup).

| Enemy | FormID | Level | Health | Armor (`DamageResist`) | `EnergyResist` |
|---|---|---|---|---|---|
| Assaultron Invader (`EncAssaultron02`) | `000D8418` | 36 | 400 | 105 | 60 |
| Deathclaw (`DN092AmbushDeathclaw`) | `0002C969` | 22 | from template `LvlDeathclaw` `00075338` | 100 | 150 |
| Mirelurk King (`EncMirelurkKing01`) | `000B7F95` | 30 | 350 | 95 | 150 |
| Mirelurk Hunter (`EncMirelurkHunter01`) | `00064C5E` | 24 | 400 | 85 | 85 |
| Mirelurk Crab (`LvlMirelurkCrab`) | `0005A9C1` | 5 | 120 | 45 | 30 |
| Protectron (`BoS302LvlProtectron`) | `0003A5E5` | 5 | 100 | 40 | 25 |
| Mr. Handy (`REAssaultSC06_MrHandy`) | `0003D62F` | 7 | 100 | 30 | 25 |
| Super Mutant melee (`LvlSupermutantMelee`) | `00088F14` | 10 | 120 | 10 | 5 |

- **Armored target plus control for damage tests:** Mirelurk Hunter (armor 85, health 400, enough to survive several hits) against the melee Super Mutant (armor 10). The Assaultron and Deathclaw have the most armor but are dangerous, the Deathclaw's health is not on its own record, and the Mirelurk Crab (health 120) dies too fast to measure.
- Spawn by console: `player.placeatme <FormID>` (general knowledge, not run here). `tai` freezes an actor and `tgm` is god mode for the player.
- Method for measuring damage without a mod: select the target by clicking it with the console open, `getav health`, land one hit, `getav health` again; the difference is the damage.
- The `Lvl...` NPC records (about 3,000 `NPC_` records in total) are leveled templates; the named `Enc...` and `DN...` records are the concrete ones. Full list: ask `Lookup.pas` for `list NPC_`.

## Fallout 4 damage resistance (formula verified 2026-11-23 against the wiki page)

- **Formula (verified; Chris read it on the Fandom "Damage Resistance" page):** `DamageCoeff = max(0.01, min(0.99, (0.15 x Damage / Resistance)^0.365))`, `DamageFinal = Damage x DamageCoeff`. Roughly `0.5 x (Damage/Resistance)^0.365`. The page's rendered equation is the source; fallout.wiki (summary fetches) agreed except for the 0.01 floor. The Fandom copy blocks automated reads with HTTP 402.
- Examples from the page: damage equal to DR gives about a 50% reduction (10/10, 50/50, 100/100 all about 5/25/50).
- Armor penetration is reported as a multiplier on the target's DR (e.g. Rifleman rank 5 x 0.7), which matches the perk entry we decoded (`Multiply 1 + Actor Value Mult`). Critical hits ignore armor (the one line read verbatim). Energy resistance is reported as bugged / not reduced in Fallout 4.
- **Predictions at 32 damage vs DR 40** (from the reported formula): plain about 14.7 per hit; DR x0.7 (stat 30) about 16.8 (+14%); DR 0 (stat 100) about 31.7 (x2.15). For DR 85 the full-penetration ratio is about x2.8.

## Blade of Bastet armor-penetration test results (Chris, Test profile, 2026-11-23)

Spawned `player.placeatme 00106B07` (a Protectron), Strength 10 so both swords show 32 damage, hits to kill, plain swings, single runs:

| Weapon | `ArmorPenetration` stat | Hits to kill |
|---|---|---|
| Plain Chinese Officer Sword | 0 | 22 to 23 |
| Blade of Bastet (spec build) | 30 | 22 |
| Blade of Bastet TEST100 | 100 | 18 |

- Expected ratios if the chain applied fully: about 1.14 (stat 30) and about 2.15 (stat 100). Observed about 1.0 and 1.2: **an effect smaller than predicted, or noise.** Early impression was a faster-draining health bar. Single runs and random criticals (which ignore armor) make this inconclusive; repeated counts (3 each) and a more heavily armored target are the next steps.
- Chris's check: 18 swings x 32 damage is about 570, far more than a 100-health target. **Alternative reading (2026-11-23):** if the stat-100 build sits at the 0.99 cap and the target has about 570 HP, the plain sword's coefficient was about 0.79 (effective DR about 9, not 40), so full penetration could gain at most 0.99/0.79 = 1.25x, and 22.5/1.25 = 18, matching the result. The penetration may already be working on a soft target. HP is inferred, not measured, so this is not proof. **Those runs were on Very Hard** (difficulty scales player damage), so the 570 HP and effective-DR-of-9 figures are unreliable; only the ratios between weapons hold. Decisive test: a heavily armored target (Mirelurk Hunter `00064C5E`, DR 85, about 2.8x at the cap).
- **Record stats did not predict the spawned actor.** The three Protectron records (`00106B07` LvlProtectron, `00106B09` EncProtectron01Template, `00186E01` LvlProtectron_Hostile) all read health 100, DR 40, level 5, yet the spawned one took about 22 swings (about 7 expected). Templates and level scaling are involved. `00106B09` was suggested as a fixed-stat spawn; not yet tried.
- Reading `getav health` in the console was unreliable on the Deck; screenshots attached to chat are readable. Counting hits to kill is the practical measure.

## Armor penetration in vanilla: how it is attached, and what failed (2026-11-23)

- **Console checks that work:** `player.getav ArmorPenetration` and `player.hasperk 001F4166` on an equipped weapon. The Sword of Wonders (vanilla legendary Penetrating) reads 30.00 and Rank 1. A weapon built by our scripts read 0 and none, three times.
- **Three vanilla patterns (all read from the records):** mundane melee mods add keyword `ap_melee_ArmorPenetrate` (`00097342`), which an entry in the base `PlayerPerk` (`00056DD7`, Mod Target Damage Resistance, Multiply Value 0.85, weapon-tab `HasKeyword`) turns into a flat 15%; mundane gun Armor Piercing receivers carry the trio keyword + `enchModArmorPenetration` + `ActorValues ArmorPenetration` 40; the legendary Penetrating mod (`001F4426`, `ap_Legendary`) carries the trio at 30. Perk entry condition tabs: 0 attacker, 1 weapon, 2 target.
- **Only four vanilla weapons ship with a legendary mod in their template** (all `DLCRobot.esm`: Tesla Rifle, two Sentry Bot miniguns, Salvaged Assaultron Head), as a Default-combination include with `Attach Point Index` 0. Copying that onto a crafted sword did not make it legendary.
- **The cheat terminal's legendary menu** calls `AttachMod` on the grabbed object (`Game.GetPlayerGrabbedRef`): a runtime attach to an item instance. The `.pex` scripts are in `cheatterminal - main.ba2` (a plain archive; a short Python reader lists and extracts the files).
- **xEdit script gotcha (cost us several tests):** an OMOD's data block is `ElementBySignature(o, 'DATA')`. `ElementByName(o, 'Data')` returns nothing, so property edits silently do nothing and counts read 0. Always log counts and fail on a missing element.
- **Gemini's record names** for these (`mod_Legendary_Weapon_IgnoreArmor`, `Perk_Legendary_Weapon_IgnoreArmor`, `dn_HasMeleeMod_ArmorPenetration`) do not exist in the game data; its `ArmorPenetration` stat description ("0-100, % armor ignored") was right. Treat its structure as a lead and its identifiers as unverified.
- **Console `help` syntax that works:** `help "word" 4 WEAP`; the wiki "Form ID" of a unique can be a placed reference, not the item.

## Running xEdit scripts without clicking (verified 2026-11-23, Proton 11.0, xEdit 4.1.5f)

- Flags (read from `xEdit/xeInit.pas`): `-script:"file.pas"` (game from the exe name or a `-FO4` switch), `-autoload`, `-autoexit`, `-R:"log"`. Run via MO2's executable Arguments. `-autoload` did not skip the module-selection dialog here (the dialog still needs one OK click); the script still ran after loading. Script work was under a second.
- **xEdit under Proton cannot see `/tmp`.** Put scripts, request files and output under a normal home path and refer to it as `Z:\home\deck\...`. An assistant's shell in the VS Code Flatpak also has a private `/tmp`.
- Launching Proton directly from a shell (`proton run`) fails (no display auth; and it rewrites runtime files in the game's prefix). Don't.
- The tool lives in Rhodes: `scripts/xedit/Lookup.pas`, driven by a plain-text request file.
- Script API facts used: `FileByIndex`, `GroupBySignature`, `MainRecordByEditorID(group, edid)`, `RecordByFormID(file, id, True)`, `ElementCount`/`ElementByIndex` (groups too), `Name`, `GetEditValue`, `EditorID`, `FormID`, `IntToHex`. There is no `HexFormID`.

- **Lookup tool gotchas (2026-11-24):** `file <n>` indexes xEdit's own list, which has one more file than the usual load order before the DLCs (`DLCUltraHighResolution`): with the base game and six DLC ticked Nuka-World is **7** (index 6 was `DLCworkshop03`). `id` resolves a FormID **inside the current file**, so a Nuka-World record (`06xxxxxx`) needs `file 7` first; under another file it returns NOT FOUND, quietly. The script prints the file name it switched to: check it. `grepall` needs the relevant DLC and `cc*.esl` ticked in the module list, and matches text anywhere in a record's dump (so "Disciples" also matches guns). Light (`.esl`) plugins load fine. A weapon sold only in a Creation Club pack that is not installed cannot be read at all.
- **Plugin script gotchas (2026-11-24):** `wbCopyElementToFile(src, file, True, True)` makes a new record; with `False` as the third argument it makes an **override** of the same FormID (used to extend a vanilla naming record). `ElementAssign(container, HighInteger, srcElement, False)` copies an existing element into a list (used to duplicate a perk entry and a naming rule). `ElementByPath(rec, 'A\B')` works. After adding a rule to an array, set the array's count field (`VNAM - Count`) yourself. Removing a property entry: `Remove(p)`, then set `Property Count`.

- **Cross-plugin references in a new file (verified 2026-11-24, xEdit 4.1.5f):** `FormID(e)` returns the ID in the numbering of the file the record **lives in**, not its load-order ID. For a record of the file you are building that is exactly right (write it as is). For a record of **another plugin** it is wrong: writing it into your new file makes it read as one of *your* records (an Eye keyword became the file's own magic effect, silently). Convert instead: `raw := LoadOrderFormIDtoFileFormID(f, GetLoadOrderFormID(rec))`, after `AddMasterIfMissing(f, 'OtherPlugin.esp')`, and decide by `GetFileName(GetFile(rec)) = GetFileName(f)`. Passing `FormID(rec)` into `LoadOrderFormIDtoFileFormID` raises "Load order FileID [nn] can not be mapped". For records from `Fallout4.esm` (file index 0) both give the same number, which is why vanilla references always worked. **Always log the parameter you wrote** (`GetElementEditValues(ctda, 'Parameter #1')`): the wrong record shows up by name.
- **A failed script run can still leave a partial plugin:** if you accept the save prompt after an abort, a few hundred bytes of an empty plugin land in `Data`; answer No, or delete it before rerunning (the filename box refuses an existing name).

## Known unknowns

- FormIDs for other vanilla records (templates and so on) are not verified here; add them to a request file and run `Lookup.pas`.
- Whether a particular enchantment condition can see the wielder (Subject vs Reference) in-game is untested.
