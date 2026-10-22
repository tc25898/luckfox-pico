# Local development baseline

The local `main` branch includes the verified host compatibility work and a
bounded CMake modernization for the selected RV1106 development target.
These local changes are not submitted as new pull requests.

## Published commits

The following commits are merged into local `main` with their original
identities preserved. Their PR branches have not been amended or rebased:

- `85e339a1e33c0cc29d18525afc7e659028b33d7a`: board e2fsprogs compatibility,
  published in Luckfox PR #379.
- `e040f1e0609bd69f979d260009c5d7f07ce27f89`: vendor CMake compatibility,
  published in Luckfox PR #380.

## Local CMake modernization

- The common algorithm component and selected RV1106 camera project use
  `cmake_minimum_required(VERSION 3.16...4.2)`. The syntax records CMake
  3.16 as the minimum and selects policies through 4.2.
- The common algorithm root has an explicit `project()` declaration.
- RV1106 camera include paths, link paths, and the ISP definition are scoped
  to the executable target; its link libraries are explicitly private.
- Camera optimization/debug flags use `add_compile_options()` rather than
  `add_definitions()`, and build-type comparisons handle an empty value.

On October 22, 2026, both components were configured, built, and installed
from empty build directories using CMake 4.2.3 and the SDK's ARM GCC 8.3.0
toolchain, with `CMAKE_POLICY_VERSION_MINIMUM` absent from the environment
and cache. The selected RV1106 Release options matched the existing SDK.

Compared with fresh baseline builds using the previous CMake files:

| Component | Installed files | Content comparison |
| --- | --- | --- |
| common_algorithm | 42 | All files identical |
| RV1106 rkipc | 13 | All files identical, including the ARM executable |

The camera shared-library dependency list was also identical. CMake's
legacy-policy and missing-project warnings are resolved for these paths.
The SDK supplies two unused find-root-path options to the camera project;
those still produce an informational configuration warning.

Only the selected RV1106 Release configuration was built. Alternate boards,
optional algorithm examples, other build types, and older CMake versions
were not tested. Other vendor projects retain their existing CMake files
and may still need the compatibility default in `project/build.sh`.

The firmware images in `output/image` remain the previously verified
build; these local CMake changes were tested in isolated directories and
have not been repackaged or boot-tested on hardware.

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
