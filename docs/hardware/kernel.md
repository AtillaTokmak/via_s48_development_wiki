# Linux 4.14 Kernel Architecture: Casper VIA S48

The Casper VIA S48 uses the Linux kernel version **4.14.141** based on MediaTek's AArch64 BSP (`tb8168p1_64_bsp_f716`).

---

## 1. Kernel Architecture

* **Source Branch:** `android16-axionos-compat` (based on `origin/oc-2.1ghz-experimental`)
* **Repository:** `https://github.com/AtillaTokmak/kernel_casper_mt8168`
* **Architecture:** ARM64 (AArch64 little-endian)
* **Defconfig:** `arch/arm64/configs/tb8168p1_64_bsp_f716_defconfig`
* **Compiler:** GNU GCC 9.3.0 / AArch64 Linux Cross Compiler or Android Clang 10.0+

---

## 2. Boot Image Headers & Memory Offsets

The bootloader enforces **Android Boot Image Header Version 2**:

```
BOARD_KERNEL_BASE        := 0x40000000
BOARD_KERNEL_PAGESIZE    := 2048
BOARD_KERNEL_OFFSET      := 0x00080000   (Absolute Address: 0x40080000)
BOARD_RAMDISK_OFFSET     := 0x04000000   (Absolute Address: 0x44000000)
BOARD_TAGS_OFFSET        := 0x00000100   (Absolute Address: 0x40000100)
BOARD_DTB_OFFSET         := 0x00000100   (Absolute Address: 0x40000100)
```

### Kernel Command Line
```
bootopt=64S3,32N2,64N2 buildvariant=user androidboot.selinux=permissive
```

---

## 3. Overclock Subsystems

The 2.1 GHz overclocked kernel introduces three structural modifications:
1. **Device Tree OPP Table (`arch/arm64/boot/dts/mediatek/mt8168.dtsi`):** Adds `opp-2100000000` with 1150 mV core voltage.
2. **CPUFreq Hard Limit (`drivers/cpufreq/mt8168-cpufreq.c`):** Raises `MT8168_MAX_FREQ` to `2100000`.
3. **Thermal Mitigation Curves (`drivers/misc/mediatek/thermal/mt8168/mtk_power_throttle.c`):** Recalibrates dynamic package power dissipation to accommodate sustained 1.8W burst power.
