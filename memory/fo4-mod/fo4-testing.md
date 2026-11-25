---
name: fo4-testing
description: In-game testing on the Deck - Test profile, console commands, measuring damage, unmanaged mods.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-23
---
Tags: *(in game)* tested by playing; *(xEdit)* read from the vanilla records; *(inferred)* not tested. Setup: Proton 11.0, FO4Edit 4.1.5f, game 1.11.240.

## Testing in game on the Deck

- **A clean Test profile plus a prepared save** (level 20, the perks needed, a few of the item under test) makes every test start from the same state. Local saves are on, so saves stay in the profile.
- **Console:** needs a keyboard (Bluetooth works); the tilde key opens it. The cheat terminal holotape is easier on a controller for god mode and spawning. Console text is small and `getav` readouts were unreliable here, so prefer a measure that needs no reading, such as counting swings to kill, or attach a screenshot to chat for the assistant to read.
- **Record stats do not always match spawned actors** (templates, level scaling). Check against the game, not just the xEdit record.
- **Finding an item's FormID in the console:** `help "word" 4 WEAP` (quotes around the word; `4` = other forms; the last word is the 4-letter form type, e.g. `WEAP`, `ARMO`, `PERK`). It lists matches as `(FormID) 'Display Name'`. Short single words work best (`"Chinese"` found the Chinese Officer Sword, `"Wonders"` found nothing, so some items are not matched by their wiki name). A FormID starting `FE` belongs to an ESL-flagged plugin. Use the wiki's `xx` Form ID with the file's load-order prefix (Nuka-World is `06`), but note a wiki "Form ID" can be a placed reference, not the item.
- **Console checks for a weapon effect:** `player.getav ArmorPenetration` and `player.hasperk <PerkID>` read the equipped state. A mundane melee mod (e.g. Puncturing) shows no stat: it only carries a keyword that the base `PlayerPerk` reads (flat 15%).
- **Hand-offs (2026-11-24):** the in-game console **accepts a paste**, so give console commands and paths in fenced code blocks (one per paste target) that the editor shows with a copy button. The Steam screenshot shortcut did not work with the console open; the keyboard screenshot did. Prefer pasted text for readouts when possible.
- **Set difficulty to Normal and keep it fixed when measuring damage.** Difficulty scales the damage you deal (the Very Hard Protectron runs made absolute inferences such as target HP unreliable; ratios between weapons stay valid). A damage-numbers mod (Floating Damage, an F4SE mod on Nexus) beats counting swings, but check its game-version support first.
- For a before/after test of an effect, use a throwaway "TEST" plugin variant (different plugin name, editor IDs and item name) so the real plugin stays untouched; untick it afterwards.

## Testing

- The SKK workshop target dummy shows damage numbers, never goes hostile, and takes any armor pieces: damage unarmored vs in the heavy combat chest shows the armor effect directly.
- Difficulty to Normal and fixed; the in-game console accepts a paste.

## Unmanaged mods

MO2 lists plugins in the game's `Data` folder that aren't in a mod folder as "Unmanaged". On this Deck that's about 85 plugins:

- About 18 official: base game, 6 DLC, 10 Creation Club `cc*.esl`. Normal.
- The rest are third-party (Sim Settlements, Clothing Overhaul, SKK, Eli's mods, and so on), almost certainly from the in-game Mods menu. They load, but MO2 can't version or update them.

For mod development, use a separate **Test profile** with the third-party plugins disabled, leaving vanilla, DLC and CC. This doesn't touch the game folder. Don't delete the unmanaged mods: they're packed as BA2s named after their plugins (107 non-Bethesda archives; the only loose `Data` folders are F4SE's `Scripts`, `Video` and xEdit's cache), so unticking a plugin also stops its assets loading.

**Building the Test profile by hand** (done 2026-11-23, with MO2 closed; MO2 accepted it and showed the right plugin list):

```bash
cd ~/Games/mo2-lint/mo2-fallout4-steam/profiles
cp -r Default Test
# in Test/plugins.txt: strip the leading * from every line except the plugins to keep
# (files use CRLF line endings; keep them)
# in Test/settings.ini: LocalSaves=false -> LocalSaves=true
```

- `*Unmanaged:` lines in `modlist.txt` can't be switched off there; `plugins.txt` is what controls loading.
- Official DLC and Creation Club plugins aren't listed in `plugins.txt` (the game enables CC through `Fallout4.ccc`), so they stay on.
- Kept enabled: `cheatterminal.esp` (needs only `Fallout4.esm`) for god mode, spawning items and moving around during tests.
- `LocalSaves=true` keeps test saves out of the normal profile. In MO2 pick **Test** from the profile dropdown; answer No if asked to transfer saves.
- Game launch with this profile: works (2026-11-23). The prologue autosaves landed in `profiles/Test/saves`, so local saves are working. Confirmed in game: the cheat terminal holotape is there and the third-party mods are gone.
