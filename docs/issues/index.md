# Technical Issue & Blocker Directory

This section provides in-depth technical investigations, root-cause analyses, crash stack traces, and verified resolutions for all major compatibility blockers encountered during the Android 16 bring-up on Casper VIA S48.

---

## Issue Directory

| Issue ID | Subsystem | Blocker Level | Description | Status |
| :--- | :--- | :--- | :--- | :--- |
| [**`ISSUE-01`**](system-server-mp2-crash.md) | Audio / MediaCodec | **Critical Blocker** | UBSAN arithmetic underflow crash in `system_server` on `audio/mpeg-L2`. | **RESOLVED (Vendor Overlay)** |
| [**`ISSUE-02`**](fbe-encryption.md) | Storage / Crypto | **Critical Blocker** | File-Based Encryption deadlock due to Keymaster 4.0 deprecation. | **RESOLVED (DFE Baseline)** |
| [**`ISSUE-03`**](vintf-fcm4.md) | VINTF / Treble | **Critical Blocker** | Framework Compatibility Matrix mismatch (Android 10 vendor vs Android 16). | **RESOLVED (Matrix 4)** |
| [**`ISSUE-05`**](selinux-compatibility.md) | Security / Sepolicy| **Major Blocker** | Missing platform policy rules for legacy MediaTek vendor domains. | **MITIGATED (Permissive)**|
| [**`ISSUE-07`**](vndk-deprecation.md) | Linking / VNDK | **Build Blocker** | Legacy Android 10 vendor HAL dependencies on deprecated VNDK-29 libraries. | **RESOLVED (VNDK-29 Blobs)** |

---

## Archived Issues
Historical issues encountered during initial bring-up experiments are archived in [`docs/archived/`](../archived/).
