# DTB sources

`active-stock.dts` and `active-oc.dts` are the **RP5 board FDT** (`fdt-02`,
`msm-id` `0x164`/`0x20001`, `pmic-id` `1e 20 1f 21`) decompiled from the stock
`vendor_boot` and from the patched image, respectively.

They are the reviewable source of the change: diff them to see exactly what the
overclock does. Everything outside `qcom,gpu-pwrlevel-bins` (and the
`qcom,initial-pwrlevel` / `qcom,throttle-pwrlevel` scalars) is unchanged in the
binary; the DTS text can still differ in formatting because `dtc` re-emits the
whole tree.

```bash
diff -u dts/active-stock.dts dts/active-oc.dts
```

The full set — all 7 FDTs of the payload, before and after — ships as
`dts-<tag>.tar.gz` on each release. The other FDTs are for the same SoC on
different boards (they are patched identically for consistency, but the RP5
never selects them).

Regenerate both files with `scripts/gpu/build-vendor-boot.sh` in the ROM repo;
it writes `active-stock.dts` / `active-oc.dts` into the artifact directory.
