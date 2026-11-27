---
name: fo4-gotchas-testing
description: Standing rules for assets, hand-offs and in-game tests. Read at the start of any FO4 mod work.
applies-to: [fo4-mod]
kind: gotcha
verified: 2026-11-26
---

# Asset and testing gotchas

Detail: `fo4-testing`, `fo4-deck-workflow`, `fo4-materials-and-models`.

## Assets
- Never commit game assets, built plugins or anything derived from them (Rhodes `docs/decisions.md`, 2026-11-24). Commit the script that makes them.
- Never write a generated file under a vanilla path.
- Loose files need, in the profile's `fallout4custom.ini`, `[Archive]` with `bInvalidateOlderFiles=1` and an empty `sResourceDataDirsFinal=`.

## Hand-offs and tests
- Anything the owner has to type or paste (console commands, IDs, paths, search text) goes in a fenced code block, one per paste target, with real IDs filled in where known: the Deck trackpad makes selecting text awkward. Not inside tables or inline code.
- God mode (`tgm`) gives unlimited AP: turn it off before judging AP drain (fresh test saves often have it on).
- Keep difficulty on Normal when measuring damage.
- Record stats do not always match spawned actors; check in game.
- Hits to kill is too noisy to judge a damage effect: compare per-hit numbers on the SKK dummy. The dummy cannot show a stagger; use a live enemy.
- Fandom and Nexus have refused automated reads: do not route around it; ask for pasted text. Treat AI-generated record details as leads until read.
- Say what is verified in game and what is only built or read.
