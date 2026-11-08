# EXP-001: Android 14 GSI Initial Bring-Up

## Metadata
* **Experiment ID:** `EXP-001`
* **Date:** `2026-10-25`
* **Target Android Version:** Android 14 (Upside Down Cake / API 34)
* **ROM Source:** `lineage-21.0-20261020-UNOFFICIAL-arm64_bvN.img`
* **Kernel Version:** Linux 4.14.141 Stock (`Casper C1.03 Baseline`)
* **Vendor Baseline:** Casper VIA S48 Android 10 Stock Vendor
* **Final Result:** **SUCCESS**

---

## 1. Objective & Hypothesis
* **Objective:** Test generic Treble GSI compatibility on MT8168 using Android 14.
* **Hypothesis:** MT8168 Android 10 vendor implementation supports Treble HIDL HALs if FCM Level 4 is satisfied and userdata encryption is bypassed.

---

## 2. Operations Executed
1. Flashed `vbmeta_disabled.img` with `--disable-verity --disable-verification`.
2. Flashed custom DFE boot image removing `fileencryption=aes-256-xts`.
3. Injected `compatibility_matrix.4.xml` into `/system/etc/vintf/`.
4. Flashed LineageOS 21 GSI into `system` partition via Fastbootd:
   ```bash
   fastboot reboot fastboot
   fastboot flash system lineage-21.0-arm64_bvN.img
   fastboot erase userdata
   fastboot reboot
   ```

---

## 3. Observed Behavior
* Little Kernel booted kernel cleanly.
* SurfaceFlinger started Mali-G52 GPU rendering.
* System reached Setup Wizard.
* Wi-Fi and Bluetooth paired successfully.

---

## 4. Lessons Learned
* MT8168 Android 10 vendor HALs are capable of supporting Android 14.
* Disabling File-Based Encryption is mandatory because TrustKernel Keymaster 4.0 cannot handle modern key attestation.
