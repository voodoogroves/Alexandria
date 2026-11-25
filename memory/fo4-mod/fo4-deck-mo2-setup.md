---
name: fo4-deck-mo2-setup
description: Rebuilding Mod Organizer 2 and F4SE on a new or reset Steam Deck, recovery and troubleshooting.
applies-to: [fo4-mod]
kind: lookup
verified: 2026-11-23
---
Runbook for rebuilding the modding setup on a new or reset Steam Deck. **Status (2026-11-23):** the Fallout 4 instance was built from this runbook and reached the main menu through F4SE. Everything under "Per game" is verified. Open items are at the bottom. Facts from upstream docs that haven't been run here are marked *(upstream)*.

## The model

One independent MO2 instance per game, built by **MO2-LINT** (`~/Repos/upstream/modorganizer2-linux-installer`, docs at docs.furglitch.com/modorganizer2-linux-installer). It installs MO2 into a normal folder, puts `mo2-redirector.exe` in the game folder, and registers a Steam compatibility tool for the game (`mo2_<appid>_redirector`, so `mo2_377160_redirector` for Fallout 4). You set the **game's own Steam entry** to that tool. Launching the game from Steam then opens MO2, and the game is started from MO2's top-right dropdown (F4SE, Fallout 4, ...). It appears in gaming mode like any Steam game.

Never launch the game any other way (plain Steam without the tool, or Wine directly). That can break the prefix.

## Prerequisites

1. Desktop mode, and a real terminal (Konsole). The installer's menus need a TTY, so an assistant's shell can't run `install` (it aborts with "Input is not a terminal").
2. **Proton 11.0** installed in Steam. It's the only version upstream supports.
3. The game installed and **launched once to the main menu**, so Steam creates its prefix (`compatdata/<appid>/pfx`).
4. At least one other game or shortcut left on the stock Proton, or Steam may remove it and break the MO2 tool.

## Install the installer

1. Download the `mo2-lint` release binary (v7.0.0+) from the GitHub releases page.
2. `chmod +x mo2-lint`, then move it to `~/.local/bin/` (already on PATH).
3. Config is created at `~/.config/mo2-lint/settings.toml`. Defaults put instances under `~/Games/mo2-lint/mo2-{game}-{launcher}`. State is `~/.config/mo2-lint/state.json`; logs are `~/.cache/mo2-lint/logs/`.
4. v7.0.0 worked, though 7.0.5 was out. If something fails, update the binary first.

## Per game

```bash
mo2-lint install --help                      # find the exact game id
mo2-lint install fallout4 ~/Games/mo2-lint/mo2-fallout4-steam -L steam -s
```

`-s` installs the script extender (F4SE for Fallout 4). The prompts, in order:

1. **"Have you set the compatibility tool?"** Answer **y**. The tool doesn't exist yet at this point; it appears only after the install. The prefix just needs to exist.
2. **Script extender version.** F4SE must match the game's exact version. Read it first:
   ```bash
   python3 -I -c 'd=open("Fallout4.exe","rb").read();i=d.find("FileVersion".encode("utf-16le"));print(d[i:i+100].decode("utf-16le","ignore"))'
   ```
   (run from the game folder `~/.local/share/Steam/steamapps/common/Fallout 4`). On 2026-11-23 the game was 1.11.240.0, which is **F4SE 0.7.9**. Menu entries list the runtime they need.
3. **Menus use arrow keys and Enter.** Typing a number does nothing visible and selects the highlighted entry. Check the highlight before pressing Enter.

Then in Steam: Fallout 4 **Properties → Compatibility → force a specific tool → `mo2_377160_redirector`**. Launch the game from Steam; MO2 opens.

### MO2 first run

- **Nexus categories:** choose Nexus categories (cosmetic; changeable in Settings → Categories).
- **nxm links:** yes. Only one program owns `nxm://` at a time, so if you ever run more than one MO2 on the Deck, the most recent registration wins. The "Mod Manager Download" button is untested here.
- **Unmanaged entries** in the mod list are normal: base game, DLC, Creation Club, and anything installed from the in-game menu live in the game's `Data` folder. See "Unmanaged mods".

### Verify F4SE

Pick **F4SE** in MO2's dropdown and Run. Get to the main menu, then quit. Then check the logs:

`~/.local/share/Steam/steamapps/compatdata/377160/pfx/drive_c/users/steamuser/Documents/My Games/Fallout4/F4SE/`

- `f4se.log` should show `F4SE runtime: initialize (version = 0.7.9 ...)` and end at `init complete` / `RegisterPapyrusFunctions_Hook`.
- `f4se_loader.log` ends at `launching`. `WinVerifyTrust ... 80090008` lines are normal under Wine; ignore them.
- The log is in the **prefix's Documents**, not MO2's `overwrite` folder.

Verified 2026-11-23: F4SE 0.7.9 loaded and the game reached the main menu with about 85 plugins in the load order.

### Recovery: wrong F4SE version (or any bad install)

Don't re-run `install` over it (the first run had already completed; a second one hits a non-empty folder). Run:

```bash
mo2-lint uninstall -g fallout4
```

It removes the redirector, the Steam wrapper, the launch option, and every tracked F4SE file. Verified clean on 2026-11-23: no leftover `f4se_1_10_163.dll`. Then install again.

## Fallout 4 caveats *(upstream)*

- Some F4SE plugins misbehave under Proton. No F4SE plugins are installed yet.
- Missing some audio: add `WINEDLLOVERRIDES="winmm,version=n,b" %command%` to the game's launch options. If it persists, try Proton GE.
- ENB 0.393 and older may need `EnablePostPassShader` disabled.
- F4SE breaks whenever Steam updates Fallout 4 ahead of a new F4SE build. Fix: install the newer F4SE (uninstall and reinstall the instance, or see upstream `update`).

## Troubleshooting

- Install aborts with "Could not resolve Steam Proton wrapper": the required Proton isn't installed (seen 2026-10-28, Proton Experimental). Install Proton 11.0 and retry.
- Install aborts with "Input is not a terminal": run it in Konsole, not from an assistant.

## Open

- Split the FO4Edit entry in two (normal and lookup); see `fo4-xedit-lookup`.
- Test a Nexus "Mod Manager Download" and confirm it lands in this instance.
