# ISSUE-07: Android 16 VNDK Deprecation & Legacy Vendor HAL Dependencies

## Metadata
* **Issue ID:** `ISSUE-07`
* **Subsystem:** Dynamic Linking / VNDK / Treble
* **Severity:** **Build-Time Blocker for Pure Source Builds (GSI Unaffected)**
* **Status:** **ANALYZED & DOCUMENTED (Mitigation Required)**

---

## 1. Problem Statement
When compiling a full device-specific ROM (AxionOS / LineageOS) from source using the extracted MediaTek Android 10 vendor blobs, the dynamic linker flags unresolved symbols during packaging or runtime:
* `libexpat.so`
* `libkeymaster_messages.so`
* `libprotobuf-cpp-lite.so`
* `libmedia_helper.so`
* `android.hardware.wifi.supplicant@1.2.so`

---

## 2. Root Cause Analysis
In Android 10 (API 29), MediaTek built HALs against the Android 10 Vendor Native Development Kit (**VNDK 29**), which provided system-side C++ runtimes (`/system/lib/vndk-29/`).
Starting in Android 14 and fully enforced in Android 16 (API 36), Google completely deprecated VNDK (`BOARD_VNDK_VERSION` is deprecated in Soong). Android 16 platform system images no longer export VNDK-29 stubs by default.

* **Why GSI Boot Succeeded:** Phh-based GSIs (including AxionOS GSI) package backward-compatible VNDK-28/29/30 shim libraries directly in `/system/lib64/vndk-sp-29/` and configure `ld.config.txt` linker namespaces to resolve them for legacy vendor processes.
* **Why Pure Source ROM Builds Need Mitigation:** A standard unmodified AOSP / AxionOS source compilation does not include VNDK-29 targets unless:
  1. Prebuilt VNDK-29 apex/shared libraries (`prebuilts/vndk/v29`) are enabled in the build manifest.
  2. Or missing vendor-side shims are compiled into `vendor/casper/via_s48` as prebuilt vendor libraries.

---

## 3. Recommended Resolution for Source Builds
1. In `device_casper_via_s48/device.mk`, add the legacy VNDK shim packages:
   ```makefile
   # Prebuilt VNDK 29 runtime compatibility for MediaTek HALs
   PRODUCT_PACKAGES += \
       vndk_v29
   ```
2. For standalone HAL builds, provide local prebuilt copies of `libexpat.so`, `libprotobuf-cpp-lite.so`, and `libkeymaster_messages.so` directly within `vendor/casper/via_s48/proprietary/lib64/` and `lib/`.
