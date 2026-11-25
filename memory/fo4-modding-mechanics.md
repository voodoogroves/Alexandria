# Fallout 4 modding mechanics (project independent)

How things work in the game data, as verified on the Steam Deck (Proton 11.0, FO4Edit 4.1.5f, game 1.11.240). Facts marked *(in game)* were tested by playing; *(xEdit)* were read from the vanilla records; *(inferred)* are conclusions not yet tested. Project-specific results (the Blade of Bastet) are in `fo4-regalia-of-mystery/spec/`; xEdit scripting facts are in `fo4-xedit-reference.md`. Dated 2026-11-24.

## Giving the player a perk from an equipped item, with no script

- Chain: `ENCH` (Constant Effect, target Self) -> `MGEF` (archetype "Add Perk" shown as *Unknown 36*, constant, self, hidden; the perk is in `Magic Effect Data / DATA - Data / Perk to Apply`) -> `PERK`. Vanilla's penetration chain is `001F4425 -> 001F4423 -> 001F4166`. Copy and re-point: the ENCH's effect is `Effects / Effect / EFID - Base Effect`. *(xEdit; copies verified in game: `hasperk` is 1 equipped, 0 unequipped)*
- It attaches through an **OMOD** property `Enchantments ADD <ENCH>`, on a mod in an Upgrade slot (`ap_melee_MeleeMod`). A weapon record's own `EITM`, and weapon-level `ActorValues` stats in the object template, did **not** apply anything. *(in game)*
- Console check: `player.hasperk <ID>`. Note a perk added by the console (`player.addperk <ID>`) stays; the chain adds and removes it with the item.

## Perk entries that change damage

- Entry point **Mod Target Damage Resistance**, function **Multiply Value** (or "Multiply 1 + Actor Value Mult" with an actor value): the target's resistance is multiplied. 0.70 ignores 30%, 0.0 ignores all. *(in game)*
- A perk entry has condition **tabs**: 0 = attacker (the player), 1 = weapon, 2 = target. `HasKeyword` on the weapon tab matches a keyword on the equipped weapon (including keywords an OMOD added). `WornHasKeyword` on the attacker tab matches any worn item with the keyword. Both verified. *(in game)*
- **Several entries multiply.** A second entry with multiplier 0.0 and an extra condition zeroes the product, so a "bonus" item can be a bare keyword on a worn item; no script and no per-item perk. *(in game)*
- Vanilla non-legendary melee penetration is one entry in the base `PlayerPerk` (`00056DD7`): 0.85 when the weapon has `ap_melee_ArmorPenetrate`. Copying `PlayerPerk` and keeping one effect makes a clean starting point. *(xEdit)*

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
- A rule can only add words around the base name. A full custom name needs a weapon record of its own, or accepting the base name with a prefix or suffix. Overriding the shared naming record is a mod-compatibility cost. *(inferred from the data; the suffix test is the next in-game check)*
- Vanilla melee name words include Serrated, Electrified, Sharp, Bladed, Heavy, Weighted, Spiked, Heated, and colours (Black, Red, Oak, Mahogany, Grey, Blue, Aluminum, Cedar). There is no gold word. *(xEdit)*

- **Verified 2026-11-24:** a naming rule's text lands **in front** of the base name even at index 10001 (the base `*` is 10000). So rules cannot add a suffix, and ordering is not by index alone. **The way to a custom full name is a weapon record of its own**: copy the base weapon, give it its own `FULL`, and keep name keywords off its mods (no Serrated), so no rule adds words. *(in game)*
- **A custom weapon can carry its mod through its template**: replace the `Mod` of every template include (`Object Template / Combinations / <combination> / OBTS / Includes / Include / Mod`) that was the base's "No Upgrade" with your own Upgrade-slot mod. It attached on a crafted item (damage, weight, VATS, perk all applied). A *legendary* mod in a template include did not. *(in game)*
- **Workbench mod lists are filtered by keyword (verified 2026-11-24).** Mods are offered when the weapon carries the keyword in the mod's target list (`MNAM`; for the sword `ma_ChineseOfficerSword`, which the recipes' filters also use). A copy of a weapon keeps the `ma_` keyword, so the vanilla mods still appear in the same slot as yours. Swapping the `ma_` keyword on the copy and on your own mod's target list removed them: the weapon then **does not appear at the weapon workbench at all** (like glasses at the armor station) and the base weapon is unaffected. *(in game)*

## Recolouring a weapon: material swaps and new materials (verified 2026-11-24)

