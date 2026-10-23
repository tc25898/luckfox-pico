# Local development baseline

The local `main` branch includes the verified host compatibility work and a
CMake modernization for all active components of the selected RV1106 target.
These local changes are not submitted as new pull requests.

## Published commits

The following commits are merged into local `main` with their original
identities preserved. Their PR branches have not been amended or rebased:

- `85e339a1e33c0cc29d18525afc7e659028b33d7a`: board e2fsprogs compatibility,
  published in Luckfox PR #379.
- `e040f1e0609bd69f979d260009c5d7f07ce27f89`: vendor CMake compatibility,
  published in Luckfox PR #380, withdrawn (closed without merging) on
  October 22, 2026 in favor of direct CMake modernization. PR #379 remains open.

## Local CMake modernization

All seven CMake components exercised by the selected RV1106 Pico Pi build
now declare `cmake_minimum_required(VERSION 3.16...4.2)`. The syntax requires
CMake 3.16 and selects policies through 4.2. `project/build.sh` no longer
supplies a global policy-minimum workaround. An explicitly exported
`CMAKE_POLICY_VERSION_MINIMUM` is still inherited normally.

- The common algorithm root has an explicit `project()` declaration.
- RV1106 camera and ISP demo include paths, link paths, and definitions
  are scoped to their executable targets, with private link libraries.
- IVE scopes its sample includes and links to the target and installs it
  using `install(TARGETS ...)`.
- Rockit compiler flags and definitions use their respective CMake commands.
- Camera build-type comparisons handle an empty value. ISP uses a string
  comparison for its operating system and removes a redundant policy setting.

On October 22, 2026, all seven components were configured, built, and
installed from empty build directories using CMake 4.2.3 and the SDK's ARM
GCC 8.3 toolchain. Modernized builds had no policy override in the environment
or cache. Options matched the existing RV1106 configuration: Release,
except for the ISP demo's MinSizeRel configuration.

Compared with fresh baseline builds using the previous CMake files:

| Component | Outputs checked | Result |
| --- | ---: | --- |
| common_algorithm | 42 installed files | Identical contents and permissions |
| IVA | 11 installed files | Identical contents and permissions |
| IVE | 10 installed files | Identical contents and permissions |
| Rockit MPI | 67 installed files | Identical contents and permissions |
| rkfsmk | 2 installed files | Identical contents and permissions |
| RV1106 rkipc | 13 installed files | Identical contents and permissions |
| RV1106 ISP demo | 1 executable | 44 identical object files; identical shared-library dependencies |

The ISP executable's runtime search path differs: it embeds the temporary
build directory and no longer includes an accidental source-directory entry
caused by empty arguments to directory-level `link_directories()`. After
accounting for these path differences, its link commands match.

Temporary comparison builds, logs, options, harness, and previous SDK CMake
build directories are under `/tmp/luckfox-cmake-expansion/` and may disappear
on reboot. The comparison summary is also documented in the neighboring
backyard-defense repository's `docs/luckfox-cmake-modernization.md`.

These checks cover the selected RV1106 build. Other boards, optional TGI and
algorithm examples, RGA samples, bundled third-party sources, MCU builds,
other build types, and older CMake versions remain unverified. Their CMake
files were left unchanged during this expansion.
Some separately built legacy projects may need an explicit policy override.
Device boot and hardware operation require separate testing.

## Full SDK integration result

The subsequent full `./build.sh` command exited with status 0 and completed
`build_all`, `build_firmware`, `build_updateimg`, `build_save`, and
`build_allsave`. The selected CMake build directories and cached package
outputs were moved aside to force fresh configuration and compilation.
ISP's prebuilt libraries were retained and its demo build directory was
refreshed. Other SDK stages used their normal incremental state.

All seven resulting CMake caches contain no policy-minimum override. The
log contains no CMake errors or deprecation warnings; existing warnings
about unused generic options supplied by SDK wrappers remain.

- Log: `~/luckfox-cmake-modernization-build.log`
- Firmware: `output/image/update.img` (478,310,986 bytes)
- Release: `IMAGE/IPC_EMMC_BUILDROOT_RV1106_LUCKFOX_PICO_PI_20261022.1633_RELEASE_TEST/`

No hardware was flashed or tested.

## Preserved Buildroot fixes

`sysdrv/tools/board/buildroot/local-host-compat.patch` records the changes
already applied to the current extracted Buildroot 2023.02.6 directory.
This preserves the fixes in Git instead of relying solely on ignored files.

After extracting a fresh, unmodified copy of the bundled Buildroot archive,
apply them before building packages:

```sh
cd sysdrv/source/buildroot/buildroot-2023.02.6
patch --dry-run -p1 < ../../../tools/board/buildroot/local-host-compat.patch
patch -p1 < ../../../tools/board/buildroot/local-host-compat.patch
```

The current extraction is already patched. SDK extraction automation remains
deferred; there is no third PR. Changing package recipes in a partially
built tree may require package reconfiguration or re-extraction.

## Generated outputs

The existing rebuilt hostapd, hostapd_cli, and librkwifibt.so outputs are
recorded in a separate local commit from the source changes. Firmware images
and large generated SDK directories remain ignored. No remote branch was
pushed as part of this local consolidation.
