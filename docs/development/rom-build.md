# AxionOS Android 16 ROM Compilation

## 1. Setting Up the Local Manifest

Create `.repo/local_manifests/casper_via_s48.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <project path="device/casper/via_s48" name="AtillaTokmak/device_casper_via_s48" remote="github" revision="master" />
    <project path="vendor/casper/via_s48" name="AtillaTokmak/vendor_casper_via_s48" remote="github" revision="master" />
    <project path="kernel/casper/mt8168" name="AtillaTokmak/kernel_casper_mt8168" remote="github" revision="android-16" />
</manifest>
```

---

## 2. Compilation Commands
```bash
source build/envsetup.sh
lunch axion_via_s48-userdebug
m bacon -j$(nproc)
```

Target flashable images will be generated in `out/target/product/via_s48/`.
