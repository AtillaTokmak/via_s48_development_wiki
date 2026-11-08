# EXP-002: Android 15 crDroid Bootloop Investigation

## Metadata
* **Experiment ID:** `EXP-002`
* **Date:** `2026-10-28`
* **Target Android Version:** Android 15 (Vanilla Ice Cream / API 35)
* **ROM Source:** `crDroid-11.0-20261022-arm64_bgN.img`
* **Kernel Version:** Linux 4.14.141 Stock
* **Final Result:** **FAILED (Bootloop / Soft Brick)**

---

## 1. Objective & Hypothesis
* **Objective:** Test Android 15 crDroid GSI on MT8168.
* **Hypothesis:** Android 15 will boot with the same DFE boot image used for Android 14.

---

## 2. Observed Behavior & Crash Trace
The tablet showed the crDroid boot animation for approximately 40 seconds before abruptly rebooting back into Little Kernel fastboot.

### Logcat Excerpt via Early USB Debugging
```
E vold    : Failed to mount /mnt/user/0/emulated: Function not implemented
E vold    : FUSE-BPF initialization error: bpf_prog_load failed (Operation not permitted)
F libc    : Fatal signal 6 (SIGABRT), code -1 (SI_QUEUE) in tid 421 (vold)
```

---

## 3. Root Cause Analysis
Android 15 tightened FUSE-BPF enforcement for external/internal storage virtualization. Linux 4.14 lacks in-tree kernel FUSE-BPF driver support, causing `vold` to crash when attempting to attach eBPF filters to the storage daemon.

---

## 4. Recovery Steps
Re-flashed working Android 14 system partition via Fastbootd to restore operation.
Documented need for `persist.sys.fuse.bpf.override=false` properties for future versions.
