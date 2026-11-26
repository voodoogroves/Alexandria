---
name: fo4-world-placement
description: Placing things in the world with xEdit scripts - dead actors and poses, cell and worldspace overrides, containers that do not refill, loose items.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-26
---
Tags: *(in game)* seen by playing; *(plugin files)* read from saved plugins. Working script: `fo4-regalia-of-mystery/build/xedit/BuildWorld.pas`. Get coordinates with `player.getpos x`, `y`, `z`.

## Adding references to a cell
- Override the cell record (`wbCopyElementToFile(cell, file, False, True)`; its references are not copied), then `Add(cell, 'REFR' or 'ACHR', True)` and set `NAME`, `DATA` position and rotation. Worked in exterior and interior cells; scenery looked normal and the previs data matched vanilla. *(in game, plugin files)*
- An exterior cell brings an override of the worldspace record. xEdit's copy drops `OFST`, `RNAM` and `CLSZ`; no problem seen. It is the `Fallout4.esm` version, so check what DLC and Creation Club files change in it (2026-11-26: only those blocks and an icon path). *(plugin files)*
- Check that no other loaded plugin overrides the same cells.
- `No Respawn` on a placed object is record flag `0x40000000`. Behaviour after an area reset was not yet checked.

## A dead body
- Base: a copy of `Loot_CorpseSettlerFemale_Empty` `0020A578` (no respawn). Clear "Use template: base data" (`ACBS` template flag bit 7), or it takes the template's name. *(in game)*
- The placed `ACHR` needs **Starts Dead** (`0x200`) and a saved pose: copy `XRGD` and `XRGB` from a vanilla placement (e.g. `ACHR` `001289CF`). Without them it stood upright in a T-pose. *(in game)*
- Clothing it should wear but not give as loot: a copy flagged **Non-Playable** (record flag `0x4`) in its outfit (`OTFT`).

## A container with one item, once
- Our own copy of a vanilla container: clear the Respawns flag (`DATA` bit `0x02`), remove `VMAD` and the vanilla items, add the item. *(in game; refill after a reset not yet checked)*
- Add our own container rather than editing a shared base record.
- Some meshes name no material to swap (`Suitcase01.nif`); the prewar suitcase mesh does.
