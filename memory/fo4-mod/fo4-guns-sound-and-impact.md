---
name: fo4-guns-sound-and-impact
description: Gun sounds and silencing, explosions on impact, stagger and other on-hit effects.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-25
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Guns: sound, silencing, impact explosions, on-hit effects (2026-11-25, *in game* unless marked)

- **Gunshot sounds** come from `KSSM` (sound keyword mapping) records matched on the weapon's `s_` keywords (e.g. `s_40_PistolFortyFour` + `s_10_Player`); a suppressor adds `s_20_Silenced`. A weapon with no silenced mapping still sounds loud: copy another weapon's silenced mapping and swap its `s_40_` keyword.
- **Silent to NPCs** is the weapon's `DNAM` Sound Level = Silent; it does not change the audible sound.
- **Explosion on impact** needs a projectile with the `Explosion` flag. Bullets (`Projectile45Caliber`) are Beam type without it, and naming an explosion on a copy did nothing. A copy of `MS09PushProjectile` `00199046` (Missile) works. The override projectile is `FNAM` `Override Projectile` on the weapon.
- **Legendary effects** can be built into a weapon by including the legendary mod in every object template combination (Staggering `001E81AB`); the "legendary item" pickup screen appears. Console `amod <modID>` on a dropped item tests one without a build. A weapon record's `EITM` stagger enchantment never applied.
- **Chance-based on-hit effect from a perk:** an `Apply Combat Hit Spell` entry (copy from `BigLeagues02` `000E36FC`), attacker-tab conditions (`WornHasKeyword`, `GetRandomPercent`), weapon-tab `HasKeyword`, spell copied from `PerkDisarm` `001D604B`.
- **Recolouring BC1/BC3 textures** per block, per mip, with plain Python works for weapons and clothing; blend and re-encode only edge blocks for smooth shapes.
