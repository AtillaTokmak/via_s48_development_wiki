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

### PATCH-02: Disable File-Based Encryption (DFE)
* **Domain:** Boot Ramdisk / Vendor
* **Target File:** `/fstab.mt8168`
* **Diff:**
```diff
--- a/fstab.mt8168
+++ b/fstab.mt8168
@@ -26,3 +26,3 @@
-/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m,fileencryption=aes-256-xts,checkpoint=block
+/dev/block/platform/bootdevice/by-name/userdata /data ext4 noatime,nosuid,nodev,noauto_da_alloc wait,check,formattable,quota,resize,reservedsize=128m
```

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
* **Purpose:** Complete BoardConfig, modern product properties, 570 proprietary blob extractor, and tablet overlays.
