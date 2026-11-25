---
name: fo4-armor-and-clothing
description: Armor and clothing items - underarmor slots, ballistic weave, sneak effects, conditions on enchantment effects, neck items, dress texture wrapping.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the records. Source: the Regalia Garb, Veil and Eye specs.

## Slots and mods
- **Underarmor layout:** slots 33 and 36 to 40 (`First Person Flags` = 1992) and none of 41 to 45, as the Raider, Rustic, High School and Gunner underarmors (`BOD2 c8070000`). Chest and leg armor then go over it. *(in game)*
- Most vanilla dresses also occupy the [A] torso and leg slots, so armor cannot be worn with them. *(xEdit)*
- **Ballistic weave** is offered by keyword: a copy of the Sequin Dress inherits `ma_Railroad_ClothingArmor` `0020DE40` and is offered it. *(in game)*
- The Surgical Mask (`000787DA`) uses slots 48 and 49 only, so hats work with a copy of it. *(in game)*
- A `MISC` item cannot be worn. For a worn neck item, the Nuka-World Shock Collar (`06029AFA`) works as a base (it adds that DLC as a master); no base-game human neck item was found. *(in game for the collar)*
- A copy of armor with `EITM` delivers a constant enchantment, and so a perk, while worn. *(in game)*

## Stealth anchors *(xEdit)*
- Sneak rank 1: `Mod Detection Sneak Skill` Add +20 and `Mod Detection Movement` x0.9. The Sneak Skill value reads as the "harder to detect" percentage.
- Stealth Boy: +50 and x0.5. Sneak bobblehead: `Mod Detection Light` x0.9 and `Mod Detection Movement` x0.9.
- Ninja uses `Mod Sneak Attack Mult` Add (rank 1: +1 ranged, +0.5 melee). A copy changed sneak attack damage. *(in game)*
- Sneaking speed: `enchArmorStealthMoveMod` `00245BDE` (`Fortify Actor Speed Mult` 5, effect condition `IsSneaking`).
- Chameleon (`001F4D16`) uses `Stealth Field` `000976E0` magnitude 20, conditions `IsMoving` and `IsSneaking`. Shadowed: `00183313`.
- An enchantment's effects can carry conditions, so a second item can amplify through `WornHasKeyword`.

## Resist effects *(xEdit)*
`AbFortifyResistPoison` `0004D86B`, Energy `000559A4`, RadiationExpose `0004C47D`, RadiationIngest `0004A0C2`, Damage `0004A0AC`, Fire `0004D86E`, Cryo `0004D86D`, Electricity `0004D86F`. Damage types: `dtRadiationExposure` `00060A85`, `dtPoison` `00060A84`, `dtEnergy` `00060A81`.

## Recolouring clothing *(in game)*
- To learn how a texture wraps, paint a UV test texture (a hue per tenth of the width, fading down the height) and photograph it on mannequins. Regalia's `build/art/make_dress.py --debug-uv` does this for the Sequin Dress.
- Painting whole 4x4 blocks stair-steps at distant mip levels; blend and re-encode edge blocks.
- BC3 (the Surgical Mask): recolour the colour half of each block and keep the alpha half.
- Sequin Dress: `0011A27B`, addon `0011A27A`; the swap goes on the addon (`MO2S` male, `MO3S` female).
