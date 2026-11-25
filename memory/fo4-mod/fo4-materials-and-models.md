---
name: fo4-materials-and-models
description: Recolouring with material swaps and new textures, loose files, asset paths, armor addon models, biped slots.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Recolouring a weapon: material swaps and new materials (verified 2026-11-24)

- **A recolour is a material swap.** A `MSWP` record maps an *original* `.bgsm` to a *replacement* (`Material Substitutions / Substitution`: `BNAM` original, `SNAM` replacement, both paths relative to `Materials\`, backslashes). A weapon mod applies it with the property `MaterialSwaps` (`FormID,Int`, `ADD`, Value 1 = the `MSWP`, Value 2 = 441, as vanilla mods use). It applies to the model that mod supplies (here the blade), not to the weapon record's own model (the hilt). *(in game)*
- **A test that cannot be subtle:** point the swap at a very different vanilla material (`Weapons\RocketHammer\Mod_Stunpack.BGSM`): a pattern across the blade proves the mechanism works. Swapping to a material that differs only in the cubemap (`ChineseOfficerSword01B`, bronze reflections) was invisible. *(in game)*
- **The sword's materials are not palette based** (no greyscale texture), so the colour-remapping route does not apply; real gold needs a new diffuse texture and a new `.bgsm`. A `.bgsm` is a small binary (about 450 bytes): length-prefixed strings (u32 length including the NUL) for the texture paths, the cubemap and the template; copy an existing one and patch the diffuse path. *(worked)*
- **Recolouring a BC1 texture without decoding it:** a BC1 block is 8 bytes (two RGB565 endpoints, 16 two-bit indices), so recolouring the two endpoints of every block, across all mip levels, recolours the picture. Keep the block mode: `c0 > c1` is four-colour, `c0 <= c1` is three-colour; if the order flips after recolouring, swap the endpoints and permute the indices (0 and 1 swap, 2 and 3 swap for four-colour). Pure Python (standard library only) did it for a 512 x 1024 texture in seconds; decoding mip 0 to a PNG lets you check the colours without the game. *(worked in game: gold with the original pitting)*
- **Reading the vanilla archives:** `baunpack` (fo76utils) lists and extracts materials and textures (`fo4-archives`).
- **Loose files (MO2):** vanilla `Fallout4.ini` has `sResourceDataDirsFinal=STRINGS\`, which keeps loose textures and materials from loading. Put `[Archive] bInvalidateOlderFiles=1` and an empty `sResourceDataDirsFinal=` in the profile's `fallout4custom.ini` (the profile must use its own ini files: `LocalSettings=true`). For release, pack into a `.ba2` named after the plugin instead. *(loose files loaded in game with these settings)*

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
