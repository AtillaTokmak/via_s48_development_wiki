# Patch Management & Architecture

This section documents all patches, code modifications, and configuration adjustments applied across the Casper VIA S48 Android 16 stack.

---

## Patch Index

| Patch ID | Layer | Target Component | Purpose |
| :--- | :--- | :--- | :--- |
| [**`PATCH-01`**](patch-inventory.md#patch-01-mp2-audio-decoder-removal) | Vendor / Framework | `media_codecs_mediatek_audio.xml` | Prevents UBSAN integer underflow crash in `system_server`. |
| [**`PATCH-02`**](patch-inventory.md#patch-02-disable-file-based-encryption-dfe) | Ramdisk / Vendor | `fstab.mt8168` | Bypasses Keymaster 4.0 crypto deadlock on `/data`. |
| [**`PATCH-03`**](patch-inventory.md#patch-03-vintf-fcm-level-4-matrix) | Framework / VINTF | `compatibility_matrix.4.xml` | Bridges Android 10 vendor HALs with Android 16. |
| [**`PATCH-04`**](patch-inventory.md#patch-04-fuse-bpf-userspace-fallback) | System Properties | `build.prop` / `device.mk` | Falls back to userspace FUSE daemon without kernel BPF. |
| [**`PATCH-05`**](patch-inventory.md#patch-05-selinux-permissive-bring-up-mode) | Kernel Command Line | `BOARD_KERNEL_CMDLINE` | Sets permissive mode for development bring-up. |
| [**`PATCH-06`**](patch-inventory.md#patch-06-usb-udc-gadget-binding) | Init / USB | `init.mt8168.usb.rc` | Binds ConfigFS gadget to `11201000.usb`. |
| [**`PATCH-07`**](patch-inventory.md#patch-07-21-ghz-opp-device-tree-table) | Kernel DTS | `mt8168.dtsi` | Adds 2,100,000 kHz operating performance point @ 1.15V. |
| [**`PATCH-08`**](patch-inventory.md#patch-08-cpufreq-driver-frequency-ceiling) | Kernel Driver | `mt8168-cpufreq.c` | Raises driver clock limit to 2.1 GHz. |
| [**`PATCH-09`**](patch-inventory.md#patch-09-thermal-throttle-budget-recalibration) | Kernel Thermal | `mtk_power_throttle.c` | Smooths thermal mitigation curves for sustained 1.8W burst. |
| [**`PATCH-10`**](patch-inventory.md#patch-10-source-integrated-device-and-vendor-trees) | Build System | `device_casper_via_s48` | Production makefiles, overlays, sepolicy, and 570-blob extractor. |
