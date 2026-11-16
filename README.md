<p align="center">
  <img src="assets/brand/ashfall-icon.svg" width="80" height="80" alt="AshFall">
</p>

<h1 align="center">rp5-vendor_boot</h1>

`vendor_boot` artifacts for the Retroid Pocket 5 (SM8250 "kona"), as AshFall
boots it.

AshFall runs a **GSI on the stock 5.15 GKI vendor** and does **not** build a
kernel. The only thing changed inside `vendor_boot` is the **Adreno 650 GPU
power-level table** in the board DTB (phase 6, `docs/phase6.md` §2 of the ROM
repo). This repository exists so the exact image that was flashed is a named,
hash-pinned artifact instead of a path under `~/ashfall/artifacts` on the build
host.

## What `vendor_boot` contains — and what it does not

A boot header v4 container holding:

- the **vendor ramdisk** (stock modules, stock fstab),
- a **DTB payload that is not a single DTB**: 7 concatenated FDTs, one per board
  variant, selected at boot by `qcom,msm-id` / `qcom,pmic-id`. The RP5 board is
  `fdt-02` (`msm-id` `0x164`/`0x20001`, `pmic-id` `1e 20 1f 21`),
- the bootconfig and the v4 ramdisk/dtb tables.

The **kernel lives in `boot.img`** and is not in this repository and not in
`vendor_boot`. So are the 222 vendor modules.

## The change

Exactly one property, under `qcom,kgsl-3d0@3d00000`:

```
qcom,gpu-pwrlevel-bins
```

Every speed-bin table whose stock top is 670 MHz (`-0`, `-1`, `-3`, `-4`) is
**replaced** by 8 entries:

| MHz | `qcom,level` | corner |
|---|---|---|
| 1000 | 416 (`0x1a0`) | TURBO_L1 |
| 940 | 384 (`0x180`) | TURBO |
| 850 | 384 (`0x180`) | TURBO |
| 765 | 352 (`0x160`) | NOM_L3 |
| 670 | 320 (`0x140`) | NOM_L1 (stock) |
| 587 | 256 (`0x100`) | NOM (stock) |
| 305 | 64 (`0x40`) | MIN (stock) |
| 0 | — | off |

`qcom,level` selects the GPU rail voltage corner, so it is the voltage half of
the overclock. `qcom,initial-pwrlevel` becomes 6 (the 305 MHz boot entry) and
`qcom,throttle-pwrlevel` stays stock. The 525-top bin (`-2`) and the one variant
without a bins table stay byte-identical.

It is a **replacement**, not an extension: the stock 525 / 490 / 441.6 / 400 MHz
steps are dropped because RPMh only covers `num_pwrlevels + 1` `gfx.lvl` votes
(12 entries returned `-ERANGE`, issue #116).

The DTB only makes the higher steps *available*. The runtime cap is still
`ashfall-hwd` writing devfreq `max_freq`; nothing here forces a clock.

## Rules

1. **Content-only images only**: 12,931,072 B. The zero-padded 100,663,296 B
   variant wedges the RP5 ABL on `Sending 'vendor_boot'` (issue #54). Never
   publish or flash it.
2. **No binaries in git.** Images are release assets; `.gitignore` enforces
   it. The stock image stays behind its URL + sha256 (`PROVENANCE.md`).
3. **Tags are immutable.** `vb-<table>-<yyyymmdd>`; never move a tag or upload
   different bytes under an existing one.
4. **Private.** The vendor ramdisk and vendor modules are proprietary; this
   repository is not public.

## Releases

| tag | image sha256 | size | notes |
|---|---|---|---|
| `vb-oc8-20261115` | `804c2f088a9a16efb1f12775a80fb22b18da148f723f5fb1f64c9edca92f3eee` | 12,931,072 | first 8-entry OC table (1000/940/850/765/670/587/305/0), flashed and booted on the RP5 |

Assets per release:

- `vendor_boot-<tag>.img` — the content-only image to flash
- `vendor_boot-<tag>.img.sha256`
- `manifest-<tag>.json` — sizes, hashes, per-FDT verification, tools
- `dts-<tag>.tar.gz` — every board FDT, stock and patched
- `summary-<tag>.txt`, `SHA256SUMS-<tag>`

## Building and publishing

The source of truth is the ROM repository:

- `scripts/gpu/build-vendor-boot.sh` — stock image + `dtb_oc.py` → patched DTB →
  repacked image, two deterministic passes, artifacts under `out/gpu/<id>/`
- `scripts/gpu/publish-vendor_boot.sh` — creates the tag and uploads the assets
  here
- `docs/rp5/gpu.md` — measurements, table rationale, recovery

```bash
scripts/gpu/build-vendor-boot.sh --id vb-oc8-20261115
scripts/gpu/publish-vendor_boot.sh --dir out/gpu/vb-oc8-20261115 --tag vb-oc8-20261115
```

## Consuming a release

`config/ashfall.conf` in the ROM repo pins `VENDOR_BOOT_TAG` and
`VENDOR_BOOT_SHA256`. `scripts/device/flash-vendor-boot.sh --release <tag>`
downloads the asset, verifies the hash and flashes it through the one-session
fastboot batch (`AGENTS.md`, issue #54). `--stock` restores the stock
`vendor_boot`.

See `PROVENANCE.md` for the stock URL/hash, tool versions, the exact table and
the verification results.
