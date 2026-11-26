# Building the SKSE plugin (AE)

## Prerequisites

| Tool | Notes |
|------|--------|
| **Visual Studio 2022+** | Desktop development with C++ (MSVC) |
| **CMake** 3.20+ | On `PATH` |
| **vcpkg** | Set `VCPKG_ROOT` to your vcpkg install |
| **Git** | For CommonLibSSE |
| **Address Library** | Runtime dependency (not a compile dependency) |

Target: **Skyrim AE** (`BUILD_SKYRIMAE=ON`), Address Library–based, version-independent metadata.

## One-time setup

From the repository root:

```powershell
cd source-publish\SimpleOffenceSuppression

# CommonLibSSE (powerofthree), pinned to the last snapshot that still has include/REL.
# Later dev dropped include/REL (commonlib-shared migration, 2026-08-21); CI uses the same pin.
git clone --depth 1 --branch dev https://github.com/powerof3/CommonLibSSE.git extern\CommonLibSSE
git -C extern\CommonLibSSE fetch --depth 1 origin 86eb2b853101053390a81378b6263c2911603824
git -C extern\CommonLibSSE checkout --quiet 86eb2b853101053390a81378b6263c2911603824
git clone --depth 1 https://github.com/alandtse/CommonLibVR.git extern\CommonLibVR

# A classic vcpkg clone (git clone https://github.com/microsoft/vcpkg + bootstrap-vcpkg.bat).
# The copy bundled with Visual Studio refuses vcpkg.json without a builtin-baseline.
$env:VCPKG_ROOT = "C:\path\to\vcpkg"
```

vcpkg packages are pulled via `vcpkg.json` (simpleini, spdlog, xbyak, boost-stl-interfaces, rsm-binary-io, …).

## Configure & build

```powershell
# From source-publish\SimpleOffenceSuppression:

cmake --preset vs2022-windows-vcpkg-ae -DCOPY_BUILD=OFF
cmake --build buildae --config Release
```

Output:

```text
buildae\Release\po3_SimpleOffenceSuppression.dll
```

Copy that DLL into a mod's `SKSE\Plugins\` (same name as stock SOS, it **replaces** the original DLL).

`COPY_BUILD` defaults to ON in `CMakeLists.txt`: when `SkyrimAEPath` is set, a post-build step copies the DLL and PDB
into that game folder. Pass `-DCOPY_BUILD=OFF` to keep the build out of your game install.

Equivalent manual configure without the preset:

```powershell
cmake -B buildae -S . `
  -DBUILD_SKYRIMAE=ON `
  -DCOPY_BUILD=OFF `
  -G "Visual Studio 17 2022" -A x64 -T v143
cmake --build buildae --config Release -j 8
```

### Several MSVC toolsets installed

vcpkg builds its libraries with the newest MSVC it finds, while the preset's `v143` toolset uses that Visual Studio
instance's default version. If the two differ, the link fails with unresolved `__std_find_*_trivial_*` symbols from
`spdlog.lib`. Point vcpkg at the same compiler: set `VCPKG_VISUAL_STUDIO_PATH` to the VS 2022 instance, and if that
instance holds several v143 versions, add an overlay triplet (`VCPKG_OVERLAY_TRIPLETS`) whose
`x64-windows-static.cmake` sets `VCPKG_PLATFORM_TOOLSET_VERSION` to the version the project compiles with
(shown as `The CXX compiler identification is MSVC 19.xx` during configure). GitHub's windows-2022 runner has one
toolset and needs neither.

## Source map

| Path | Role |
|------|------|
| `src/Hooks.cpp` / `Hooks.h` | `GetFactionFightReaction` thunk; `ShouldSuppress` gate |
| `src/Exclusions.h` | Faction / NPC base / keyword / aggression cache |
| `src/Settings.h` | INI load including `[Exclusions]` |
| `src/main.cpp` | SKSE plugin entry (AE + SE query path) |

## Design note

The hook only rewrites **Neutral → Friend/Ally** reactions. Exclusions force Neutral (no suppression) so real foes can take offence and damage normally. MCM Block Friendly Fire is patched in the ESP so damage is not multiplied by zero on keyword-tagged hostiles.
