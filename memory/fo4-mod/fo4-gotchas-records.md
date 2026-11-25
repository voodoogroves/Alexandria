---
name: fo4-gotchas-records
description: Engine behaviour that looks like it should work and does not. Read before designing records.
applies-to: [fo4-mod]
kind: gotcha
verified: 2026-11-25
---

# Record gotchas

Detail: `fo4-perks-and-damage`, `fo4-weapon-mods-and-names`, `fo4-guns-sound-and-impact`, `fo4-materials-and-models`.

- Perk without script: constant `ENCH` -> Add Perk `MGEF` -> `PERK`, attached by an OMOD `Enchantments ADD` (or armor `EITM`).
- A weapon record's own `EITM` and template `ActorValues` did not apply. *(in game)*
- A legendary mod in a sword's template did not attach; on the .44, including one in **every** combination worked. *(in game)*
- `Speed` does nothing on a melee weapon; swing speed is on the weapon record. *(in game)*
- OMOD numeric `MUL+ADD` values add into one multiplier: edit the existing entry.
- Item name words come from keywords on the weapon and its mods. A full custom name needs its own weapon record.
- A recolour is a `MSWP` applied with `MaterialSwaps` **ADD**. It covers only the model that mod supplies, and only materials the mesh references.
- An explosion fires only from a projectile with the `Explosion` flag; ordinary bullets lack it.
- Fields under `FNAM` (e.g. `Override Projectile`) are not under `DNAM`.
- Treat record names from generated or brainstormed text as unverified until read in xEdit.
