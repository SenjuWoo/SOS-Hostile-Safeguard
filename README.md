<p align="center">
  <img src="docs/images/logo.svg" width="72" height="72" alt="SOS Hostile Safeguard mark">
</p>

<h1 align="center">SOS Hostile Safeguard</h1>

<p align="center"><strong>Civilians stay protected. Temporary “friends” do not.</strong></p>

<p align="center">
  Improved fork of <a href="https://github.com/powerof3/SimpleOffenceSuppression">Simple Offence Suppression</a>
  (powerofthree).<br>
  Toll bandits, Arvel, Weylin, and other calm-for-a-moment foes can take damage again.
</p>

<p align="center">
  <a href="https://github.com/SenjuWoo/SOS-Hostile-Safeguard/actions/workflows/build.yml"><img src="https://github.com/SenjuWoo/SOS-Hostile-Safeguard/actions/workflows/build.yml/badge.svg" alt="Build"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-f07178?labelColor=0d0f11" alt="MIT License"></a>
  <a href="https://github.com/SenjuWoo/SOS-Hostile-Safeguard/releases/tag/v1.0.1"><img src="https://img.shields.io/badge/release-v1.0.1-f07178?labelColor=0d0f11" alt="v1.0.1"></a>
  <img src="https://img.shields.io/badge/DLL-SOS%202.3.1-8f9aa6?labelColor=0d0f11" alt="DLL 2.3.1">
</p>

<p align="center">
  <a href="#install">Install</a>
  ·
  <a href="#configuration">Config</a>
  ·
  <a href="#build">Build</a>
  ·
  <a href="#honest-status">Honest status</a>
  ·
  <a href="CHANGELOG.md">Changelog</a>
</p>

## Why it exists

