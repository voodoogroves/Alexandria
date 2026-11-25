---
name: fo4-palette-and-glow
description: Palette (colour remapping index) recolours, emissive glow and real lights. Unverified community research.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
**Unverified:** community posts and wikis (mostly 2015 to 2019), nothing built or tested. For the material swap we did verify, see `fo4-materials-and-models`; a script can write a `.bgsm`, so no material editor is needed for that.

Not every material is palette based (the Chinese Officer Sword is not): check the `.bgsm` for a greyscale texture first.

## Palette recolouring

**Mechanism.** FO4 recolors weapons by swapping the `.bgsm` material. Many materials are *palette based*: a grayscale base texture plus a palette `.dds` (a strip of colors), where a **color remapping index** picks where on the strip to read. Like a Photoshop gradient map. Examples to inspect: baseball-bat paint OMODs use `PaintedBatLookup.dds`; hair uses `HairColor_LGrad_d.dds` (hardcoded, can't redirect).

**How the palette works (Kutsumiya, 2016, tested on combat armor; the best source so far).** The palette is a look-up table (LUT):
- **Horizontal axis = the diffuse map's gray value.** 128 px wide = gray 0 to 127 (256 px wide also works, as in Custom Combat and Synth Armor). Each part of the model (on combat armor: pouch, armor, belt, buttons) sits in its own gray range, so the palette recolors parts independently. For the sword, the blade, guard and grip will each sit in their own gray range; look at the diffuse map to see which.
- **Vertical axis = which palette row, chosen by the color remapping index** (a float, 0 to 1; the last row is 1). Vanilla is 64 rows with seven material "bands" stacked. Only **one pixel row** per band is actually read (the rest is padding; he expects even a 128x7 palette to work if the indices are calculated right, *unconfirmed*). Observed: index 0.18 = row 11, 0.33 = 21, 0.5 = 31, 0.66 = 42, 0.82 = 52, 1.0 = 63. That is roughly `row = index x (height - 1)`; so for our own palette, **pick the row, then compute the index**.
- **Index lives on the OMOD** (the `ColorRemappingIndex` property), which works. **An index set only inside a Material Swap did not work**: the game read row 0 instead (this is the "rusty Brotherhood combat armor" bug). Fix that worked: put the index on the OMOD too. Set the index on the OMOD, not only in the swap.
- **Palette file format: uncompressed 32-bit A8R8G8B8, no DXT/BC.** He says it will not work properly otherwise (reason unknown).
- Making a new palette is the recommended way for a new color; he warns that stretching the vanilla one means editing vanilla OMOD indices and conflicts easily.
- **NifSkope always shows row 0** (it can't compute the index), so its preview is wrong for palette materials; judge in game.
- Experiment pattern he used: paint one 1-px row a loud color, point the index at it, and see what turns that color (a good way to verify mapping). Related article he wrote, not read: a Minutemen blue combat-armor palette ("super easy").

**Also noted.** A forum author found each new color needed roughly 3 OMODs for weapon paints; that was for a multi-color variety mod, not a single fixed gold. Color index can also be overridden per placed reference (for statics).

**Not established.** Whether a swapped `.bgsm` needs its own palette or the original's persists *(one forum data point suggests the original's can persist)*. The index-to-row mapping is now answered above, but from one author's combat-armor tests.

## Glow

Two separate things; people conflate them.

**Surface glow (the sheen).** Set in the `.bgsm` (the NIF's own emissive values are **ignored** when the mesh points at a material file):
- enable Emittance; set the emittance color (gold) and multiplier (glow brightness). Start modest.
- Glowmap checkbox + a glow map restricts the glow to parts of the blade; without one the whole surface glows. Save glow maps DXT1 with mipmaps.
- Pitfalls: `bEnvironmentMapping` on can make the glow map ignored (turn it off); visual distortion with a bgsm plus glow map was fixed by disabling the vanilla normal map; a NIF with a TextureSet plus an injected bgsm can fail silently; editing a shared vanilla `.bgsm` affects every mesh using it (so copy it).
- A glow does **not** light surroundings; it only haloes the mesh.

**Real light (optional, and the more fiddly).** It is **not** the NiPointLight in the NIF (people edited those in 2016 without effect). *(One 2017 source, unverified:)* the NIF carries an **AddOnNode** with an index; that index points to an AddOnNode record in the Creation Kit (Special Effects > AddOnNode), which points to a **Light** record (World Object > Light) where color and intensity are set. Use a vanilla light or make a gold one. Risks: the index is shared across mods and a clash causes lighting conflicts (a community list of claimed indexes: https://www.nexusmods.com/fallout4/mods/16497/); and the mesh needs the node.

**Effect mesh (a halo or aura).** A separate shape with a BSEffectShaderProperty and a `.bgem` material, as plasma and fire weapons do. Reference mods to open in NifSkope: a Plasma Sword mod, a glowing power-fist mod, a Shishkebab glow thread. Not researched further; only if the sheen isn't enough.

## Sources

Community, mostly 2015 to 2019; verify against the current game.
- Creation Kit wiki: [Material Swap](https://falloutck.uesp.net/w/index.php?title=Material_Swap&mobileaction=toggle_view_desktop), [Object Mod](https://falloutck.uesp.net/wiki/Object_Mod) (`fColorRemappingIndex`, `pwMaterialSwaps`).
- [Simple material swap guide](https://chiaroscorner.wordpress.com/2018/10/01/how-to-do-a-simple-material-swap-in-the-fallout4-creation-kit/) and [FO4Edit material swap tutorial](https://www.modsfallout4.com/material-swap-mod-tutorial-fo4edit-tips-by-dm/).
- [fo4docs: Textures and Materials](https://www.fo4docs.org/File%20Formats/Assets/Textures%20and%20Materials/) and [Bethesda Modding Library](https://baddogskyrim.github.io/BethesdaLibrary/file-formats/textures-materials/) (emittance fields, NIF values overridden by material).
- Kutsumiya, Fallout Sandbox DIY (read via Chris's translated PDFs): [Investigating the structure of the color palette](http://f4mod.hatenablog.com/entry/textures/Color_Palette_1) and [Changing the palette and correcting the BoS combat armor](http://f4mod.hatenablog.com/entry/textures/Color_Palette_2), both 2016-09-24.
- Nexus threads (read via pasted text): [Changing weapon luminance/glow](https://forums.nexusmods.com/topic/3562285-changing-weapon-luminanceglow) (2015 to 2017), [Material swaps and grayscale](https://forums.nexusmods.com/topic/7273116-question-regarding-material-swaps-and-grayscale) (2018), [Color remapping index](https://forums.nexusmods.com/topic/4839060-color-remapping-index) (2016).
- Search snippets only (pages not read): [Shishkebab glow](https://forums.nexusmods.com/topic/3915231-shishkebab-glow-color-assistance), [Glow maps don't work](https://forums.nexusmods.com/topic/9766568-glow-maps-dont-work), [Retexture of weapons and material editing](https://www.nexusmods.com/fallout4/mods/27574).
