# LG G8 (alphaplus) port

This branch adapts the proven iOS 16.7 `ios-sweet` path to the Verizon LG G8
ThinQ LM-G820UM (SM8150). It is an early bring-up branch, not a flashable ROM.

## Confirmed hardware profile

- platform: `lge,alphaplus`, SoC `qcom,sm8150`
- CPUs: 8 x `qcom,kryo485`, MPIDRs `0x000` through `0x700`
- GICv3: distributor `0x17a00000`, redistributors `0x17a60000`
- architectural timer: 19.2 MHz, virtual timer INTID 19
- continuous-splash framebuffer: `0x9d200000`, size `0x01200000`
- framebuffer geometry: 1440 x 3120, BGRA stride 5760
- UFSHCI: `0x01d84000`, size `0x3000`
- conservative high-memory window: `0x140000000` + `0x40000000`

Generate the Apple Device Tree with:

```sh
python3 tools/mkadt.py out/lg-g8.adt --device lg-g8 --highmem
```

The iPhone X identity remains intentional because the iOS 16.7 userspace is
the D22AP/iPhone10,3 build. The Qualcomm hardware identity is added after the
Apple identity so XNU drivers can match the LG platform.

## Safety state

The Xiaomi PMIC key driver, Goodix/GENI touch driver, and Sweet USB probe are
disabled by default. Their MMIO layouts are not valid evidence for LG hardware.
UFS is retained because the controller base is the same and the driver obtains
the root partition from GPT, but it still needs a read-only phone test before
any iOS image is written to `userdata`.

The early console maps the 32 MiB block beginning at `0x9c000000`; the actual
LG framebuffer begins 18 MiB into that block. Its final virtual alias therefore
includes the `0x01200000` offset.

## Required gates before a ROM ZIP

1. CI must apply all patches and produce a `kernel.development.sweet` artifact.
2. A temporary boot must show early-console progress on the LG framebuffer.
3. The kernel must enumerate UFS and find `userdata` read-only.
4. HFS must mount and reach `launchd` without writing the iOS image first.
5. Only after those gates may the verified sparse iOS root be flashed.
6. A recovery ZIP is packaging for a proven kernel/image pair, not a substitute
   for kernel bring-up.

Do not flash engineering ABL or permanently install the previously tested TWRP
images. The original LG ABL and stock Android boot are the known-good recovery
path for this phone.
