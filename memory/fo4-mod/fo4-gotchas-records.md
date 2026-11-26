---
name: fo4-gotchas-records
description: Engine behaviour that looks like it should work and does not. Read before designing records.
applies-to: [fo4-mod]
kind: gotcha
verified: 2026-11-26
---

# Record gotchas

Detail: `fo4-perks-and-damage`, `fo4-weapon-mods-and-names`, `fo4-guns-sound-and-impact`, `fo4-materials-and-models`, `fo4-armor-and-clothing`.

- Perk without script: constant `ENCH` -> Add Perk `MGEF` -> `PERK`, attached by an OMOD `Enchantments ADD` (or armor `EITM`).
- A weapon record's own `EITM` and template `ActorValues` did not apply. *(in game)*
- A legendary mod in a sword's template did not attach; on the .44, including one in **every** combination worked. *(in game)*
- `Speed` does nothing on a melee weapon; swing speed is on the weapon record. *(in game)*
- OMOD numeric `MUL+ADD` values add into one multiplier: edit the existing entry.
- Item name words come from keywords on the weapon and its mods. A full custom name needs its own weapon record.
- A recolour is a `MSWP` applied with `MaterialSwaps` **ADD**. It covers only the model that mod supplies, and only materials the mesh references.
- An explosion fires only from a projectile with the `Explosion` flag; ordinary bullets lack it.
- Fields under `FNAM` (e.g. `Override Projectile`) are not under `DNAM`.
- The craft screen shows only **two** perk prerequisites; use at most two per recipe. `HasPerk` greys a recipe, it does not hide it. See `fo4-recipes-and-gates`.
- A condition list cannot be added to a spell effect with `Add`; put the condition on the perk entry.
- A recipe ingredient should be a plain item: crafting from a wearable one crashed once (cause unproven) and crafting from a worn one left its look on the player.
- An explosion's own `Stagger` at 0 damage did not affect bystanders; give it a conditional enchantment. See `fo4-guns-sound-and-impact`.
- A crafted weapon with a legendary mod: remove the mod copy's `FeaturedItem` keyword, or its pop-up over the craft menu can crash the game.
- Treat record names from generated or brainstormed text as unverified until read in xEdit.
