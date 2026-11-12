# Luckfox Pico Pi: UART2 M1 reserved for ESP32

## Scope

This is a **board-profile kernel Device Tree** change for the EMMC Buildroot
`BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Pi-IPC.mk` target
(`RK_KERNEL_DTS=rv1106g-luckfox-pico-pi.dts`). It is not a generic
RV1106 or every Luckfox board change.

On the observed Pico Pi A W, native UART2 at `/dev/ttyS2` drives a Waveshare
ESP32 over GPIO1_B2 (TX) and GPIO1_B3 (RX). Direct register experiments
reported that the default TX drive Level 0 failed while every tested
higher level communicated. A Level 1 (4 mA) setting is the least-strength
known-working selection; this does **not** quantify its long-term signal
integrity or margin over temperature, cable load, and supply variation.

The vendor tree leaves `&uart2` disabled and enables
`&fiq_debugger` on serial ID 2. Enabling a pinctrl setting in an unused
pin group alone would not program it at boot. This board profile therefore:

- Enables UART2 using its existing M1 pinctrl group.
- Disables the competing FIQ debugger for serial ID 2.
- Removes `earlycon=...ff4c0000` and `console=ttyFIQ0` from the inherited
  `/chosen/bootargs` while preserving the other inherited boot arguments.
- Sets **only TX** (GPIO1_B2) to
  `pcfg_pull_up_drv_level_1`; RX (GPIO1_B3) retains `pcfg_pull_up`.
  The existing RV1106 kernel driver encodes level N as
  `(1 << (N + 1)) - 1`, thus Level 1 produces `0x03`.

This intentionally sacrifices UART2/FIQ kernel serial-console diagnostics in
exchange for exclusive application access to the ESP32. Normal USB-based
recovery and logging should be verified before deploying this profile. Do not
enable both drivers on the same UART or restore ttyFIQ0 boot arguments without
reassessing transport ownership.

## Build and board qualification (not yet performed by this PR)

1. Confirm `./build.sh lunch` selected the **EMMC Buildroot Pico Pi**
   profile, not Pro/Max/Ultra/Zero or another board. Check the local generated
   `config/dts_config` and any custom build-time DTS overrides: they must
   not silently shadow this tracked source.
2. Rebuild the selected kernel and firmware image through the existing SDK
   build flow. Inspect the **packaged DTB/boot image**, not just source text,
   to confirm `serial@ff4c0000` is okay, the FIQ debugger is disabled, and
   the two pin configs reflect TX Level 1 / RX unchanged.
3. Flash only a verified image for the exact Pico Pi A W hardware revision.
   No flash, hardware operation or deployment is authorized by this PR itself.
4. After boot, check:
   ```sh
   cat /proc/cmdline
   cat /proc/consoles
   cat /proc/tty/driver/serial
   busybox devmem 0xFF538094 32
   ```
   Expected drive register at `0xFF538094` with unchanged RX:
   **`0x00000103`** (TX `0x03`, RX `0x01`). The source declaration
   alone is *not* proof that the pinctrl driver applied it.
   `/proc/cmdline` must contain neither `earlycon=...` for UART2 nor
   `console=ttyFIQ0`. Some bootloaders can override DT bootargs; if this
   happens, stop qualification and inspect bootloader config before using
   the port. Validate `/dev/ttyS2` is the actual 115200 8N1 UART2 instance.
5. With Controller correctly stopped via its supported S97 lifecycle,
   run the existing **non-actuating** UART2 `T:139` test from
   [backyard-defense issue #121](https://github.com/tc25898/backyard-defense/issues/121).
   Confirm numeric L/R replies, no timeout or framing error, and Controller
   restart afterward. Servo motion, spray and firmware changes to ESP32
   are outside this qualification.
6. Reboot/power cycle and re-check the register and `T:139` response,
   then confirm Bluetooth, Wi-Fi, camera, media and usual boot behavior.
   Compare the effective shipped bootargs against the previous image, paying
   special attention to bootloader modifications.

## Rollback

If UART2, debugging or system boot regresses, restore the prior known-good
board DTS / firmware from recovery rather than relying on a runtime
`devmem` override. Do not change the shared `rv1106-pinctrl.dtsi` or
the pinctrl driver to address this one board's electrical load.

## Performance and memory

No userspace process, worker, polling loop, heap allocation or camera buffer
is added. Only static pinctrl and UART driver initialization change. Enabling
UART2 may consume its normal driver resources/IRQ; actual target memory and
power difference have not been measured. No new automated tests or CI tasks
are added.
