---
name: fo4-hazards-and-disease
description: Poison gas immunity, disease resistance (none exists), Survival disease risk and herbals, persuade perk entry points.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
## Poison gas immunity (read in xEdit; **not yet tested in game**)
The Hazmat Suit blocks the two vanilla poison gases through a perk, not through poison resistance:
- `ARMO Armor_HazmatSuit` `000CEAC4` has `EITM` `ENCH EnchPA_HazmatRadReduction` `0014EF47` (constant effect) -> `MGEF HazmatSuitPerkEffect` `0014EF48` (archetype Value Modifier on `RadResistExposure`, magnitude 0, with a **Perk to Apply**) -> `PERK HazmatSuitRadResistancePerk` `0014EF46` (hidden, one rank).
- The perk has three entries: (1) **`Mod Incoming Spell Magnitude`, `Set Value` 0.0, with the perk condition `EPMagic_SpellHasKeyword DN102_PoisonGasEffectKeyword` (`KYWD 000449B9`) == 1** (tab 1): this blocks the gas; (2) `Mod Incoming Spell Magnitude`, `Multiply Value` 0.02 on spells with `DamageTypeRadiation` (`0004B25C`); (3) `Mod Typed Incoming Spell Magnitude`, `Multiply Value` 0.02 for `dtRadiationExposure`. Entries 2 and 3 are the radiation reduction.
- **Exactly three effects carry the gas keyword** in the whole game (all DLC included, no Creation Club): `DN102_GasHazardEffect` "HalluciGen Gas" `000E6203`, and the Nuka-World `DLC04Gauntlet_GasHazardInitialEffect` `0602D828` and `DLC04Gauntlet_GasHazardSubsequentEffect` `0603DB50` (they use `PoisonResist` and hit `Health`). So the perk blocks those and nothing else.
- Poison resistance only softens the gas; it does not zero it. Power armor and gas masks lack this perk (matching reports that they do not block it).

To copy the immunity, copy entry (1) only into your own perk. Test sites: HalluciGen, Inc. basement (quest Hazardous Material) and the Nuka-World Gauntlet gas room (quest Taken for a Ride).

## Disease *(xEdit and script reading)*
- There is **no disease resistance stat**: `Resist Disease` is in the engine definitions but has no record. Poison resistance is the simple stand-in.
- No script-free way to switch disease off: the global `HC_Rule_DiseaseEffects` cannot be set from a record.
- Why a worn item cannot hold the herbal effects: they are Fire and Forget with a 1 hour duration *(casting type unverified)*, and the manager script tests those exact records, so copies do not count. A script would be needed.
- Herbal to disease: Anodyne counters Insomnia and Weakness; Stimulant counters Lethargy and Fatigue; Antimicrobial counters Hunger ("Parasites") and Infection. Read from `HC_Manager`'s `DiseaseEffects` property.
- Mechanism (script `hardcore/hc_managerscript.pex`): risky actions add to a risk pool (standard food or chem +0.07, high-risk food +0.12, cannibalism or faction hit +0.05, rain or swimming +0.03, cola +0.02, immunodeficiency +0.05); the pool drains 0.01 per 8 game hours; rolls start at 0.25; 1 game day grace after catching one. Per the script's text, a disease needs `herbal threshold (0.2) <= roll <= pool` while a herbal is active. If the roll is uniform *(unverified)*, that cuts the chance about 40 to 57% at pools of 0.35 to 0.5 and removes it at 0.2 or below. Hunger and thirst states clamp the pool (floors 0.05 to 0.3+), so a starving player is hard to protect.
- Cures and sources: Antibiotics (craft at the Chemistry Station with Chemist rank 1), the three craftable herbals (Survival only). Lead Belly affects radiation only.
- Check in game, in game: `player.hasmagiceffect 00249F9A` (and `9B`, `9C`) *(command unverified)*.

**How it was read:** `esmdump` and `scripts/fo76utils/esmrec.py` for records; the script's variable defaults from the compiled `.pex` (little-endian; string table then variable entries); the quest's properties by parsing its `VMAD` blob. The `VMAD` parser was a throwaway and was not kept.

## Persuade and intimidate *(xEdit)*
`Mod Charisma Challenge Chance` Add Value is persuade (Black Widow, Lady Killer: +10/+20/+30). `Mod Player Intimidation` exists but no vanilla perk uses it.
