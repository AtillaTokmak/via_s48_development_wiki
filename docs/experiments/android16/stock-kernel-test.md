# EXP-004: Android 16 on Stock 2.0 GHz Kernel

## Metadata
* **Experiment ID:** `EXP-004`
* **Date:** `2026-10-31`
* **Target Android Version:** Android 16 (API Level 36)
* **ROM Source:** AxionOS 2.9 (Patched GSI)
* **Kernel Version:** `Linux version 4.14.141 #1 SMP PREEMPT 2022` (Stock Casper C1.03)
* **Boot Image:** `boot_gsi.img` (Stock Kernel + Stock DTB + DFE Ramdisk)
* **Final Result:** **SUCCESS (Baseline Performance)**

---

## 1. Objective
Verify whether Android 16 boots and operates reliably using the completely unmodified stock 2.0 GHz MediaTek Linux 4.14.141 kernel paired with the DFE patched initramfs.

---

## 2. Telemetry & CPU Frequencies
```bash
adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
# Output: 500000 650000 800000 1000000 1200000 1400000 1600000 1800000 2000000

adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq
# Output: 2000000
```

---

## 3. Findings
* The stock kernel successfully drives Android 16 graphics, touch, audio, hardware buttons (`mtk-kpd` + `mtk-tpd`), and cameras (`gc5025_mipi_raw` + `gc2385sub_mipi_raw`).
* Peak single-core and multi-core performance is constrained to 2.0 GHz.
* Confirmed that **zero kernel C/DTS source patches** are required to boot Android 16; all Android 16 bring-up fixes reside in the boot header (`androidboot.selinux=permissive`), ramdisk (`fstab.mt8168` DFE + AVB removal + `# product`), `vbmeta_disabled.img`, and Android userspace (`rw-system.sh`, `compatibility_matrix.4.xml`, `build.prop`, `adbd` symlink).

---

## 4. Bit-Identical Reproduction & Verification (November 19–20, 2026)
* Rebuilt `boot_stock_android16.img` from stock Casper C1.03 `boot.img` (`kernel` `588d4070...` + `dtb` `fef4c430...`) and the Android 16 DFE ramdisk (`b9cc1674...`) using exact Boot v2 offsets (`0x40080000` / `0x55000000` / `0x54000000`, `os_patch_level=2020-12`).
* Produced a **100% bit-identical match** to historical `boot_gsi.img` (`10,072,064` bytes, SHA-256: `1365822690a638c1947183e18a9ba38b03c2d7cd3c9342f14055ddd72bb2decb`).
* Verified booting both **Evolution X Android 16** and **AxionOS 2.9 MicroG Android 16** on physical hardware; confirmed both rear & front cameras work once clean stock `dtbo-verified.img` and `vendor` are present.
