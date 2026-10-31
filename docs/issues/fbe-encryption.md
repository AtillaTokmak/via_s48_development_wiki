# ISSUE-02: File-Based Encryption (FBE) Deadlock

## Metadata
* **Issue ID:** `ISSUE-02`
* **Affected Component:** `vold` / Keymaster / KeyMint / TEE
* **Target File:** `/fstab.mt8168`
* **Status:** **RESOLVED (DFE Baseline)**

---

## 1. Problem Statement
When booting with stock boot ramdisk flags (`fileencryption=aes-256-xts`, `checkpoint=block`), Android 16 fails to initialize userdata storage. `vold` loops waiting for crypto response and eventually enters a kernel panic or bootloader reboot.

---

## 2. Root Cause Analysis
Android 16 deprecates Keymaster 4.0 HIDL in favor of KeyMint AIDL.
The MT8168 TrustKernel MTEE (`android.hardware.keymaster@4.0-service.mtee`) runs in a legacy secure world enclave that cannot satisfy Android 16 attestation tokens or modern keyblob serialization. Consequently, `vold` cannot retrieve master encryption keys from the hardware keystore.

---

## 3. Resolution
In `device_casper_via_s48/rootdir/fstab.mt8168`, File-Based Encryption requirements are stripped:
```
# From:
/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m,fileencryption=aes-256-xts,checkpoint=block

# To:
/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m
```
After executing `fastboot erase userdata`, the tablet initializes a clean, unencrypted EXT4 filesystem directly on boot.
