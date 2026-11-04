# Upstream Sources & References

This page documents the upstream source repositories, original Generic System Image (GSI) projects, toolchains, and community resources referenced across the Casper VIA S48 development knowledge base.

---

## 1. Upstream Android 16 Generic System Images (GSI)

The tested Android 16 operating systems on the Casper VIA S48 are **unofficial GSI adaptations**.

### AxionOS 2.9 (Android 16 / API 36)
* **Upstream GSI Repository:** [`https://github.com/Doze-off/axion_aosp/`](https://github.com/Doze-off/axion_aosp/)
* **Original GSI Developer:** **Doze-off** and upstream AOSP / AxionOS contributors.
* **Casper VIA S48 Adaptation & Maintenance:** **AtillaTokmak** (hardware bring-up, MP2 audio UBSAN crash remediation, FUSE-BPF bypass, DFE fstab configuration, physical hardware validation, and technical documentation).
* **Adaptation Classification:** Unofficial GSI adaptation.

### Evolution X 11.11 (Android 16 / API 36)
* **Upstream GSI Repository:** [`https://github.com/Doze-off/EvoX_treble`](https://github.com/Doze-off/EvoX_treble)
* **Original GSI Developer:** **Doze-off** and upstream Evolution X contributors.
* **Casper VIA S48 Adaptation & Maintenance:** **AtillaTokmak** (hardware bring-up, MediaTek 4.14 kernel panic remediation in `mtp_free_inst()` / `twrp.sh` configfs removal, DFE ramdisk integration, physical hardware validation, and technical documentation).
* **Adaptation Classification:** Unofficial GSI adaptation.

---

## 2. Upstream Source Trees & Manifests

* **AxionOS AOSP Manifest:** `https://github.com/AxionAOSP/manifest.git` (Branch: `16.0`)
* **LineageOS Android Source:** `https://github.com/LineageOS/android`
* **MediaTek Open Source Kernel:** `https://gitlab.com/mediatek`
* **Android Generic System Images (GSI):** `https://github.com/phhusson/treble_experimentations`

---

## 3. Platform & Partition Toolchains

* **Android Verified Boot Tools:** `external/avb/avbtool` in AOSP
* **Dynamic Partitions Tools:** `system/extras/partition_tools/lpmake` in AOSP
