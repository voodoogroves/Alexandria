---
name: fo4-weapon-mods-and-names
description: Weapon mod (OMOD) properties, item naming rules, workbench filtering, vanilla melee numbers.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Weapon mod (OMOD) properties

Entries under `DATA / Properties`: `Property`, `Value Type`, `Function Type`, `Value 1`, `Value 2`. Forms that work:
- Numbers: `Float`, `MUL+ADD`, `Value 1` = the amount. **They add together into one multiplier**: the mod's `AttackDamage` of 0.75 is x1.75; bumping it by 0.45 gives 1.2 (x2.2). *(in game: 32 -> 70)* Edit the existing entry in place instead of adding a second one.
- References (keywords, enchantments): `FormID,Int`, `ADD`, `Value 1` = the FormID, `Value 2` = 440.
- Verified in game on a sword: `AttackDamage`, `Weight` (-0.33 took 3 to 2), `AttackActionPointCost` (-0.43; VATS swings 2 -> 5 on the same AP).
- **`Speed` does nothing visible on a melee weapon** (+0.45 felt identical). The vanilla examples are guns (the Rapid legendary). *(in game)*
- **Swing speed** is on the weapon record: `MASE - Melee Speed` (Fast / medium / ...), `Animation Attack Seconds` (0.77 fast, 1.12 sword), the keyword `Anims1hmShortWeapon` (short-blade animations), AP cost and stagger. Disciples Blade is fast through these; its `Speed` field is 1.0 like every other melee weapon. An OMOD cannot set them as far as we found. *(xEdit)*
- Mods can **add or remove keywords** but removing a keyword needs deleting its property entry from the copy; do it by matching `Value 1` to the FormID.

## Item names (naming rules)

- A weapon points at a naming record with `INRD` (the melee ones use `dn_CommonMelee`, `INNR 000B9738`). The record holds rulesets of rules: `WNAM` text, optional `KWDA` keywords, `YNAM` index. The shown name is built from the matching rules, and the last rule `*` (index 10000) is the item's own name. *(xEdit)*
- Name words come from **keywords on the weapon or its mods**, not from the OMOD's own `FULL` (that is only the name in the mod list and workbench menu). A copy of Electrified Serrated kept `dn_HasMeleeMod_Serrated` and `dn_HasMeleeMod_Shock`, so the item still said "Electrified Serrated". *(in game)*
- Vanilla melee name words include Serrated, Electrified, Sharp, Bladed, Heavy, Weighted, Spiked, Heated, and colours (Black, Red, Oak, Mahogany, Grey, Blue, Aluminum, Cedar). There is no gold word. *(xEdit)*

- **Verified 2026-11-24:** a naming rule's text lands **in front** of the base name even at index 10001 (the base `*` is 10000). So we found no way for a rule to add a suffix, and ordering is not by index alone. Overriding the shared naming record would cost mod compatibility. **The way to a custom full name is a weapon record of its own**: copy the base weapon, give it its own `FULL`, and keep name keywords off its mods (no Serrated), so no rule adds words. *(in game)*
- **A custom weapon can carry its mod through its template**: replace the `Mod` of every template include (`Object Template / Combinations / <combination> / OBTS / Includes / Include / Mod`) that was the base's "No Upgrade" with your own Upgrade-slot mod. It attached on a crafted item (damage, weight, VATS, perk all applied). A *legendary* mod in a template include did not. *(in game)*
- **Workbench mod lists are filtered by keyword (verified 2026-11-24).** Mods are offered when the weapon carries the keyword in the mod's target list (`MNAM`; for the sword `ma_ChineseOfficerSword`, which the recipes' filters also use). A copy of a weapon keeps the `ma_` keyword, so the vanilla mods still appear in the same slot as yours. Swapping the `ma_` keyword on the copy and on your own mod's target list removed them: the weapon then **does not appear at the weapon workbench at all** (like glasses at the armor station) and the base weapon is unaffected. *(in game)*

## Vanilla melee numbers worth knowing *(xEdit)*

| Weapon | Damage | Weight | AP | Swing | Notes |
|---|---|---|---|---|---|
| Chinese Officer Sword | 16 | 3.0 | 35 | 1.12s | stagger small; Serrated mod +75% and bleed |
| Machete | 14 | 2.0 | 30 | 1.12s | |
| Combat Knife / Switchblade | 9 / 8 | 1.0 | 20 | 0.77s | Fast, short-blade keyword |
| Disciples Blade (Nuka-World) | 20 | 1.5 | 20 | 0.77s | Fast; Cutlass mod +75% and bleed (weight x2), Knife +25%, Hatchet +25%, Serrated Hatchet +75% |

At Strength 10 the displayed damage is double the record's base (16 shows 32).
