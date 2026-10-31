# Android Verified Boot (AVB 2.0) Architecture

## 1. AVB Structure on Casper VIA S48

The tablet implements Google AVB 2.0 with three dedicated descriptor partitions:
* `vbmeta`: Contains root keys, rollback indexes, and descriptor hashes for `boot` and `dtbo`.
* `vbmeta_system`: Contains the hash tree metadata for the `system` partition.
* `vbmeta_vendor`: Contains the hash tree metadata for the `vendor` and `product` partitions.

---

## 2. Disabling dm-verity and Verification

To boot custom kernels, modified GSIs, or custom system partitions, AVB verification flags must be cleared:

```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta_disabled.img
```

The resulting `vbmeta_disabled.img` has the flag byte set to `0x02` (`AVB_VBMETA_IMAGE_FLAGS_HASHTREE_DISABLED`), permitting the Little Kernel (LK) bootloader to bypass signature enforcement.
