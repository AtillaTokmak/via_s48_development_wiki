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
* The stock kernel successfully drives Android 16 graphics, touch, and audio.
* Peak single-core and multi-core performance is constrained to 2.0 GHz.
* Confirmed that kernel-level overclocking is not a strict prerequisite for booting Android 16, but an optional performance enhancement.
