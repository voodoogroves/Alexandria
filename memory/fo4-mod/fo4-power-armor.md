---
name: fo4-power-armor
description: Power armor modding - the race and skeleton, extra wearables on a suit (ArmorTypePower), arm mod connect points, body slots in use, how backpack mods attach.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-27
---
Tags: *(in game)* tested by playing; *(read)* read from plugin or model files; *(inferred)* not tested. Source: PAX.

## Wearing an extra item in power armor
- **Plain apparel is refused** in the suit ("you can not equip that in power armor"), even with an armor addon that lists `PowerArmorRace`. *(in game)*
- **A piece with keyword `ArmorTypePower` `0004D8A1` and health 100 equips and shows** on the suit: a new `ARMO` + `ARMA` (race Human, additional race `PowerArmorRace` `0001D31E`, a free body slot), pointing at a vanilla skinned model. *(in game)* Which of the keyword or health is needed was not separated.
- Vanilla skinned models work as the worn model: `SMArmor0_Arms.nif` (super mutant bones the suit skeleton shares) and a power armor arm mod model, `PA_X1_LArm.nif`. *(in game)*
- On exiting, the extra piece stays with the frame (listed in the frame's menu with the other pieces, not moved to the player) but **is not drawn on the empty suit**. Cause unknown. Getting back in equips and shows it again. *(in game)*
- Wearing pieces in slots 56 and 57 displaced none of the five suit pieces. *(in game)* Slots 56 and 57 are the ones tested. Not yet tested: save and reload, workbench behaviour, damage to the piece.

## Our own model on a suit
- A model written by our own script (skinned `BSSubIndexTriShape`, one bone `LArm_ForeArm1`, identity skin-to-bone transform, vertices in bone coordinates, all triangles in the fifth of five segments) loaded and sat where the offline preview put it. *(in game, 2026-11-26)* Writer: `scripts/nif/writeshield.py` in Rhodes.
- A part cut from a larger armor model is open where it met the rest: it is **see-through from that side** until back faces are added; a second set of triangles with reversed winding and normals, just behind the first, made it solid. *(in game)*

## Fist weapons on a suit
- The "WaFists" mod makes fist weapons usable in power armor with new `WEAP` records whose `DNAM` animation type (byte 54) is 1 (one-handed sword), not 0 (hand to hand), keeping the unarmed keywords; its Deathclaw Gauntlet differs from vanilla in only that byte. *(read; that this byte is the cause is inferred)*

## Backpack mods and slots (plugins read 2026-11-27; none run in game)

Most suit backpacks are **torso modifications**, which use no body slot. To stay clear of them, leave **54** free. *(read)*

| Mod (Nexus ID) | In the suit | Built as | Body slot in the suit |
|---|---|---|---|
| Improvised Backpack (50101) | yes | torso mod on `ap_PowerArmor_Misc`, the jetpack's point | none |
| Dynamic Power Armor 2024 (75900) | yes | torso mod on its own new attach point; overrides standard torso linings; needs Automatron | none |
| Immersive Backpacks (39300) | yes | script swaps in a backpack version of the torso piece; its slot 54 item is for outside the suit | none extra |
| Wearable Backpacks and Pouches (3258) | main file: no `ArmorTypePower` | worn items on 54, 55, 56 or 57 | none in the suit |
| Its "Power Armor Carry Capacity" add-on | keyword present | separate power armor item | 54 |

- None ships an F4SE plugin. A torso addition of our own as a separate worn piece would not compete for these attach points. *(inferred)*

## Records and models *(read)*
- `PowerArmorRace` has its own skeleton, `actors\PowerArmor\CharacterAssets\skeleton.nif`.
- Slots used by power armor pieces: 30-39, 41-50, 52, 60.
- Slots 53 to 58 in the base game, DLC, Creation Club and the few mods present (2026-11-27): **no wearable item on 53, 56 or 57**, but naked body armor addons list **33, 52, 53**. 55 has a DLC face mask; 54 and 58 have bags in one installed mod.
- A piece of ours on slot 53 equipped and showed on the suit. *(in game)*
- Each base game left arm mod model has parent points `p-Misc`, `p-Paint`, `p-Lining` and attaches by `C-LArmMod`; none has a forearm accessory point.
- Reading and previewing these models: `scripts/nif/` in Rhodes.
