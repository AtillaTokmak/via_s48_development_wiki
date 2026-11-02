# EXP-004: PixelOS Android 16 GSI Adaptation & Canonical Ext4 Repacking Resolution

## Metadata
* **Experiment ID:** `EXP-004`
* **Date:** `2026-11-02`
* **Target OS:** PixelOS Android 16 (API Level 36, Baklava QPR0)
* **Target Build ID:** `BP2A.250605.031.A2`
* **Release Tag:** `20250806` (`pixelos_arm64_bgN_ext4-16.0-unofficial-20250806.img.xz`)
* **Host Platform:** Casper VIA S48 (MediaTek MT8168 / F716, Mali-G52, Linux 4.14.141 @ 2.1 GHz)
* **Working Baseline:** AxionOS Android 16 (`Axion-2.9-patched.img`) + `boot_oc_2.1ghz_android16.img`
* **Original GSI Asset:** `/home/atilla/Desktop/Via S48/pixelos_investigation/pixelos_arm64_bgN_ext4-16.0-unofficial-20250806.img` (`4,520,169,472` bytes, SHA-256: `791a546e33f07d0c25cf11a4db3fb24407c99badd8776d8d20f79d89d47bcc66`)
* **Intermediate Patched Candidate:** `/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-candidate.img` (`4,429,475,840` bytes, SHA-256: `1e9ed5baa67bb242bcdf5fb52479705c0c77de0bd9bf0fe899dc0b4e06026d52`)
* **Final Repacked Image:** `/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-repacked.img`
* **Repacked Size:** `4,160,749,568` bytes (`1,015,808` blocks × `4,096` B = `8,126,464` sectors = `3.875 GiB`)
* **Repacked SHA-256:** `02c0a248f2910a33a6b8a25bc3edc2e4a94090537069e20a1e58226887b783e3`
* **Filesystem Status:** 100% clean (`/sbin/e2fsck -fn` passed all 5 passes with 0 errors across 5,389 files)
* **Dynamic Partition Margin:** **`263,708,672` bytes (`251.49 MiB`) free headroom** within `super`'s `main` group
* **Live Flash Outcome:** **CONCLUDED (Bootloop at Casper Logo)** — Device bootloops immediately at the primary Casper bootloader logo prior to Zygote/SystemUI initialization.
* **Final Recommendation:** **CONCLUDE EXPERIMENTAL DEBUGGING** (AxionOS Android 16 retained as the sole verified, fully functional daily-driver baseline).

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
* **Decompressed Raw Image Size:** `4,520,169,472` bytes (`1,103,557` 4096-B blocks)
* **Decompressed Raw Image SHA-256:** `791a546e33f07d0c25cf11a4db3fb24407c99badd8776d8d20f79d89d47bcc66`

### Internal Properties (`system/build.prop`)
```properties
ro.system.build.version.release=16
ro.system.build.version.sdk=36
ro.system.build.id=BP2A.250605.031.A2
ro.system.build.fingerprint=google/pixelos_arm64_bgN_ext4/tdgsi_arm64_ab:16/BP2A.250605.031.A2/eng.mitja:userdebug/release-keys
ro.treble.enabled=true
```

### EROFS Incompatibility Proof
The alternative release asset `pixelos_arm64_bgN_erofs-16.0-unofficial-20250806.img.xz` was evaluated. Inspection of the MT8168 Linux 4.14.141 kernel source confirmed that `CONFIG_EROFS_FS` is completely absent (`fs/erofs` does not exist; EROFS was merged in Linux 5.4). Therefore, EXT4 is the only bootable filesystem on the MT8168 4.14 kernel.

---

## 2. Technical Comparison & Subsystem Adaptations

