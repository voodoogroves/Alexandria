---
name: fo4-perks-and-damage
description: Perks from equipped items with no script, perk entry points, armor penetration chains, damage resistance formula.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Giving the player a perk from an equipped item, with no script

- Chain: `ENCH` (Constant Effect, target Self) -> `MGEF` (archetype "Add Perk" shown as *Unknown 36*, constant, self, hidden; the perk is in `Magic Effect Data / DATA - Data / Perk to Apply`) -> `PERK`. Vanilla's penetration chain is `001F4425 -> 001F4423 -> 001F4166`. Copy and re-point: the ENCH's effect is `Effects / Effect / EFID - Base Effect`. *(xEdit; copies verified in game: `hasperk` is 1 equipped, 0 unequipped)*
- It attaches through an **OMOD** property `Enchantments ADD <ENCH>`, on a mod in an Upgrade slot (`ap_melee_MeleeMod`). A weapon record's own `EITM`, and weapon-level `ActorValues` stats in the object template, did **not** apply anything. *(in game)*
- Console check: `player.hasperk <ID>`. Note a perk added by the console (`player.addperk <ID>`) stays; the chain adds and removes it with the item.

## Perk entries that change damage

- Entry point **Mod Target Damage Resistance**, function **Multiply Value** (or "Multiply 1 + Actor Value Mult" with an actor value): the target's resistance is multiplied. 0.70 ignores 30%, 0.0 ignores all. *(in game)*
- A perk entry has condition **tabs**: 0 = attacker (the player), 1 = weapon, 2 = target. `HasKeyword` on the weapon tab matches a keyword on the equipped weapon (including keywords an OMOD added). `WornHasKeyword` on the attacker tab matches any worn item with the keyword. Both verified. *(in game)*
- **Several entries multiply.** A second entry with multiplier 0.0 and an extra condition zeroes the product, so a "bonus" item can be a bare keyword on a worn item; no script and no per-item perk. *(in game)*
- Vanilla non-legendary melee penetration is one entry in the base `PlayerPerk` (`00056DD7`): 0.85 when the weapon has `ap_melee_ArmorPenetrate`. Copying `PlayerPerk` and keeping one effect makes a clean starting point. *(xEdit)*

## Vanilla armor penetration (from the game's own strings and records)

- Actor value `ArmorPenetration` (AVIF `00097341`); its description reads "For weapons with Armor Penetration. 0-100 (treat as % armor ignored)". It is a weapon stat, a percentage of the target's armor that is ignored.
- Legendary "Penetrating" (OMOD `001F4426`) sets that weapon property to `30.0`, i.e. ignores 30% of the target's armor.
- Rifleman perk text: ranks ignore 15%, 20%, 25% and 30% of a target's armor (non-automatic rifles).
- Armor-lining mods for melee/unarmed: "Unarmed and Melee attacks ignore some of target's armor" (`Mod_IgnoreArmor_AV`, `00245BA4`).
- Anchors for tuning: 15% (Rifleman rank 1) and 30% (legendary Penetrating).

## Armor penetration in vanilla: how it is attached, and what failed (2026-11-23)

- **Console checks that work:** `player.getav ArmorPenetration` and `player.hasperk 001F4166` on an equipped weapon. The Sword of Wonders (vanilla legendary Penetrating) reads 30.00 and Rank 1. A weapon built by our scripts read 0 and none, three times.
- **Three vanilla patterns (all read from the records):** mundane melee mods add keyword `ap_melee_ArmorPenetrate` (`00097342`), which an entry in the base `PlayerPerk` (`00056DD7`, Mod Target Damage Resistance, Multiply Value 0.85, weapon-tab `HasKeyword`) turns into a flat 15%; mundane gun Armor Piercing receivers carry the trio keyword + `enchModArmorPenetration` + `ActorValues ArmorPenetration` 40; the legendary Penetrating mod (`001F4426`, `ap_Legendary`) carries the trio at 30. Perk entry condition tabs: 0 attacker, 1 weapon, 2 target.
- **Only four vanilla weapons ship with a legendary mod in their template** (all `DLCRobot.esm`: Tesla Rifle, two Sentry Bot miniguns, Salvaged Assaultron Head), as a Default-combination include with `Attach Point Index` 0. Copying that onto a crafted sword did not make it legendary.
- **The cheat terminal's legendary menu** calls `AttachMod` on the grabbed object (`Game.GetPlayerGrabbedRef`): a runtime attach to an item instance. The `.pex` scripts are in `cheatterminal - main.ba2` (a plain archive; a short Python reader lists and extracts the files).
- **xEdit script gotcha (cost us several tests):** an OMOD's data block is `ElementBySignature(o, 'DATA')`. `ElementByName(o, 'Data')` returns nothing, so property edits silently do nothing and counts read 0. Always log counts and fail on a missing element.
- **Gemini's record names** for these (`mod_Legendary_Weapon_IgnoreArmor`, `Perk_Legendary_Weapon_IgnoreArmor`, `dn_HasMeleeMod_ArmorPenetration`) do not exist in the game data; its `ArmorPenetration` stat description ("0-100, % armor ignored") was right. Treat its structure as a lead and its identifiers as unverified.
- **Console `help` syntax that works:** `help "word" 4 WEAP`; the wiki "Form ID" of a unique can be a placed reference, not the item.

## Fallout 4 damage resistance (formula verified 2026-11-23 against the wiki page)

- **Formula (verified; Chris read it on the Fandom "Damage Resistance" page):** `DamageCoeff = max(0.01, min(0.99, (0.15 x Damage / Resistance)^0.365))`, `DamageFinal = Damage x DamageCoeff`. Roughly `0.5 x (Damage/Resistance)^0.365`. The page's rendered equation is the source; fallout.wiki (summary fetches) agreed except for the 0.01 floor. The Fandom copy blocks automated reads with HTTP 402.
- Examples from the page: damage equal to DR gives about a 50% reduction (10/10, 50/50, 100/100 all about 5/25/50).
- Armor penetration is reported as a multiplier on the target's DR (e.g. Rifleman rank 5 x 0.7), which matches the perk entry we decoded (`Multiply 1 + Actor Value Mult`). Critical hits ignore armor (the one line read verbatim). Energy resistance is reported as bugged / not reduced in Fallout 4.
- **Predictions at 32 damage vs DR 40** (from the reported formula): plain about 14.7 per hit; DR x0.7 (stat 30) about 16.8 (+14%); DR 0 (stat 100) about 31.7 (x2.15). For DR 85 the full-penetration ratio is about x2.8.
