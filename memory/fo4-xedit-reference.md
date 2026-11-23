# FO4 / xEdit scripting reference

Verified against the official xEdit source (`TES5Edit/TES5Edit`, shallow clone at commit `73f919a`, 2026-11-23), mainly `Core/wbDefinitionsFO4.pas`, `Core/wbDefinitionsCommon.pas` and `xEdit/JvI/xejviScriptAdapter*.pas`. Reread the source before trusting this after an xEdit update. Not tested in-game.

## Condition functions (CTDA `Function` index)

| Function | Index | Params |
|---|---|---|
| GetItemCount | 47 | base object |
| GetStageDone | 59 | quest, quest stage |
| GetInFaction | 71 | faction |
| GetEquipped | 182 | base object |
| IsSneaking | 286 | none |
| HasPerk | 448 | perk |
| GetPlayerTeammate | 453 | none (no `IsPlayerTeammate` exists in FO4) |
| HasKeyword | 560 | keyword |
| WornHasKeyword | 682 | keyword |

## Condition `Type` byte

- Low bits are flags: `1` Or, `2` Use Aliases, `4` Use Global, `8` Use Packdata, `16` Swap Subject and Target.
- Comparison is `Type and 224`: `0` equal, `32` not equal (by elimination; not read directly), `64` greater, `96` greater-or-equal, `128` less, `160` less-or-equal.
- So `>= 1` is Type `96` and `== 1` is Type `0`. The OR flag sits on the condition that is OR'd with the one after it.

## Condition `Run On`

`0` Subject, `1` Target, `2` Reference, `3` Combat Target, `4` Linked Reference, `5` Quest Alias, `6` Package Data, `7` Event Data, `8` Command Target, `9` Event Camera Ref, `10` My Killer.

Note `Target` is `1`, not `2`. Using `2` selects *Reference*.

## xEdit Pascal script host

- `AddNewFile` exists; `CreateNewFile` does not.
- `AddMasterIfMissing(file, masterName)` is built in. Redefining it in a script as a wrapper that calls itself recurses forever.
- `wbCopyElementToFile(element, file, asNew, asDeepCopy)` exists.
- `AddNewRecord` was not found in the script adapters; add records with `Add(file, 'SIG', True)` (container `Add`).
- `ElementAssign(container, HighInteger, nil, False)` is the append pattern; `HighInteger` is a defined constant.
- Script entry points run per selected record (`Process`). One-time setup belongs in `Initialize`.

## Armor Addon (ARMA) models

- `MOD2` is the **Male** biped model and `MOD3` is the **Female** one (first person: `MOD4` male, `MOD5` female). The Gemini brainstorm and its script have these swapped.
- Either model may be left out in the record definition; what a character of the other gender sees in that case is untested here.

## Biped slots (first-person flags)

`46` Headband, `47` Eyes, `48` Beard, `49` Mouth, `50` **Neck** (`0x00100000`), `51` **Ring** (`0x00200000`), `52` Scalp, `54` Unnamed. The Gemini brainstorm calls 52 the Ring slot; it is Scalp.

## Vanilla asset paths (checked against `Fallout4 - Meshes.ba2` / `Materials.ba2`, Steam install, 2026-11-22)

- **Slinky (red) dress:** worn meshes `Meshes\Clothes\SlinkyDress\DressF.nif` and `DressM.nif` (a male version exists); material `Materials\Clothes\SlinkyDress\SlinkyDress.BGSM`. There is no `Clothes\RedDress\`.
- **Tuxedo:** `Meshes\Clothes\Tuxedo\SuitF.nif` / `SuitM.nif`; material `Materials\Clothes\Tuxedo\Tuxedo.BGSM`.
- **Maxson cape:** worn meshes `Meshes\Armor\KnightUnderArmor\FCapeMaxon.nif` / `MCapeMaxon.nif` (ground object `CapeGO.nif`); one material, `Materials\Armor\KnightUnderArmor\CapeMaxon.BGSM`. Textures are `MaxonCape_d/n/s.DDS`. There is no `MaxonCape.nif`, `MaxonCape.bgsm` or `MaxonCapeClasp.bgsm`.
- **Pool table:** only `Materials\SetDressing\PoolTable\PoolTable01.BGSM` (whole table, not felt-only). There is no `Props\PoolTable\PoolTableFelt.bgsm` and no `Props\Trophy\` materials.
- Method: the BA2 name tables, plus reading the `.bgsm` strings inside each `.nif`. A material swap only works on a material the mesh actually references.

## Vanilla records for the Blade (Fallout4.esm, Steam install, 2026-11-22)

- **Chinese Officer Sword:** `WEAP` `00147BE4` (`ChineseOfficerSword`). The Gemini script's `$000330B8` is wrong.
- **Armor penetration chain used by the vanilla legendary:** OMOD `001F4426` `mod_Legendary_Weapon_ArmorPenetrating` -> ENCH `001F4425` `enchModArmorPenetration` -> MGEF `001F4423` `ModArmorPenetrationAddPerkEffect` -> PERK `001F4166` `ModArmorPenetrationPerk`, whose single entry point carries actor value `ArmorPenetration` (AVIF `00097341`) and a small float. The numeric strength was not decoded here; read it in xEdit.
- Other armor-ignoring records exist for armor linings (`mod_armor_*_IgnoreArmor`, `Mod_IgnoreArmor_AV` `00245BA4`).
- A weapon's own enchantment is its base `EITM`, separate from the legendary OMOD slot (`ap_Legendary`).

## Known unknowns

- FormIDs for vanilla records (e.g. components, templates) are not verified here; look them up in FO4Edit on a real load order.
- Whether a particular enchantment condition can see the wielder (Subject vs Reference) in-game is untested.
