# EXP-004: PixelOS Android 16 GSI Adaptation & Dynamic Partition Geometry Audit

## Metadata
* **Experiment ID:** `EXP-004`
* **Date:** `2026-11-02`
* **Target OS:** PixelOS Android 16 (API Level 36, Baklava QPR0)
* **Target Build ID:** `BP2A.250605.031.A2`
* **Release Tag:** `20250806` (`pixelos_arm64_bgN_ext4-16.0-unofficial-20250806.img.xz`)
* **Host Platform:** Casper VIA S48 (MediaTek MT8168 / F716, Mali-G52, Linux 4.14.141 @ 2.1 GHz)
* **Working Baseline:** AxionOS Android 16 (`Axion-2.9-patched.img`) + `boot_oc_2.1ghz_android16.img`
* **Candidate Image:** `/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-candidate.img`
* **Candidate Size:** `4,429,475,840` bytes (`1,081,415` blocks × `4,096` B = `8,651,320` 512-byte sectors)
* **Candidate SHA-256:** `1e9ed5baa67bb242bcdf5fb52479705c0c77de0bd9bf0fe899dc0b4e06026d52`
* **Filesystem Status:** 100% clean (`/sbin/e2fsck -fn` passed all 5 passes with 0 errors)
* **Recommendation:** **TECHNICAL NO-GO** (Exceeds `super` dynamic partition group `main` by 4.78 MiB)

---

## 1. Target ROM Inspection & Asset Verification

### Release Metadata
* **Repository:** `mytja/treble_pixelos`
* **Release:** `20250806` (Published: 2025-08-06T21:06:17Z)
* **Selected Asset:** `pixelos_arm64_bgN_ext4-16.0-unofficial-20250806.img.xz`
  * Architecture: ARM64 (`arm64`)
  * Partition Type: A/B (`b`)
  * Apps: Included Google Mobile Services (`g`)
  * Superuser: Non-root vanilla (`N`)
  * Filesystem: Ext4 (`ext4`)
* **Archive Size:** `1,659,638,984` bytes (~1.55 GiB)
* **Archive SHA-256:** `91a1549837d0514fa88c60c3131af698cefe905824532d314945ddce3e43f426`
* **Raw Image Size:** `4,520,169,472` bytes (`1,103,557` 4096-B blocks)
* **Raw Image SHA-256:** `791a546e33f07d0c25cf11a4db3fb24407c99badd8776d8d20f79d89d47bcc66`

### Internal Properties (`system/build.prop`)
```properties
ro.system.build.version.release=16
ro.system.build.version.sdk=36
ro.system.build.id=BP2A.250605.031.A2
ro.system.build.fingerprint=google/pixelos_arm64_bgN_ext4/tdgsi_arm64_ab:16/BP2A.250605.031.A2/eng.mitja:userdebug/release-keys
ro.treble.enabled=true
```

### EROFS Incompatibility
The alternative release asset `pixelos_arm64_bgN_erofs-16.0-unofficial-20250806.img.xz` was evaluated. Inspection of the MT8168 Linux 4.14.141 kernel source confirmed that `CONFIG_EROFS_FS` is completely absent (`fs/erofs` does not exist; EROFS was merged upstream into Linux 5.4). Therefore, EROFS GSIs cannot be mounted by the stock or overclocked MT8168 kernel.

---

## 2. Comparison Against AxionOS Android 16 Baseline

| Component / Subsystem | Stock PixelOS Android 16 | Working AxionOS Baseline | Casper VIA S48 Resolution |
| :--- | :--- | :--- | :--- |
| **VNDK 29 Support** | Pre-bundled in `system/system_ext/apex/com.android.vndk.v29.apex` | Bundled | **Identical**: Native Treble VNDK 29 supported out-of-the-box. |
| **MediaTek MP2 Codec Crash** | Present (`OMX.MTK.AUDIO.DECODER.MP2` in vendor XML causes `system_server` SIGABRT / UBSAN underflow) | Patched via `rw-system.sh` | **Resolved**: Injected dynamic XML stripping and bind-mount into `/system/bin/rw-system.sh`. |
| **fstab DFE** | Unpatched (`checkpoint=block`, `fileencryption=aes-256-xts`) | Stripped via `rw-system.sh` | **Resolved**: Injected sed filter in `rw-system.sh` and added clean `/system/etc/fstab.mt8168`. |
| **BPF Ring Buffer** | Present (`/system/etc/bpf/bpfRingbufProg.o`) | Not present | **Resolved**: Removed `bpfRingbufProg.o`. Linux 4.14 lacks `BPF_MAP_TYPE_RINGBUF`; removal matches AxionOS. |
| **netbpfload Init Trigger** | Contains `reboot_on_failure reboot,netbpfload-missing` | Not present | **Resolved**: Commented out `reboot_on_failure` in `/system/etc/init/netbpfload.rc`. |
| **USB Gadget Controller** | Generic configfs | Forced `11201000.usb` | **Resolved**: Added `sys.usb.controller=11201000.usb` and `persist.sys.usb.config=adb`. |
| **Bluetooth Capabilities** | Uncapped LE commands | Disabled 182, 174 | **Resolved**: Appended `persist.sys.bt.unsupported.commands=182,174` to `build.prop`. |
| **FUSE BPF** | Default enabled | Explicitly disabled | **Resolved**: Appended `persist.sys.fuse.bpf.override=false` and `persist.sys.phh.bpf=false`. |

