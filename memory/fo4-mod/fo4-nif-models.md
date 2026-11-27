---
name: fo4-nif-models
description: Writing or rebuilding FO4 .nif models with your own scripts (skinned armor pieces from vanilla shapes), and finding why a model crashes the game.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-27
---
Tags: *(in game)* tested by playing; *(read)* seen in the files; *(upstream)* from outside docs, not checked here. Game 1.11.240, model stream version 130. Writers and preview tools: Rhodes `scripts/nif/`.

## String references: the random crash

- A model has one string table in its header; blocks refer to names by **index**. If you rebuild the table, rewrite **every** index in **every** block you copied. *(read)*
- A `BSLightingShaderProperty` has **two**: the material name at byte offset 4 and the **root material** (the `.bgsm` template, easy to miss) at offset 60. Offsets read in vanilla weapon and power armor shaders with no extra data. *(read)*
- An index past the end **crashes the game while loading the model, only sometimes**: the same file can load once and crash the next time. *(in game, 2026-11-26/27)*
- Crash signature: access violation in `BSFixedString::operator=`, under `NiStream::LoadFixedString` and `BSLightingShaderProperty::LoadBinary`; the report names the model file and material. *(in game, crash logger)*
- Check on read-back: every string index is -1 or inside the table, and names the same string as in the source. Offline renderers and size checks do not catch this. *(read)*

## Finding a crash in your own model

1. **Install a crash logger first** (`fo4-testing`). It named the file and the loading step on the first try. Seven rounds of swapping model parts before that did not find it. *(in game)*
2. With a crash that comes and goes, **"it did not crash" proves little.** A part carrying the same fault survived and pointed the search the wrong way. *(in game)*
3. Comparing against a model that works helps, but compare **what each reference points to**, not only block layout and sizes.

## Skinned shape from a static weapon shape

- A worn piece: `BSSubIndexTriShape` + `BSSkin::Instance` + `BSSkin::BoneData`. Layout and placement that loaded in game: `fo4-power-armor`. *(in game)*
- Vertex layout word: static `0x0001B00000430205` (20 bytes: position, UV, normal, tangent) becomes skinned `0x0005B00050430208` (32 bytes: adds four half-float weights and four bone indices). Other layouts exist. *(read)*
- Set the skinned bit `0x2` in the shader's first flag word (offset 16). *(read; that it is required is untested)*
- The shader and its `BSShaderTextureSet` can be copied from the source model; repoint the texture set block reference (offset 40) and both strings. *(read)*

## Format reference

- Field-by-field definitions of `.nif` blocks: `nif.xml` from the NifTools project (github.com/niftools/nifxml), which NifSkope reads. *(upstream)* Check it before working offsets out from bytes.