Simple Offence Suppression — especially **[SOS MCM – Block Friendly Fire](https://www.nexusmods.com/skyrimspecialedition/mods/41774)** — treats any NPC who is **not currently hostile** as a friend when you hit them.

That also catches enemies who are only calm for a moment:

| Example | Why stock SOS breaks the fantasy |
| --- | --- |
| **Toll / highway bandits** after you pay | Still bandits; surprise attacks do nothing useful |
| **Arvel the Swift** | `BanditAllyFaction`, flees with the claw — protected as “friend” |
| **Weylin** (Markarth market, attacks Margret) | Base record has **zero factions** — faction-only patches miss him |
| Modded / quest “talk first” foes | Neutral reaction → Friend promotion → no damage / no offence |

Stock SOS is right for accidental hits on guards and shopkeepers. It should **not** protect scripted assassins and paid-off bandits.

## What this project is

Hybrid package (ship tree: `release/1.0.1/`):

1. **Replacement DLL** — `po3_SimpleOffenceSuppression.dll` (same filename as stock SOS; improved **2.3.1** fork). Disable the original Simple Offence Suppression mod folder so only this DLL loads.
2. **Light ESP** — keyword, formlist, and a **patch** to the MCM friendly-fire perk (does not redistribute the full MCM mod).
3. **SPID `_DISTR.ini`** — tags common hostile factions + unique EditorIDs with the exclusion keyword.

| Component | Role |
| --- | --- |
| DLL exclusions | Skip Neutral→Friend promotion for hostiles |
| `HostileFactions` / `HostileNPCs` INI | Data-driven lists (no recompile for new EditorIDs) |
| Keyword + SPID | MCM damage block also skips tagged hostiles |
| Very Aggressive rule | Restores SOS v1.1-style behaviour for high aggression |

The hook only rewrites **Neutral → Friend/Ally** reactions. Exclusions force Neutral (no suppression) so real foes can take offence and damage normally.

## Requirements

**Hard**

- [SKSE64](https://skse.silverlock.org/)
- [Address Library for SKSE Plugins](https://www.nexusmods.com/skyrimspecialedition/mods/32444)
- [Simple Offence Suppression MCM – Block Friendly Fire](https://www.nexusmods.com/skyrimspecialedition/mods/41774) (ESP master; perk this package patches)

**Strongly recommended**

- [Spell Perk Item Distributor (SPID)](https://www.nexusmods.com/skyrimspecialedition/mods/36869)

**Replaced at runtime**

- [Simple Offence Suppression](https://www.nexusmods.com/skyrimspecialedition/mods/41764) **DLL** — do not run stock and this fork side-by-side under the same plugin name.

## Install

1. Disable the original **Simple Offence Suppression** mod (po3).
2. Keep **SOS MCM** enabled.
3. Install `release/1.0.1` (or a Nexus zip built from that tree).
4. Enable **`SOS Hostile Safeguard.esp`** (light / ESPFE) **after** SOS MCM.
5. Launch with SKSE. Optional: check the SKSE log for Hostile Safeguard / exclusion cache lines.

```text
SKSE/Plugins/po3_SimpleOffenceSuppression.dll
SKSE/Plugins/po3_SimpleOffenceSuppression.ini
SOS Hostile Safeguard.esp
SOS Hostile Safeguard_DISTR.ini
```

## Configuration

Full key list: [`docs/CONFIGURATION.md`](docs/CONFIGURATION.md).

Cover another unique assassin:

```ini
[Exclusions]
HostileNPCs = MS01Weylin,MS13Arvel,e3DemoArvel,MyModAssassinEditorID
```

Optional SPID line:

```ini
Keyword = SOS_NoOffenceSuppression|MyModAssassinEditorID
```

| Situation | Use |
| --- | --- |
| Bandits / forsworn / undead packs | `HostileFactions` or SPID faction lines |
| Unique with an **empty** faction list (Weylin) | `HostileNPCs` + SPID EditorID line |
| Modded assassin by EditorID | Append to `HostileNPCs` |
| Shareable extension without editing INI | Inject into FormList `SOS_HostileFactionList` |

Restart the game after INI changes. Without SPID, DLL faction/NPC EditorID lists still work; MCM damage blocking may still apply to untagged uniques on the perk path — `HostileNPCs` still covers offence reaction in the DLL.

## Build

Tools and the longer recipe: [`docs/BUILD.md`](docs/BUILD.md).

CI (windows-2022) pins CommonLibSSE to `86eb2b853101053390a81378b6263c2911603824` because later `dev` dropped `include/REL`. From `source-publish/SimpleOffenceSuppression`:

```powershell
git clone --depth 1 https://github.com/powerof3/CommonLibSSE.git extern/CommonLibSSE
git -C extern/CommonLibSSE fetch --depth 1 origin 86eb2b853101053390a81378b6263c2911603824
git -C extern/CommonLibSSE checkout 86eb2b853101053390a81378b6263c2911603824
git clone --depth 1 https://github.com/alandtse/CommonLibVR.git extern/CommonLibVR

cmake --preset vs2022-windows-vcpkg-ae
cmake --build buildae --config Release
```

Needs Visual Studio 2022 C++ tools, CMake 3.20+, and `VCPKG_ROOT`. Output: `buildae\Release\po3_SimpleOffenceSuppression.dll`. Copy that over stock SOS (same filename).

Equivalent manual configure (`BUILD_SKYRIMAE=ON`) is in [`docs/BUILD.md`](docs/BUILD.md).

## Project map

```text
README.md                 this file
LICENSE                   MIT (powerofthree + fork)
CHANGELOG.md              1.0.0 / 1.0.1
docs/                     BUILD, CONFIGURATION, Nexus copy
docs/images/logo.svg      mark
release/1.0.1/            player-facing ship tree
source-publish/
  SimpleOffenceSuppression/   forked plugin sources (no CommonLibSSE binary tree)
```

Plugin sources: `src/Hooks.cpp` (`GetFactionFightReaction` thunk, `ShouldSuppress`), `src/Exclusions.h`, `src/Settings.h` (`[Exclusions]`), `src/main.cpp`.

## Honest status

- Package version **1.0.1**; DLL **2.3.1**.
- Built for AE / Address Library style loading (1.6.x class runtimes).
- Structural validation (compile, ESP packaging) is done in development. **In-game confirmation of every encounter is still appreciated** — report NPCs that remain undamageable with EditorID if possible.
- True civilians, guards, and genuine allies remain protected by design.
- Other SOS “INI-only tweaks” are unnecessary if you use this package’s INI. Never load two `po3_SimpleOffenceSuppression.dll` files.
- GitHub Actions compiles the AE preset; this README pass does not claim a live-game matrix.

## Credits

| Credit | Contribution |
| --- | --- |
| **powerofthree** | Original Simple Offence Suppression (design, SKSE plugin, MIT source); SPID |
| **wankingSkeever** (et al.) | SOS MCM – Block Friendly Fire (perk behaviour patched, not rehosted wholesale) |
| **Ryan / CommonLibSSE lineage** | Library used by the upstream plugin family |
| **meh321** | Address Library |
| **ShugokiFable** | Hostile Safeguard fork (exclusions, unique NPC list, MCM perk patch, packaging) |

## Permissions

- Upstream SOS allows modification (MIT + Nexus modification permission).
- This repo ships **modified** DLL sources and **new** addon files only.
- Do not sell. Do not convert to other games under upstream conversion rules where they apply to assets.

## Links

- Upstream source: https://github.com/powerof3/SimpleOffenceSuppression
- Upstream Nexus: https://www.nexusmods.com/skyrimspecialedition/mods/41764
- Issues / PRs: this repository, for fork behaviour (Weylin, exclusions, packaging)

## License

[MIT](LICENSE) — powerofthree + this fork.
