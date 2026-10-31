# EXP-005: Android 16 with 2.1 GHz Overclocked Kernel

## Metadata
* **Experiment ID:** `EXP-005`
* **Date:** `2026-10-31`
* **Target Android Version:** Android 16 (API Level 36)
* **ROM Source:** AxionOS 2.9 (Build `BP4A.251205.006`)
* **Kernel Version:** `Linux version 4.14.141 (atilla@fedora) (gcc version 9.3.0) #10 SMP PREEMPT Sun Oct 18 17:44:57 +03 2026`
* **Boot Image:** `boot_oc_2.1ghz_android16.img`
* **Final Result:** **SUCCESS (Current Verified Baseline)**

---

## 1. Objective & Hypothesis
* **Objective:** Test the author's custom 2.1 GHz overclocked Linux 4.14 kernel on Android 16.
* **Hypothesis:** The custom kernel will deliver a 5% CPU clock increase (2,100,000 kHz) without causing thermal panics or driver instability on Android 16, provided the ramdisk uses DFE flags.

---

## 2. Boot Image Assembly
The 32MB Ventoy boot image (`boot_via_s48_32m_oc_2.1ghz_standard.img`) had previously hung because its bundled ramdisk was identical to stock C1.03 (enforcing FBE).

We unpacked the Ventoy image, extracted the 2.1 GHz kernel binary and DTB, and repacked them with our verified DFE ramdisk:

```bash
unpack_bootimg --boot_img boot_via_s48_32m_oc_2.1ghz_standard.img --out oc_work
mkbootimg \
    --kernel oc_work/kernel \
    --ramdisk ramdisk_dfe.cpio.gz \
    --dtb oc_work/dtb \
    --cmdline "bootopt=64S3,32N2,64N2 androidboot.selinux=permissive" \
    --base 0x40000000 \
    --kernel_offset 0x00080000 \
    --ramdisk_offset 0x04000000 \
    --tags_offset 0x00000100 \
    --dtb_offset 0x00000100 \
    --pagesize 2048 \
    --header_version 2 \
    -o boot_oc_2.1ghz_android16.img
```

---

## 3. Live Telemetry & Verification
```bash
adb shell cat /sys/devices/system/cpu/online
# 0-3 (All 4 cores online)

adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
# 500000 650000 800000 1000000 1200000 1400000 1600000 1800000 2000000 2100000

adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
# 2100000 (Scaled under load)
```

Visual proof confirmed via framebuffer grab: `axion_android16_oc21.png`.
Core voltage rail measured steady at 1.15V. Zero thermal throttle drops during standard navigation.
