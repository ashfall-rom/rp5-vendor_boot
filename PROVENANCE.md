# Provenance

## Stock input

| | |
|---|---|
| firmware | Retroid Pocket 5 Android 13 `V1.0.0.31` (pre-release, 5.15 GKI) |
| source | `https://github.com/TheGammaSqueeze/Retroid_Pocket_Stock_Firmware/releases/download/Android13_RP5_V1.0.0.31_20241026_190102_user/Android13_RP5_V1.0.0.31_20241026_190102_user.7z` |
| extracted file | `firmware/img/lun4_vendor_boot_a.bin` (slot A; `lun4_vendor_boot_b.bin` is the same bytes) |
| size | 100,663,296 B (padded to the 96 MiB partition) |
| sha256 | `c6e497410af928da86344edf8ffb491651fc79b3a82c130403249b48a1c6d2f4` |

Kernel / modules (not modified, not in `vendor_boot`): `Linux version
5.15.148-android13-8-gbfe94e95c709`, modules `vermagic=5.15.148-gbfe94e95c709`.

## Build tools

| tool | version |
|---|---|
| `dtc` | `DTC 1.7.0-Android-build` (AOSP `prebuilts/kernel-build-tools/linux-x86/bin/dtc`) |
| `unpack_bootimg.py` / `mkbootimg.py` | AOSP `system/tools/mkbootimg` (Android 13) |
| Python | 3.15.0 |

## Output (`vb-oc8-20261115`)

| | |
|---|---|
| image | `vendor_boot-<tag>.img`, content-only |
| size | 12,931,072 B |
| sha256 | `804c2f088a9a16efb1f12775a80fb22b18da148f723f5fb1f64c9edca92f3eee` |
| DTB payload | 3,184,830 B → 3,189,678 B, sha256 `0f912a33bcb0f9557dfa6c97e38291401f8d636d5dff23e23801a5cb5587ccee` |
| active FDT | 2 (`fdt-02`, RP5 board) |
| padded variant (never flash) | 100,663,296 B, sha256 `df049ebd074de58774d8a4db07ced3b65fdefd62dc56516af292816539cea7bf` |

## Verification

- **Reproducible:** two independent passes produced byte-identical DTB payloads
  and byte-identical images (`reproducible: yes`).
- **Per-FDT:** 6 of 7 FDTs carry `qcom,gpu-pwrlevel-bins`; each got 4 patched
  670-top tables (8 entries, 4 steps added, 4 stock steps dropped) and kept its
  1 525 MHz table untouched. The 7th FDT (legacy `qcom,gpu-pwrlevels` only) is
  **byte-identical** to stock.
- **Structural:** every non-GPU node/property in the patched FDTs is unchanged;
  the transform splices only the transformed `qcom,gpu-pwrlevel-bins` nodes back
  into the stock FDT bytes (no `dtc` round-trip of the whole tree).
- **Container:** header fields, vendor ramdisk, ramdisk table and bootconfig
  compare equal to stock (`image-compare.json`).
- **On hardware:** flashed on the RP5 (serial `2f184581`), booted, 1000/940/850/
  765 MHz visible in `gpu_available_frequencies`, GPU cap selectable from the
  AshFall settings page; see `docs/rp5/gpu.md` in the ROM repo.

## Flashing

`vendor_boot` and `vbmeta` are **non-slotted** physical partitions: the flash
runs in the bootloader's fastboot (not fastbootd), without `--slot`, as one
detached `fastboot` process (issue #54). The image is **unsigned**; `vbmeta` is
flashed with `--disable-verity --disable-verification`, exactly as for the GSI
system image. `scripts/device/flash-vendor-boot.sh --stock` restores the stock
`vendor_boot`.

## License

The image wraps Retroid/Qualcomm proprietary content (vendor ramdisk, vendor
modules). The DTB payload is Qualcomm device-tree data distributed with the
GPL kernel. Keep this repository private; do not redistribute the image.
