# EXP-008: Hardware Fixes (Cameras, LED191 Flash, Stock-Exact Capacitive Touch Keys)

## Metadata
* **Experiment ID:** `EXP-008`
* **Date:** `2026-11-08` (Updated & Finalized `2026-11-20`)
* **Target Android Version:** Android 16 (API Level 36 — AxionOS 2.9 MicroG)
* **Kernel Version:** Linux 4.14.141 Overclocked (2.1 GHz, `kernel_casper_mt8168` branch `android-16` @ `414965654`)
* **Hardware Verified By:** AtillaTokmak (Physical Device Verification)
* **Final Result:** **SUCCESS (Initial Release Ready)**

---

## 1. Scope & Implementation

| Feature | Subsystem / Driver | Verified Outcome |
| :--- | :--- | :--- |
| **Rear Camera** | `gc5025_mipi_raw` (`i2c-5 @ 0x3c`) | Device 0 (`2592×1944`, 5 MP) recognized and operational with clean stock `dtbo`/`vendor`. |
| **Front Camera** | `gc2385sub_mipi_raw` (`i2c-5 @ 0x30`) | Device 1 (`1600×1200`, 2 MP) recognized and operational with clean stock `dtbo`/`vendor`. |
| **Flashlight / Torch** | `flashlights_led191` | Quick Settings flashlight toggle and camera capture flash functional. |
| **Physical Side Keys** | `mtk-kpd` (`10010000.keypad`) | `KEY_VOLUMEDOWN (114)`, `KEY_VOLUMEUP (115)`, `KEY_POWER (116)` mapped cleanly via `Generic.kl`. |
| **Capacitive Touch Keys** | `mtk_gslX680` (`414965654`) | Stock-exact bezel band `Y=1293..1351` (`X=91..149` → `KEY_APPSELECT 580`, `X=411..469` → `KEY_HOMEPAGE 172`, `X=651..709` → `KEY_BACK 158`) verified working on physical hardware. |

---

## 2. Boot Image Artifacts

| Artifact | Size (Bytes) | SHA-256 | Notes |
| :--- | ---: | :--- | :--- |
| `boot_oc_2.1ghz_android16_hwfix_v1_dtbfix.img` | 10,582,016 | `a92375eebd011e3e51788273a5c6446e72570394aee357e38491ecc8e95695a5` | V1 Known-Good baseline (OC + cameras + flash working; old `TPD_HAVE_BUTTON` `key_y=883` did not trigger capacitive keys). |
| **`boot_oc_2.1ghz_android16_v1_plus_capkeys.img`** | **10,586,112** | **`4017d6529fa75581a3417d71b768b34516a8d4e0d93f7611a8001607f8bbf60a`** | **Final Initial Release Candidate (`414965654`):** V1 + stock-exact capacitive keys + complete 69,134-byte 2.1 GHz DTB + DFE ramdisk (`b9cc1674...`). All hardware verified working. |

> **Important Bring-Up Note:** When flashing from experimental states, ensure stock `dtbo-verified.img` and stock Android 10 `vendor` remain intact; corrupted/modified `dtbo` or vendor state causes both `gc5025` and `gc2385` to return `Fail to get sensor ID ffffffff` even on the stock kernel.