- **A recolour is a material swap.** A `MSWP` record maps an *original* `.bgsm` to a *replacement* (`Material Substitutions / Substitution`: `BNAM` original, `SNAM` replacement, both paths relative to `Materials\`, backslashes). A weapon mod applies it with the property `MaterialSwaps` (`FormID,Int`, `ADD`, Value 1 = the `MSWP`, Value 2 = 441, as vanilla mods use). It applies to the model that mod supplies (here the blade), not to the weapon record's own model (the hilt). *(in game)*
- **A test that cannot be subtle:** point the swap at a very different vanilla material (`Weapons\RocketHammer\Mod_Stunpack.BGSM`): a pattern across the blade proves the mechanism works. Swapping to a material that differs only in the cubemap (`ChineseOfficerSword01B`, bronze reflections) was invisible. *(in game)*
- **The sword's materials are not palette based** (no greyscale texture), so the colour-remapping route does not apply; real gold needs a new diffuse texture and a new `.bgsm`. A `.bgsm` is a small binary (about 450 bytes): length-prefixed strings (u32 length including the NUL) for the texture paths, the cubemap and the template; copy an existing one and patch the diffuse path. *(worked)*
- **Recolouring a BC1 texture without decoding it:** a BC1 block is 8 bytes (two RGB565 endpoints, 16 two-bit indices), so recolouring the two endpoints of every block, across all mip levels, recolours the picture. Keep the block mode: `c0 > c1` is four-colour, `c0 <= c1` is three-colour; if the order flips after recolouring, swap the endpoints and permute the indices (0 and 1 swap, 2 and 3 swap for four-colour). Pure Python (standard library only) did it for a 512 x 1024 texture in seconds; decoding mip 0 to a PNG lets you check the colours without the game. *(worked in game: gold with the original pitting)*
- **Reading the vanilla archives:** `baunpack` (fo76utils) lists and extracts materials and textures (`Rhodes/refs/fo4-archives.md`).
- **Loose files (MO2):** vanilla `Fallout4.ini` has `sResourceDataDirsFinal=STRINGS\`, which keeps loose textures and materials from loading. Put `[Archive] bInvalidateOlderFiles=1` and an empty `sResourceDataDirsFinal=` in the profile's `fallout4custom.ini` (the profile must use its own ini files: `LocalSettings=true`). For release, pack into a `.ba2` named after the plugin instead. *(loose files loaded in game with these settings)*

## Vanilla melee numbers worth knowing *(xEdit)*

| Weapon | Damage | Weight | AP | Swing | Notes |
|---|---|---|---|---|---|
| Chinese Officer Sword | 16 | 3.0 | 35 | 1.12s | stagger small; Serrated mod +75% and bleed |
| Machete | 14 | 2.0 | 30 | 1.12s | |
| Combat Knife / Switchblade | 9 / 8 | 1.0 | 20 | 0.77s | Fast, short-blade keyword |
| Disciples Blade (Nuka-World) | 20 | 1.5 | 20 | 0.77s | Fast; Cutlass mod +75% and bleed (weight x2), Knife +25%, Hatchet +25%, Serrated Hatchet +75% |

At Strength 10 the displayed damage is double the record's base (16 shows 32).

## Guns: sound, silencing, impact explosions, on-hit effects (2026-11-25, *in game* unless marked)

- **Gunshot sounds** come from `KSSM` (sound keyword mapping) records matched on the weapon's `s_` keywords (e.g. `s_40_PistolFortyFour` + `s_10_Player`); a suppressor adds `s_20_Silenced`. A weapon with no silenced mapping still sounds loud: copy another weapon's silenced mapping and swap its `s_40_` keyword.
- **Silent to NPCs** is the weapon's `DNAM` Sound Level = Silent; it does not change the audible sound.
- **Explosion on impact** needs a projectile with the `Explosion` flag. Bullets (`Projectile45Caliber`) are Beam type without it, and naming an explosion on a copy did nothing. A copy of `MS09PushProjectile` `00199046` (Missile) works. The override projectile is `FNAM` `Override Projectile` on the weapon.
- **Legendary effects** can be built into a weapon by including the legendary mod in every object template combination (Staggering `001E81AB`); the "legendary item" pickup screen appears. Console `amod <modID>` on a dropped item tests one without a build. A weapon record's `EITM` stagger enchantment never applied.
- **Chance-based on-hit effect from a perk:** an `Apply Combat Hit Spell` entry (copy from `BigLeagues02` `000E36FC`), attacker-tab conditions (`WornHasKeyword`, `GetRandomPercent`), weapon-tab `HasKeyword`, spell copied from `PerkDisarm` `001D604B`.
- **Recolouring BC1/BC3 textures** per block, per mip, with plain Python works for weapons and clothing; blend and re-encode only edge blocks for smooth shapes.

## Testing

- The SKK workshop target dummy shows damage numbers, never goes hostile, and takes any armor pieces: damage unarmored vs in the heavy combat chest shows the armor effect directly.
- Difficulty to Normal and fixed; the in-game console accepts a paste.
