# Casper VIA S48 (MT8168 / F716) Development Wiki

> **Project status:** Unofficial Casper VIA S48 research and development.
> Both tested Android 16 operating systems—**AxionOS 2.9** and **Evolution X 11.11**—are **unofficial GSI adaptations**.
> AxionOS 2.9 Android 16 serves as the working GSI baseline. Native LineageOS ROM and Recovery builds are not claimed as completed.
> Historical PixelOS work is archived at [PixelOS Android 16](docs/archived/pixelos-android16.md).
> Experimental reports are not installation instructions unless independently validated.

Welcome to the central technical knowledge base and development wiki for the **Casper VIA S48** tablet.
This repository documents hardware reverse engineering, Linux 4.14 kernel overclocking, Android 16 (AxionOS 2.9 & Evolution X 11.11 / API 36) bring-up, proprietary vendor extraction, and device tree maintenance.

---

## Upstream GSI Attribution & Adaptation Credits

Both Android 16 systems evaluated and running on the Casper VIA S48 are **unofficial GSI adaptations**:

* **Evolution X 11.11 (Android 16 / API 36):**
  * **Upstream GSI Source:** [`https://github.com/Doze-off/EvoX_treble`](https://github.com/Doze-off/EvoX_treble)
  * **Original GSI Development:** Credited to **Doze-off** and upstream Evolution X contributors.
  * **Casper VIA S48 Adaptation & Maintenance:** Credited to **AtillaTokmak** for hardware adaptation, MediaTek 4.14 kernel panic remediation (`mtp_free_inst()` / `twrp.sh` configfs removal), DFE ramdisk integration, physical hardware validation, and technical documentation.
* **AxionOS 2.9 (Android 16 / API 36):**
  * **Upstream GSI Source:** [`https://github.com/Doze-off/axion_aosp/`](https://github.com/Doze-off/axion_aosp/)
  * **Original GSI Development:** Credited to **Doze-off** and upstream AOSP / AxionOS contributors.
  * **Casper VIA S48 Adaptation & Maintenance:** Credited to **AtillaTokmak** for hardware bring-up, MP2 audio UBSAN crash remediation, FUSE-BPF bypass, DFE fstab configuration, physical testing, and technical documentation.

---

## 1. Quick Navigation

* **Hardware & Board Architecture:** [MT8168 Architecture](docs/hardware/mt8168.md) | [Partition Table](docs/hardware/partition-layout.md) | [Kernel Architecture](docs/hardware/kernel.md)
* **Experiments Journal:** [Android 16 AxionOS Bring-Up](docs/experiments/android16/axionos-bringup.md) | [Android 16 Evolution X USB Fix](docs/experiments/android16/evox-usb-gadget-kernel-panic.md) | [2.1 GHz Overclock Verification](docs/experiments/android16/oc-2.1ghz-test.md) | [Experiment Index](docs/experiments/index.md)
* **Critical Issues & Fixes:** [MP2 Audio Crash](docs/issues/system-server-mp2-crash.md) | [FBE Crypto Deadlock](docs/issues/fbe-encryption.md) | [VINTF FCM 4](docs/issues/vintf-fcm4.md)
* **Build Integration:** [Device Tree](docs/development/device-tree.md) | [Vendor Tree](docs/development/vendor-tree.md) | [Kernel Compilation](docs/development/kernel-build.md)
* **References:** [Official Repositories](docs/references/repositories.md) | [Upstream Sources & Attribution](docs/references/upstream-sources.md)

---

## 2. Core Repository Ecosystem

| Repository | Role | Upstream / Target Branch |
| :--- | :--- | :--- |
| [**`kernel_casper_mt8168`**](https://github.com/AtillaTokmak/kernel_casper_mt8168) | Linux 4.14.141 Kernel Source with 2.1 GHz Overclock | `android-16` |
| [**`device_casper_via_s48`**](https://github.com/AtillaTokmak/device_casper_via_s48) | Android 16 Device Tree (BoardConfig, makefiles, sepolicy) | `master` |
| [**`vendor_casper_via_s48`**](https://github.com/AtillaTokmak/vendor_casper_via_s48) | Proprietary 570-Blob Extraction Framework & Makefiles | `master` |
| [**`via_s48_development_wiki`**](https://github.com/AtillaTokmak/via_s48_development_wiki) | Technical Documentation, Experiment Records & Wiki | `master` |

---

## 3. High-Level Summary of Findings

1. **Android 16 Boot Animation Hang:** Caused by `StorageManagerService.handleSystemReady()` executing `MediaCodecList` queries. The legacy MediaTek audio decoder `OMX.MTK.AUDIO.DECODER.MP2` triggered an unsigned integer underflow abort (`__ubsan_handle_sub_overflow_minimal_abort`) in Android 16 `libmedia_codeclist_capabilities.so`. Stripping the MP2 XML entry completely resolved the crash.
2. **Userdata Crypto Deadlock:** Android 16 deprecated Keymaster 4.0 HIDL in favor of KeyMint AIDL. Stock `fstab.mt8168` enforcing `fileencryption=aes-256-xts` prevented `vold` from mounting `/data`. Configuring Disabled File-Based Encryption (DFE) allows clean boot and 24 GB usable storage.
3. **2.1 GHz Kernel Overclock:** Operating Performance Point (OPP) table in `mt8168.dtsi` expanded to `2,100,000 kHz` at `1.15V`. When paired with an Android 16 DFE ramdisk, all 4 Cortex-A53 cores dynamically scale up to 2.1 GHz in the reported frequency-scaling tests. Long-duration thermal stability has not been independently established.
