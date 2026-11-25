---
name: fo4-vanilla-records
description: Verified vanilla FormIDs (sword, penetration chain, components, keywords) and test enemy stats.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-23
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Vanilla records for the Blade (Fallout4.esm, Steam install, 2026-11-22)

- **Chinese Officer Sword:** `WEAP` `00147BE4` (`ChineseOfficerSword`). The Gemini script's `$000330B8` is wrong.
- **Armor penetration chain used by the vanilla legendary:** OMOD `001F4426` `mod_Legendary_Weapon_ArmorPenetrating` -> ENCH `001F4425` `enchModArmorPenetration` -> MGEF `001F4423` `ModArmorPenetrationAddPerkEffect` -> PERK `001F4166` `ModArmorPenetrationPerk`, whose single entry point carries actor value `ArmorPenetration` (AVIF `00097341`) and a small float. Decoded in xEdit 2026-11-23: entry point Mod Target Damage Resistance, function "Multiply 1 + Actor Value Mult", float `-0.01`, so the target's resistance is multiplied by `1 + (ArmorPenetration x -0.01)`; a weapon stat of 30 gives x0.70. The OMOD supplies the stat as an `ActorValues` property (`ArmorPenetration` ADD `30.0`) and the enchantment as an `Enchantments` property (`001F4425`).
- Other armor-ignoring records exist for armor linings (`mod_armor_*_IgnoreArmor`, `Mod_IgnoreArmor_AV` `00245BA4`).
- A weapon's own enchantment is its base `EITM`, separate from the legendary OMOD slot (`ap_Legendary`).

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