| Subsystem / Issue | Stock PixelOS Android 16 | Casper VIA S48 Resolution | Implementation Verification |
| :--- | :--- | :--- | :--- |
| **VNDK 29 Availability** | Pre-bundled in `system/system_ext/apex/com.android.vndk.v29.apex` | **Preserved intact**: Essential for MT8168 Android 10 vendor blobs (`ro.vndk.version=29`). | Inode 4819, `103,788,544` bytes, verified intact. |
| **fstab DFE Execution Timing** | `mount_all` in vendor init triggers on `on fs` | **Fixed**: Installed `/system/etc/init/casper_compat.rc` to bind-mount `/system/etc/fstab.mt8168` on `on early-fs`, **strictly before** vendor `mount_all`. | Verified trigger `on early-fs` in `casper_compat.rc`. |
| **MediaTek MP2 Codec Crash** | `OMX.MTK.AUDIO.DECODER.MP2` in vendor XML triggers UBSAN underflow SIGABRT in `system_server` | **Fixed**: `/system/bin/rw-system.sh` strips MP2 definitions and bind-mounts clean XML on `post-fs` before runtime starts. | Injected sed filter in `rw-system.sh` (mode `0755`, UID `0`, GID `2000`). |
| **build.prop Permissions** | Mode `0600` in initial patch | **Corrected**: Enforced mode `0644` (`root:root`) so all non-root Android apps and Bionic runtime can query system properties. | Inode 715, mode `0644`, verified via `debugfs stat`. |
| **BPF Ring Buffer Bypass** | `system/etc/bpf/bpfRingbufProg.o` requires Linux 5.8+ ringbuf map | **Fixed**: Deleted `bpfRingbufProg.o` (matches AxionOS baseline). Commented `reboot_on_failure` in `netbpfload.rc`. | BPF directory verified clean; `reboot_on_failure` disabled. |
| **USB Gadget & Bluetooth** | Unconfigured | **Fixed**: Appended `sys.usb.controller=11201000.usb`, `persist.sys.fuse.bpf.override=false`, and capped BT LE commands (`182,174`). | Appended to `/system/build.prop`. |

---

## 3. Resolving the Dynamic Partition Inode Constraint via Canonical Repacking

### The Initial Shrink Constraint (Why resize2fs Stopped at 4.429 GB)
The original PixelOS image was formatted with `s_inodes_count = 5440` and `s_inodes_per_group = 160`. In ext4, $\lceil 5440 / 160 \rceil = 34$ block groups are structurally mandated. `resize2fs` only resizes data blocks (`s_blocks_count`) and cannot modify `s_inodes_count` or reduce the number of block groups. Consequently, Group 33's inode table locked the minimum filesystem boundary at `1,081,415` blocks (`4,429,475,840` bytes), which exceeded `super`'s `main` group capacity (`4,424,458,240` bytes) by 4.78 MiB.

### Canonical Android Repacking Workflow
Rather than truncating the filesystem or stripping components, the entire root filesystem was safely repacked using the official Android build toolchain (`mke2fs` + `e2fsdroid`):

1. **Complete Extraction with Relative Symlink Fidelity:**
   Extracted all files, directories, and relative symlinks (e.g. `system/framework/arm/boot-apache-xml.vdex -> ../boot-apache-xml.vdex`) using `debugfs rdump`.
2. **Metadata & SELinux Mapping:**
   Parsed exact ownership, octal permissions, and SELinux contexts for all 5,380 entries from the candidate image into canned configuration files:
   * `/tmp/vias48/pixelos_investigation/canned_fs_config`
   * `/tmp/vias48/pixelos_investigation/canned_file_contexts`
3. **New EXT4 Filesystem Initialization (`mke2fs`):**
   * Target block count: `1,015,808` blocks (`4,160,749,568` bytes = `3.875 GiB` / 31 block groups)
   * Allocated inodes: `8,432` inodes (`-N 8192` + group alignment, reserving 3,043 free inodes above the 5,389 used inodes)
   * Features: `-O ^has_journal,^metadata_csum,^64bit -I 256 -b 4096 -L / -M / -m 0`
4. **Android Tree Population (`e2fsdroid`):**
   Populated the blank filesystem with `e2fsdroid -e -S canned_file_contexts -C canned_fs_config -f repack_src -a /`.
5. **Filesystem Integrity Check:**
   `/sbin/e2fsck -fn` passed **100% clean across all 5 passes with 0 errors**:
   `/: 5389/8432 files (0.9% non-contiguous), 985660/1015808 blocks`

---

## 4. Final Partition Geometry & Capacity Assessment

