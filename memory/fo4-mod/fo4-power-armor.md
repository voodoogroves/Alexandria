---
name: fo4-power-armor
description: Power armor modding - the race and skeleton, extra wearables on a suit (ArmorTypePower), arm mod connect points, body slots in use.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-26
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

## Records and models *(read)*
- `PowerArmorRace` has its own skeleton, `actors\PowerArmor\CharacterAssets\skeleton.nif`.
- Slots used by power armor pieces: 30-39, 41-50, 52, 60. No armor of any race used 53, 56 or 57 (base game, DLC, Creation Club and the few mods present).
- Each base game left arm mod model has parent points `p-Misc`, `p-Paint`, `p-Lining` and attaches by `C-LArmMod`; none has a forearm accessory point.
- Reading and previewing these models: `scripts/nif/` in Rhodes.
