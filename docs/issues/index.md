# Technical Issue & Blocker Directory

This section provides in-depth technical investigations, root-cause analyses, crash stack traces, and verified resolutions for all major compatibility blockers encountered during the Android 16 bring-up on Casper VIA S48.

---

## Issue Directory

| Issue ID | Subsystem | Blocker Level | Description | Status |
| :--- | :--- | :--- | :--- | :--- |
| [**`ISSUE-01`**](system-server-mp2-crash.md) | Audio / MediaCodec | **Critical Blocker** | UBSAN arithmetic underflow crash in `system_server` on `audio/mpeg-L2`. | **RESOLVED** |
| [**`ISSUE-02`**](fbe-encryption.md) | Storage / Crypto | **Critical Blocker** | File-Based Encryption deadlock due to Keymaster 4.0 deprecation. | **RESOLVED (DFE)** |
| [**`ISSUE-03`**](vintf-fcm4.md) | VINTF / Treble | **Critical Blocker** | Framework Compatibility Matrix mismatch (Android 10 vendor vs Android 16). | **RESOLVED** |
| [**`ISSUE-04`**](fuse-bpf.md) | Storage / eBPF | **Critical Blocker** | Android 16 FUSE-BPF requirement unsupported by Linux 4.14 kernel. | **RESOLVED (Fallback)**|
| [**`ISSUE-05`**](selinux-compatibility.md) | Security / Sepolicy| **Major Blocker** | Missing platform policy rules for legacy MediaTek vendor domains. | **MITIGATED (Permissive)**|
| [**`ISSUE-06`**](usb-adb-initialization.md) | USB / Debugging | **Diagnostic Blocker** | USB ConfigFS gadget failed to bind to `11201000.usb` peripheral controller. | **RESOLVED** |
| [**`ISSUE-07`**](vndk-deprecation.md) | Linking / VNDK | **Build Blocker** | Legacy Android 10 vendor HAL dependencies on deprecated VNDK-29 libraries. | **DOCUMENTED (Mitigation Defined)** |
