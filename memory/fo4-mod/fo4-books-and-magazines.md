---
name: fo4-books-and-magazines
description: How vanilla magazines and notes are built (meshes, cover swaps, inventory art, perk link) and what a new one needs.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: **Verified** = read from the records or files; **decoded** = a reading of a binary layout; **unverified** = not checked. Tools: `fo4-archives`. Granting a perk on read was later verified in game: `fo4-recipes-and-gates`.

## Is a new magazine just cover art?

**No. Cover art is the cheap part, about a third of the job.** A collectible with no perk is little more than art plus one record. A magazine that teaches a perk also needs a PERK record.

## How a vanilla perk magazine is built (Grognak the Barbarian, 10 issues)

- **One shared mesh per cover shape.** The ten Grognak issues use only four meshes (`Props\GrognakComic\Comic_Grognak{Jan,Feb,Mar,Apr}.nif`). Each mesh has three shapes with three materials: front cover (e.g. `Materials\Props\ComicsAndMagazines\Grognak\GrognakJan.BGSM`), back cover (`...\Backside\ComicBackGray.BGSM`) and paper edge (`Materials\Props\Grognak\Paper.BGSM`), plus Havok collision. **Verified.**
- **The cover art is swapped per issue with a Material Swap (MSWP)**, named on the BOOK's `MODS` field (e.g. `CustomMaterialSwap0008E74A` swaps `grognakjan.bgsm` to `grognakdec.bgsm`; others also swap the back cover color). **Verified.**
- **A front cover is a diffuse plus a tiny material.** `GrognakJan.bgsm` (about 385 bytes) points at `Props/ComicsAndMagazines/Grognak/GrognakJan_d.dds` plus the shared `ComicNormalSpec_n/_s.dds`. The diffuses are about 174 KB (roughly 512x512 DXT1 with mips; inferred from file size, header not read). **Verified (paths and sizes).**
- **A second set of art serves the inventory (Pip-Boy) preview.** The BOOK's `INAM` points at a STAT (`HighPoly_Grognak10`, `001D1C33`), whose model is the shared `Interface\ComicsAndMagazines\ComicHighRes01.nif` and which has **its own `MODS` swap**. Its materials and textures live under `ComicsAndMagazinesHighRes` and the covers are about 699 KB each (roughly 1024x1024; inferred). **Verified.**
- **The BOOK record's fields** (Grognak issue 10, `0008E74A`): `EDID`, `VMAD` (a Papyrus script `CA_SkillMagazineScript` with a property named `CA_Event_ReadSkillBook`; looks like achievement tracking, **purpose unverified**), `OBND`, `PTRN`, `FULL` (name, a localized string ID), `MODL` (the world mesh), `MODS` (swap), `KWDA` (keywords `FeaturedItem` `001B3FAC` and `PerkMagKeyword` `001D4A70`), `FIMD` (featured-item message), `DATA` (value 100, weight 0), `DNAM`, `INAM`. **Verified.**
- **`DNAM` holds the perk link (decoded).** 13 bytes: a flag byte `0x10` ("add perk", plausible) then the form ID `0x0008E3A6`, which is the PERK `PerkMagGrognakTheBarbarian`. The name matching is strong evidence. All ten issues carry the same `DNAM`, so they point at one perk. How ranks stack across issues is **unverified**. A plain note (`DN036MinutemenNote`) has all-zero `DNAM`, which supports the decoding.
- **Plain readable notes** use a different mesh (`Props\NoteRipped_LowPoly.nif`) and a text `DESC`; magazines have an empty `DESC`.

## What a new magazine needs

| Piece | Needed for | Work |
|---|---|---|
| `BOOK` record (copy a vanilla one: name, keywords, value, weight, model, `MODS`, `INAM`) | always | xEdit |
| World cover: MSWP + a front `.bgsm` + a diffuse `_d.dds` (about 512x512) | always | texture + tiny `.bgsm` + xEdit |
| Inventory cover: a new STAT (own `MODS`) + MSWP + hi-res `.bgsm` + diffuse (about 1024x1024) | to look right in the Pip-Boy; maybe optional (unverified) | texture + tiny `.bgsm` + xEdit |
| `PERK` record, referenced from `DNAM` | only if it teaches a perk | xEdit (perk effects are the real work) |
| `PerkMagKeyword` | shows with the vanilla magazines; effect on the magazine rack display **unverified** | xEdit |

**Cheapest version** (a lore collectible, no perk): reuse a vanilla world mesh and the vanilla hi-res mesh; the work is two textures, two `.bgsm` files, two swaps and one BOOK record. Everything except the textures and `.bgsm` files is xEdit.

## Writing the materials
Both `.bgsm` files can be written by script (`fo4-materials-and-models`). Use our own paths and a swap; never overwrite vanilla paths.

## Not checked

How perk ranks stack across issues; whether `CA_SkillMagazineScript` is needed for the perk to be granted (the `DNAM` flag probably does it); whether the magazine rack workshop item filters on `PerkMagKeyword`; whether `INAM` art is required; DDS header sizes (sizes above are inferred from file sizes).
