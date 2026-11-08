# EXP-008: Hardware Fixes (Front Camera, LED191 Flash, Capacitive Touch Keys)

## Metadata
* **Experiment ID:** `EXP-008`
* **Date:** `2026-11-08`
* **Target Android Version:** Android 16 (API Level 36)
* **Kernel Version:** Linux 4.14.141 Overclocked (2.1 GHz)
* **Hardware Verified By:** AtillaTokmak (User Physical Device Verification)
* **Final Result:** **SUCCESS**

---

## 1. Scope & Implementation

| Feature | Subelement / Driver | Verified Outcome |
| :--- | :--- | :--- |
| **Front Camera** | `gc2385sub_mipi_raw` | Device 1 recognized in Camera HAL, preview and capture fully operational. |
| **Flashlight / Torch** | `flashlights_led191` | Quick Settings flashlight toggle and camera capture flash functional. |
| **Capacitive Touch Keys** | `mtk_gslX680` | Physical Back, Home, and App Switch keys under display operational with proper input events. |

---

## 2. Boot Image Artifact
* **File:** `boot_oc_2.1ghz_android16_hwfix_final.img`
* **SHA-256:** `a92375eebd011e3e51788273a5c6446e72570394aee357e38491ecc8e95695a5`
* **Size:** 10,582,016 bytes (Header v2 + DFE ramdisk + complete 69,134-byte DT table)