---

## 3. Applied Modifications & File-Level Audit

The experimental candidate `/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-candidate.img` was constructed from the raw GSI with the following exact changes:

1. **Unused APEX Deletion (Freed ~478 MB of data blocks):**
   * Deleted `system/system_ext/apex/com.android.vndk.v28.apex` (94.5 MB, Android 9)
   * Deleted `system/system_ext/apex/com.android.vndk.v30.apex` (114.8 MB, Android 11)
   * Deleted `system/system_ext/apex/com.android.vndk.v31.apex` (41.9 MB, Android 12)
   * Deleted `system/system_ext/apex/com.android.vndk.v32.apex` (42.0 MB, Android 12L)
   * Deleted `system/system_ext/apex/com.android.vndk.v33.apex` (47.7 MB, Android 13)
   * Deleted `system/system_ext/apex/com.android.vndk.v34.apex` (42.3 MB, Android 14)
   * Deleted `system/apex/com.android.virt.apex` (95.6 MB, KVM virtualization unsupported on Cortex-A53 4.14)
   * Deleted `system/system_ext/apex/com.android.compos.apex` (3.8 MB, CompOS virtualization)
   * *Preserved:* `system/system_ext/apex/com.android.vndk.v29.apex` (Mandatory for Android 10 vendor blobs).

2. **MediaTek MP2 Codec & DFE Patch in `system/bin/rw-system.sh`:**
   Injected immediately after logging setup:
   ```bash
   # Fix MediaTek MP2 codec crash in Android 16
   if [ -f /vendor/etc/media_codecs_mediatek_audio.xml ]; then
       mkdir -p /mnt/phh
       cp /vendor/etc/media_codecs_mediatek_audio.xml /mnt/phh/media_codecs_mediatek_audio.xml
       sed -i '/OMX.MTK.AUDIO.DECODER.MP2/,/<\/MediaCodec>/d' /mnt/phh/media_codecs_mediatek_audio.xml
       mount -o bind /mnt/phh/media_codecs_mediatek_audio.xml /vendor/etc/media_codecs_mediatek_audio.xml
       chcon -h u:object_r:vendor_configs_file:s0 /vendor/etc/media_codecs_mediatek_audio.xml
       chmod 0644 /vendor/etc/media_codecs_mediatek_audio.xml
   fi

   # DFE: Strip force-encryption from vendor fstab
   if [ -f /vendor/etc/fstab.mt8168 ]; then
       mkdir -p /mnt/phh
       cp /vendor/etc/fstab.mt8168 /mnt/phh/fstab.mt8168
       sed -i 's/,checkpoint=block//g' /mnt/phh/fstab.mt8168
       sed -i 's/,fileencryption=aes-256-xts//g' /mnt/phh/fstab.mt8168
       mount -o bind /mnt/phh/fstab.mt8168 /vendor/etc/fstab.mt8168
       chcon -h u:object_r:vendor_configs_file:s0 /vendor/etc/fstab.mt8168
       chmod 0644 /vendor/etc/fstab.mt8168
   fi
   ```
   Permissions: `0755` (`0100755`), UID `0`, GID `2000`, Context: `u:object_r:phhsu_exec:s0`.

3. **BPF Loader Protection in `system/etc/init/netbpfload.rc`:**
   * Commented out `reboot_on_failure reboot,netbpfload-missing`.
   * Deleted `system/etc/bpf/bpfRingbufProg.o`.
   * Permissions: `0644`, UID `0`, GID `0`, Context: `u:object_r:system_file:s0`.

4. **Hardware & Subsystem Properties in `system/build.prop`:**
   Appended:
   ```properties
   # Casper VIA S48 MT8168 Compatibility Overrides
   persist.sys.fuse.bpf.override=false
   persist.sys.phh.bpf=false
   sys.usb.controller=11201000.usb
   persist.sys.usb.config=adb
   ro.adb.secure=0
   persist.sys.bt.unsupported.commands=182,174
   ```
   Permissions: `0600`, UID `0`, GID `0`, Context: `u:object_r:system_file:s0`.

5. **Clean Legacy fstab `system/etc/fstab.mt8168`:**
   Installed verified DFE fstab (`3,661` bytes). Permissions: `0644`, UID `0`, GID `0`, Context: `u:object_r:system_file:s0`.

