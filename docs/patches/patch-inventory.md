# Comprehensive Patch Inventory: Casper VIA S48

---

### PATCH-01: MP2 Audio Decoder Removal
* **Domain:** System / Vendor
* **Target File:** `/vendor/etc/media_codecs_mediatek_audio.xml`
* **Diff:**
```diff
--- a/vendor/etc/media_codecs_mediatek_audio.xml
+++ b/vendor/etc/media_codecs_mediatek_audio.xml
@@ -10,6 +10,4 @@
-        <MediaCodec name="OMX.MTK.AUDIO.DECODER.MP2" type="audio/mpeg-L2">
-            <Quirk name="needs-discrete-audio-sample-sizes" />
-        </MediaCodec>
```

---

### PATCH-02: Disable File-Based Encryption (DFE) & GSI `fstab.mt8168` Normalization
* **Domain:** Boot Ramdisk / Vendor
* **Target File:** `/fstab.mt8168` (Ramdisk SHA-256: `b9cc1674e6c44238b120f937a28a0d3c7e010b1b7a5811d1b955146c03f86ff1`, `791,547` bytes)
* **Diff:**
```diff
--- a/fstab.mt8168
+++ b/fstab.mt8168
@@ -6,14 +6,14 @@
-system /system ext4 ro wait,,avb=vbmeta_system,logical,first_stage_mount,avb_keys=/avb/q-gsi.avbpubkey:/avb/r-gsi.avbpubkey:/avb/s-gsi.avbpubkey
+system /system ext4 ro wait,logical,first_stage_mount
-vendor /vendor ext4 ro wait,,avb,logical,first_stage_mount
+vendor /vendor ext4 ro wait,logical,first_stage_mount
-product /product ext4 ro wait,,avb,logical,first_stage_mount
+# product /product ext4 ro wait,,avb,logical,first_stage_mount
@@ -23,7 +23,7 @@
-/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m,checkpoint=block,fileencryption=aes-256-xts
+/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m
@@ -51,7 +51,7 @@
-/dev/block/platform/bootdevice/by-name/vbmeta_system /vbmeta_system emmc defaults first_stage_mount,nofail,,avb=vbmeta
+/dev/block/platform/bootdevice/by-name/vbmeta_system /vbmeta_system emmc defaults first_stage_mount,nofail
```
* **Boot Header v2 Packaging Note:** Must use exact stock MT8168 load offsets (`base=0x40000000`, `kernel_offset=0x00080000`, `ramdisk_offset=0x15000000`, `tags_offset=0x14000000`, `dtb_offset=0x14000000`, `pagesize=2048`, `os_version=10.0.0`, `os_patch_level=2020-12`).

---

### PATCH-03: VINTF FCM Level 4 Matrix
* **Domain:** Framework VINTF
* **Target File:** `device/casper/via_s48/vintf/compatibility_matrix.4.xml`
* **Mechanism:** Defined in `BoardConfig.mk` via `DEVICE_FRAMEWORK_COMPATIBILITY_MATRIX_FILE`.

---

### PATCH-04: FUSE-BPF Userspace Fallback
* **Domain:** System Properties
* **Target File:** `device/casper/via_s48/device.mk`
* **Code:**
```makefile
PRODUCT_SYSTEM_PROPERTIES += \
    persist.sys.fuse.bpf.override=false \
    persist.sys.phh.bpf=false
```

---

### PATCH-05: SELinux Permissive Bring-Up Mode
* **Domain:** Kernel Command Line
* **Target File:** `device/casper/via_s48/BoardConfig.mk`
* **Code:**
```makefile
BOARD_KERNEL_CMDLINE += androidboot.selinux=permissive
```

---

### PATCH-06: USB UDC Gadget Binding
* **Domain:** Init / USB
* **Target File:** `device/casper/via_s48/rootdir/init.mt8168.usb.rc`
* **Code:**
```rc
on boot
    setprop sys.usb.controller 11201000.usb
    setprop sys.usb.configfs 1
```

---

### PATCH-07: 2.1 GHz OPP Device Tree Table
* **Domain:** Kernel DTS
* **Target File:** `arch/arm64/boot/dts/mediatek/mt8168.dtsi`
* **Code:**
```dts
opp-2100000000 {
    opp-hz = /bits/ 64 <2100000000>;
    opp-microvolt = <1150000>;
};
```

---

### PATCH-08: CPUFreq Driver Frequency Ceiling
* **Domain:** Kernel Driver
* **Target File:** `drivers/cpufreq/mt8168-cpufreq.c`
* **Code:**
```c
#define MT8168_MAX_FREQ 2100000
```

---

### PATCH-09: Thermal Throttle Budget Recalibration
* **Domain:** Kernel Thermal
* **Target File:** `drivers/misc/mediatek/thermal/mt8168/mtk_power_throttle.c`
* **Purpose:** Prevents immediate thermal trip-down when 2.1 GHz frequency is engaged.

---

### PATCH-10: Source Integrated Device and Vendor Trees
* **Domain:** Android Build System
* **Target Repositories:** `device_casper_via_s48`, `vendor_casper_via_s48`
* **Purpose:** Complete BoardConfig, modern product properties, proprietary blob extractor, and tablet overlays.

---

### PATCH-11: Stock-Exact Capacitive Navigation Keys (`mtk_gslX680`)
* **Domain:** Kernel Input Driver & DTS (`kernel_casper_mt8168`, branch `android-16`, commit `414965654`)
* **Target Files:**
  * `drivers/input/touchscreen/mediatek/gslX680/mt8167/mtk_gslX680.c`
  * `arch/arm64/boot/dts/mediatek/tb8168p1_64_bsp_f716.dts`
* **Root Cause:** The earlier `TPD_HAVE_BUTTON` / `use-tpd-button = <0x1>` workaround used generic phone coordinates (`key_y = 883`) and `KEY_MENU` (`139`), which did not match the physical Casper VIA S48 (800×1280) bezel capacitive buttons.
* **Reverse-Engineered Stock Behavior:** ARM64 disassembly of the factory Casper C1.03 `4.14.141` kernel (`tpd_down`/`tpd_up`) revealed that `use-tpd-button` remains `0` and touches in the bezel band `Y = 1293..1351` are directly intercepted and emitted as input key events:
  * **Left (`X = 91..149`, `Y = 1293..1351`):** `KEY_APPSELECT` (`580` → Android `APP_SWITCH`)
  * **Center (`X = 411..469`, `Y = 1293..1351`):** `KEY_HOMEPAGE` (`172` → Android `HOME`)
  * **Right (`X = 651..709`, `Y = 1293..1351`):** `KEY_BACK` (`158` → Android `BACK`)
* **Implementation:** Registered `KEY_APPSELECT`, `KEY_HOMEPAGE`, and `KEY_BACK` in `tpd_local_init()`, implemented the exact `Y=1293..1351` press/release state machine in `tpd_down()`/`tpd_up()`, disabled `TPD_HAVE_BUTTON`, and restored `use-tpd-button = <0x0>` in `tb8168p1_64_bsp_f716.dts`.
* **Standalone Patch File:** `kernel_casper_mt8168/patches/0001-via-s48-input-match-stock-capacitive-keys.patch`
* **Verification:** Hardware-verified on `boot_oc_2.1ghz_android16_v1_plus_capkeys.img` (SHA-256: `4017d6529fa75581a3417d71b768b34516a8d4e0d93f7611a8001607f8bbf60a`) running AxionOS 2.9 Android 16.
