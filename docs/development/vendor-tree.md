# Vendor Extraction Framework & Makefiles

Repository: `https://github.com/AtillaTokmak/vendor_casper_via_s48`

---

## 1. Extraction System Architecture

The Casper VIA S48 vendor extraction system extracts **570 verified proprietary blobs** directly from either a connected device via ADB or a stock firmware dump directory.

### Core Files
* **`proprietary-files.txt`:** Master categorized blob list across Graphics, Audio, MediaCodecs, Wi-Fi, Bluetooth, GPS, Cameras, Sensors, Power, and Security.
* **`extract-files.sh`:** Handles blob pulling, path validation, and symlink preservation.
* **`setup-makefiles.sh`:** Parses extracted blobs and generates `via_s48-vendor.mk` with `PRODUCT_COPY_FILES` mapping to `$(TARGET_COPY_OUT_VENDOR)`.
* **`BoardConfigVendor.mk`:** Injects vendor Soong namespaces into the Android build environment.