$$\begin{aligned}
\text{Total Super Partition Size} &= 5,891,620,864\text{ bytes } (11,507,072\text{ sectors}) \\
\text{Main Group Maximum Capacity} &= 5,890,572,288\text{ bytes } (11,505,024\text{ sectors}) \\
\text{Fixed Vendor Partition} &= 317,046,784\text{ bytes } (619,232\text{ sectors}) \\
\text{Fixed Product Partition} &= 1,149,067,264\text{ bytes } (2,244,272\text{ sectors}) \\
\text{Total Fixed Used} &= 1,466,114,048\text{ bytes } (2,863,504\text{ sectors}) \\
\mathbf{\text{Maximum Allowable System Size}} &= \mathbf{4,424,458,240\text{ bytes } (8,641,520\text{ sectors} = 4.1206\text{ GiB})} \\
\mathbf{\text{Repacked Image Size}} &= \mathbf{4,160,749,568\text{ bytes } (8,126,464\text{ sectors} = 3.8750\text{ GiB})} \\
\mathbf{\text{Free Dynamic Headroom in Group 'main'}} &= \mathbf{263,708,672\text{ bytes } (515,056\text{ sectors} = 251.49\text{ MiB})} \\
\mathbf{\text{Free Space Inside Ext4 Filesystem}} &= \mathbf{123,486,208\text{ bytes } (30,148\text{ blocks} = 117.77\text{ MiB})}
\end{aligned}$$

The repacked image fits within the `super` partition group `main` with **251.49 MiB of safe margin**, completely resolving the partition boundary block.

---

## 5. Image Artifacts Inventory

| File Path | Description | Size (Bytes) | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| `pixelos_investigation/pixelos_arm64_bgN_ext4-16.0-unofficial-20250806.img` | Original decompressed GSI | `4,520,169,472` | `791a546e33f07d0c25cf11a4db3fb24407c99badd8776d8d20f79d89d47bcc66` |
| `PixelOS-A16-Casper-VIA-S48-candidate.img` | Intermediate patched candidate | `4,429,475,840` | `1e9ed5baa67bb242bcdf5fb52479705c0c77de0bd9bf0fe899dc0b4e06026d52` |
| **`PixelOS-A16-Casper-VIA-S48-repacked.img`** | **Final repacked & verified image** | **`4,160,749,568`** | **`02c0a248f2910a33a6b8a25bc3edc2e4a94090537069e20a1e58226887b783e3`** |
| `Axion-2.9-patched.img` | Working Android 16 rollback baseline | `3,070,558,208` | (Preserved intact) |
| `boot_oc_2.1ghz_android16.img` | Working 2.1 GHz custom kernel | `10,500,096` | (Preserved intact) |

---

## 6. Controlled Installation & Rollback Procedures

### Installation Sequence (Pending User Authorization):
```bash
# 1. Verify device is connected in fastbootd mode
fastboot devices
fastboot getvar is-userspace

# 2. Resize system logical partition to repacked image size
fastboot resize-logical-partition system 4160749568

# 3. Flash verified repacked PixelOS image
fastboot flash system "/home/atilla/Desktop/Via S48/PixelOS-A16-Casper-VIA-S48-repacked.img"

# 4. Clean targeted userdata erase (Preserves /cache diagnostic logs)
fastboot erase userdata

# 5. Reboot into PixelOS Android 16
fastboot reboot
```

### Rollback Sequence (To Working AxionOS Android 16):
```bash
# 1. Enter userspace fastbootd
fastboot reboot fastboot

# 2. Resize system logical partition back to AxionOS baseline
fastboot resize-logical-partition system 3070558208

# 3. Flash verified AxionOS baseline image
fastboot flash system "/home/atilla/Desktop/Via S48/Axion-2.9-patched.img"

# 4. Reboot into AxionOS
fastboot reboot
```

---

## 7. Final Conclusion & Retaining AxionOS Baseline

> [!NOTE]
> ### **FINAL CONCLUSION: CONCLUDE EXPERIMENTAL DEBUGGING**
> 
> 1. **Repacking & Geometry Resolution:** The ext4 image repacking successfully proved that `e2fsdroid` and `mke2fs` can create an ext4 image fitting dynamic partition constraints with 251 MiB of margin and 100% clean filesystem integrity.
> 2. **Boot Execution Outcome:** Upon live flashing and targeted userdata formatting, the tablet bootloops immediately at the white Casper bootloader logo prior to Zygote/SystemUI initialization, indicating an early kernel/init or linker incompatibility specific to the PixelOS Treble GSI framework.
> 3. **Strategic Decision:** In agreement with the device owner, further time-consuming debugging of PixelOS and BlazeAOSP is terminated. **AxionOS 2.9 (Android 16 / API 36)** paired with the **2.1 GHz custom kernel** is confirmed as the stable, daily-driver production baseline for the Casper VIA S48.
> 4. **Rollback:** The system is restored cleanly to `Axion-2.9-patched.img` via fastbootd.
