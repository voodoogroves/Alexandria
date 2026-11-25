---
name: fo4-recipes-and-gates
description: Crafting recipes and gating - station and category, HasPerk locks, two visible prerequisites, a book that grants a perk, unique and broken items.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-24
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the records. Source: the Regalia Eye spec and build spec.

## Recipes (`COBJ`)
- All 20 vanilla recipes that create a weapon item (mines and grenades) use the Chemistry Station: `WorkbenchChemlab` `00102158`, category `RecipeUtility` `0006980C`. *(xEdit)*
- A `HasPerk` condition shows the recipe **greyed**, not hidden. *(in game)* Whether other conditions (a quest stage) hide it is untested.
- The craft screen lists **only two** perk prerequisites; a third is enforced but invisible. *(in game)*
- A recipe condition can see possession (`GetItemCount`) but, as far as we know, not stored gear or whether a holotape was played. *(inferred)* A held perk cannot be stashed, so gate on a perk.

## A book that grants a perk, with no script *(in game)*
- `BOOK` with the `DNAM` flag **Add Perk** and **Teaches** = the perk. The perk stays after the book is stashed. The magazines' `CA_SkillMagazineScript` is not needed.
- Base used: the Silver Shroud Photo `0018AC1E`. Perk: a copy of the Barbarian magazine perk `0008E3A6` with effects removed and `Num Ranks` set to 1 (the copy inherits 10).
- A holotape (`NOTE`) cannot grant a perk without a script.

## Limited-supply items
- "Found broken, then repaired": a plain `MISC` as the ingredient limits copies to the world's supply. *(in game)* A container holding it must not refill.
- `UnscrappableObject` `001CC46A` keeps an item off the scrap list. *(in game)* Remove scrap components too.
- A `MISC` base that worked: the Silver Locket `00059B34`.

## Placing things (not tried)
Add our own container beside the spot rather than editing a shared base record (e.g. `Prewar_Locker_01` `00018F7B`). Get coordinates with `player.getpos x`, `y`, `z`. *(inferred)*
