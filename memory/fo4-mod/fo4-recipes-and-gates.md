---
name: fo4-recipes-and-gates
description: Crafting recipes and gating - station and category, HasPerk locks, vanilla gate conditions, Automatron robot unlock globals, two visible prerequisites, a book that grants a perk, unique and broken items.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-26
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the records. Source: the Regalia Eye spec and build spec.

## Recipes (`COBJ`)
- All 20 vanilla recipes that create a weapon item (mines and grenades) use the Chemistry Station: `WorkbenchChemlab` `00102158`, category `RecipeUtility` `0006980C`. *(xEdit)*
- A `HasPerk` condition shows the recipe **greyed**, not hidden. *(in game)* Whether other conditions (a quest stage) hide it is untested.
- The craft screen lists **only two** perk prerequisites; a third is enforced but invisible. *(in game)*
- A recipe condition can see possession (`GetItemCount`) but, as far as we know, not stored gear or whether a holotape was played. *(inferred)* A held perk cannot be stashed, so gate on a perk.

## Which conditions vanilla recipes use *(plugin files, 2026-11-25)*
Recipes using each, in `Fallout4.esm` / `DLCRobot` / `DLCCoast` / `DLCNukaWorld`: `HasPerk` 1009/348/113/185; `GetGlobalValue` 44/400/14/56; `GetStageDone` 33/1/2/75; `GetItemCount` 1/0/0/0 (`co_chem_DN102_FrenzyGrenade` `000A4A95`). `GetStage`, `GetQuestCompleted` (Nuka-World) and `ModdedItemHasKeyword` (apparently filters by what is on the item) also appear.

## Automatron robot unlocks (`DLCRobot.esm`) *(plugin files and script strings, 2026-11-25)*
- Locked recipes require `GetGlobalValue(co_DLC01Bot_..._Global) == 1`; the globals default to 0. One global can gate many recipes (`co_DLC01Bot_Torso_Sentry_Armor_Wasteland_Global`: 63).
- Quest `DLC01BotModQuest` `0100416C`, script `DLC01:DLC01BotModQuestScript`, watches the player's inventory: 23 trigger `MISC` items (`miscmod_DLC01Bot_...`), each tied to a global and a message. `DLC01AllowRobotModUnlocksGlobal` (default 0) is also used by it; effect not read.
- `DLC01MQ05` fragments reference the unlock globals and `DLC01BotModUnlockAllMessage`: finishing the questline unlocks all (owner has seen this in game; the stage is not identified).
- Examples: `co_DLC01Bot_Arm_Assaultron_Global`, `..._Arm_Sentry_Global`, `..._Arm_RoboBrain_Global`, `..._Hand_LightningGun_Global`; per family also `Head`, `Torso`, `Legs`, and `Torso_..._Armor_{Wasteland,Construction}`. Not every global has recipes: check before using one.
- *(inferred)* Our recipe can test the same global with no script, needing `DLCRobot.esm` as a master. That a failed global condition hides **our** recipe is not yet tested.

## A book that grants a perk, with no script *(in game)*
- `BOOK` with the `DNAM` flag **Add Perk** and **Teaches** = the perk. The perk stays after the book is stashed. The magazines' `CA_SkillMagazineScript` is not needed.
- Base used: the Silver Shroud Photo `0018AC1E`. Perk: a copy of the Barbarian magazine perk `0008E3A6` with effects removed and `Num Ranks` set to 1 (the copy inherits 10).
- A holotape (`NOTE`) cannot grant a perk without a script.

## Limited-supply items
- "Found broken, then repaired": a plain `MISC` as the ingredient limits copies to the world's supply. *(in game)* A container holding it must not refill.
- `UnscrappableObject` `001CC46A` keeps an item off the scrap list. *(in game)* Remove scrap components too.
- A `MISC` base that worked: the Silver Locket `00059B34`.

- **Make the ingredient a plain item, not a wearable.** Crafting from a wearable broken veil crashed the game (cause not proven), and crafting from a worn dress left its look stuck on the player. Plain `MISC` copies fixed both. *(in game, 2026-11-26)*

## Placing things
See `fo4-world-placement`.
