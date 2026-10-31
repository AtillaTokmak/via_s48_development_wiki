# Stock Android 10 Firmware Baseline

## 1. Firmware Package Information

* **Official Factory Package:** `Casper_VIA_S48_0_20220606_C1.03_[WwW-(TimurGsmViP)-Com]`
* **Release Date:** June 6, 2022
* **Build Version:** `C1.03`
* **Android OS Version:** Android 10 (Quince Tart / API Level 29)
* **Security Patch Level:** June 2022
* **Linux Kernel Version:** `4.14.141 #1 SMP PREEMPT 2022`
* **Target VINTF FCM Level:** Level 4 (Android 10)
* **Encryption Scheme:** File-Based Encryption (FBE) via `fileencryption=aes-256-xts` backed by TrustKernel MTEE Keymaster 4.0.

---

## 2. Partition Image Checksums (Factory Dump)

| Partition | File Name | Size (Bytes) | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| **Boot** | `boot.img` | 33,554,432 | `bc31d5e1ca609631502d5ecc53ce9d9db45838d8ad7346d6a74f2dc436e3c311` |
| **Recovery** | `recovery.img` | 33,554,432 | `b54f280a58ad3bb24ad6e84d43be9ee8c4bc684b39130768c66e9bf7b47e2f5b` |
| **Super (Sparse)** | `super.img` | 2,717,575,824 | `3ecf342bb00f13a0fc394d232ee571d784a95ea8d2a1975e5352cbb933ad7802` |
| **Super (Raw)** | `super.raw.img` | 5,890,572,288 | `ebbf43ba11482f9d0d340dee1f5b8a2cf8c6f1997bbbce05be400be3281048eb` |
| **VBMeta** | `vbmeta.img` | 4,096 | `3989c1dae44503698888b5ec20bb6148ee5fbb5c68b75f82410a6a0ca94ff9cf` |
