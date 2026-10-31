# Device Tree Architecture & Maintenance

Repository: `https://github.com/AtillaTokmak/device_casper_via_s48`

---

## 1. Key Component Files

* **`BoardConfig.mk`:** Hardware platform definitions, dynamic partition sizes (`5,890,572,288` bytes super), kernel boot v2 headers, sepolicy directory declarations.
* **`device.mk`:** Declares proprietary vendor blob inheritance, tablet RRO overlays, init script mappings, and modern Android 16 property flags.
* **`AndroidProducts.mk`:** Product registration and lunch targets (`axion_via_s48-userdebug`).
* **`axion_via_s48.mk`:** Product definition using `inherit-product-if-exists` across AxionOS, LineageOS, or pure AOSP tablet bases.
* **`rootdir/fstab.mt8168`:** Clean DFE userdata filesystem mount definition.
* **`sepolicy/vendor/`:** Vendor domain rules and file contexts.
* **`vintf/`:** Target FCM Level 4 matrix and device manifest.
