# Source Tree Integration & Namespace Resolution

## 1. Directory Tree Alignment

To compile Android 16 cleanly, repositories must be linked under the standard AOSP folder hierarchy:

```
android_source/
├── device/casper/via_s48/      -> device_casper_via_s48
├── vendor/casper/via_s48/      -> vendor_casper_via_s48
└── kernel/casper/mt8168/       -> kernel_casper_mt8168 (android16-axionos-compat)
```

---

## 2. Soong Namespace Management

`vendor/casper/via_s48` declares:
```makefile
PRODUCT_SOONG_NAMESPACES += vendor/casper/via_s48
```
This isolates proprietary prebuilt module definitions, ensuring they do not collide with open-source AOSP modules during build blueprint compilation.