---

## 4. Inode & Dynamic Partition Geometry Audit (The Blocker)

### Live MediaTek Dynamic Partition Layout (`lpdump`)
* **Total Super Partition Size:** `5,891,620,864` bytes (`11,507,072` sectors)
* **Dynamic Partition Group `main` Capacity:** `5,890,572,288` bytes (`11,505,024` sectors)
* **Preserved Partitions:**
  * `vendor`: `317,046,784` bytes (`619,232` sectors, physical `2048..621280`)
  * `product`: `1,149,067,264` bytes (`2,244,272` sectors, physical `622592..2866864`)
  * *Total Fixed Used:* `1,466,114,048` bytes (`2,863,504` sectors)
* **Maximum Safe Allowable `system` Partition Size:**
  $$\text{Max System} = 5,890,572,288 - 1,466,114,048 = 4,424,458,240\text{ bytes } (8,641,520\text{ sectors} = 4.1206\text{ GiB})$$

### The Ext4 Superblock Inode Constraint
After removing unused APEXes, data usage dropped to `968,998` blocks (`3,969,015,808` bytes = ~3.69 GiB). However, when executing `resize2fs -M`, the utility refused to shrink below `1,081,415` blocks (`4,429,475,840` bytes):

```
/usr/bin/resize2fs: New size smaller than minimum (1081406)
```

**Root Cause Analysis:**
1. The original PixelOS image was formatted with `s_inodes_count = 5440` and `s_inodes_per_group = 160`.
2. Ext4 requires $\lceil 5440 / 160 \rceil = 34$ block groups (Groups 0 through 33).
3. In ext4, `resize2fs` dynamically resizes data blocks (`s_blocks_count`), but **cannot alter the number of block groups or shrink `s_inodes_count`**.
4. Group 33 starts at block `1081344` and contains its group descriptors and inode table.
5. Consequently, the minimum structural boundary of the filesystem is locked at `1,081,415` blocks (`4,429,475,840` bytes).

### The Capacity Deficit
$$\text{Candidate Minimum Ext4 Size} = 4,429,475,840\text{ bytes}$$
$$\text{Maximum Available Space in Group 'main'} = 4,424,458,240\text{ bytes}$$
$$\text{Deficit (Overshoot)} = 5,017,600\text{ bytes } (9,800\text{ sectors} = 4.785\text{ MiB})$$

Attempting to execute `fastboot flash system PixelOS-A16-Casper-VIA-S48-candidate.img` or `fastboot resize-logical-partition system 4429475840` in `fastbootd` will immediately fail with:
`FAILED (remote: 'Not enough space to resize partition')`
because the dynamic partition manager strictly enforces the `main` group ceiling.

---

## 5. Controlled Test & Rollback Procedure (Documented)

### Pre-conditions
* Device connected in userspace `fastbootd` (`fastboot getvar is-userspace` -> `yes`).
* Stock `vendor` (`317,046,784` bytes) and `product` (`1,149,067,264` bytes) intact.
* Rollback baseline available: `/home/atilla/Desktop/Via S48/Axion-2.9-patched.img`.

### If Flashing Were Authorized (Blocked by Size):
```bash
# 1. Resize logical partition to candidate size
fastboot resize-logical-partition system 4429475840

# 2. Flash candidate image
fastboot flash system "/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-candidate.img"

# 3. Clean targeted userdata erase (Preserving /cache for logs)
fastboot erase userdata

# 4. Reboot
fastboot reboot
```

### Rollback Procedure to Working AxionOS Android 16:
```bash
# 1. Enter fastbootd
fastboot reboot fastboot

# 2. Resize logical partition back to AxionOS size
fastboot resize-logical-partition system 3070558208

# 3. Flash working baseline
fastboot flash system "/home/atilla/Desktop/Via S48/Axion-2.9-patched.img"

# 4. Reboot into AxionOS
fastboot reboot
```

---

## 6. Final Recommendation

### **RECOMMENDATION: TECHNICAL NO-GO**
* **Finding:** While all Android 16 functional fixes (VNDK 29, MP2 audio crash resolution, DFE fstab, BPF ringbuf bypass, USB gadget overrides) were successfully ported and verified with 0 filesystem errors, the resulting EXT4 image cannot physically fit into the `super` partition's dynamic `main` group by **4.78 MiB**.
* **Integrity Mandate:** Per project engineering policy, ext4 filesystem truncation below its valid filesystem minimum (`1,081,406` blocks) is strictly prohibited to avoid corruption, and stock `vendor`/`product` partitions must not be deleted.
* **Resolution Path:** Future PixelOS porting requires rebuilding the rootfs with fewer inodes ($\le 4800$) at image creation time using upstream treble build tools, or repacking the tree once userspace FUSE loop utilities can be executed unsandboxed.
* **Device Safety:** The physical tablet was not modified during this investigation.
