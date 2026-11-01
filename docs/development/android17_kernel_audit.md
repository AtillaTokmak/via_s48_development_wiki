# Android 17 (Baklava / API 37) Investigation & Kernel Compatibility Audit

- **Target Device:** Casper VIA S48 (`MT8168` / `F716`)
- **Active Kernel Baseline:** Linux 4.14.141 (`SMP PREEMPT #10`, 2.1 GHz Overclocked)
- **Target OS:** BlazeAOSP Android 17 (`CP2A.260605.016`, API Level 37)
- **Defconfig:** `arch/arm64/configs/tb8168p1_64_bsp_f716_defconfig`
- **Status:** **ACTIVE INVESTIGATION — EMPIRICAL RECTIFICATION APPLIED**

---

## 1. Objective & Empirical Framework

This document records the empirical investigation into running Android 17 (Baklava / API 37) on the Casper VIA S48 tablet powered by MediaTek MT8168 and Linux kernel 4.14.141.

> [!IMPORTANT]
> **Methodological Principle:**
> We strictly distinguish between **confirmed empirical facts** (verified by binary analysis, filesystem inspection, or execution logs) and **hypotheses under test**. Unsupported syscalls in Linux return `-ENOSYS` by default; Bionic libc implements extensive fallback paths for modern syscalls (`clone3` -> `clone`, `faccessat2` -> `faccessat`, `openat2` -> `openat`). Syscall absence alone does **not** inherently imply `SIGSYS` unless explicitly trapped by a restrictive Seccomp filter.

---

## 2. Confirmed Findings & Validated Mitigations

### 2.1 VINTF Framework Compatibility Matrix Level 4
- **Fact:** Casper VIA S48 vendor manifest specifies `target-level="4"` (Android 10 FCM).
- **Finding:** BlazeAOSP GSI only included compatibility matrices for FCM 7, 8, 202404, 202504, and 202604.
- **Mitigation:** Injected `compatibility_matrix.4.xml` into `/system/etc/vintf/compatibility_matrix.4.xml`.

### 2.2 HIDL Service Manager Initialization
- **Fact:** Casper VIA S48 vendor HALs (audio@5.0, graphics.allocator@2.0, composer@2.1) require HIDL.
- **Finding:** Modern AOSP GSIs set `hwservicemanager.always_sets_disabled=true` in `build.prop`.
- **Mitigation:** Set `hwservicemanager.always_sets_disabled=false` and ensured explicit startup triggers.

### 2.3 Explicit APEX Failure Reboot Triggers (`reboot_on_failure`)
- **Fact:** `com.android.tethering` APEX contains `etc/netbpfload.35rc` defining:
  ```rc
  service bpfloader /apex/com.android.tethering/bin/netbpfload
      ...
      reboot_on_failure reboot,bpfloader-failed
  ```
- **Finding:** If `netbpfload` exits non-zero on older kernels lacking modern BPF map types, Android `init` executes an immediate reboot into bootloader. In contrast, our working AxionOS Android 16 image had this `reboot_on_failure` directive explicitly commented out by the Treble maintainer.
- **Mitigation:** Cleaned `netbpfload.rc`, `netd.rc`, and deployed the AxionOS-proven tethering APEX payload.

### 2.4 FUSE-BPF Daemon Override
- **Fact:** Linux 4.14 lacks upstream `CONFIG_FUSE_BPF`.
- **Mitigation:** Verified and applied `persist.sys.fuse.bpf.override=false` and `persist.sys.phh.bpf=false`.

### 2.5 MediaTek MP2 Codec UBSAN Crash Mitigation
- **Fact:** Legacy MediaTek audio codec declarations trigger integer overflow aborts in modern `libstagefright`.
- **Mitigation:** Sanitized `media_codecs_mediatek_audio.xml` bind-mounted over vendor at `post-fs`.

---

## 3. Hypotheses Under Active Investigation

The tablet's return to bootloader fastboot during Android 17 boot is actively being isolated across the following candidate stages:

### Hypothesis A: `libprocessgroup_setup` Cgroup v2 Controller Activation
- **Mechanism:** In `/system/etc/cgroups.json`, the `Cgroups2` hierarchy specifies:
  ```json
  "Controllers": [
    { "Controller": "freezer", "Path": "." }
  ]
  ```
- **Observation:** `"Optional": true` is set for `memory`, but omitted for `freezer`.
- **Testable Question:** Does Linux 4.14 return `-EINVAL` when `init` attempts to write `+freezer` to `/sys/fs/cgroup/cgroup.subtree_control`, causing `libprocessgroup_setup.so` to call `LOG(FATAL)` and abort `init`?
- **Proposed Test:** Add `"Optional": true` to the `freezer` controller entry in `cgroups.json`.

### Hypothesis B: `apexd-bootstrap` Activation Failure
- **Mechanism:** `/system/etc/init/apexd.rc` contains:
  ```rc
  service apexd-bootstrap /system/bin/apexd --bootstrap
      ...
      reboot_on_failure reboot,bootloader,bootstrap-apexd-failed
  ```
- **Testable Question:** Does `apexd-bootstrap` fail while mounting bootstrap APEXes (`com.android.runtime`, `com.android.art`, `com.android.i18n`, `com.android.conscrypt`) due to loopback, dm-verity, or namespace requirements?
- **Proposed Test:** Comment out `reboot_on_failure` in `apexd.rc` to allow init to proceed and log diagnostics rather than instantly rebooting.

### Hypothesis C: BCB / `misc` Partition Boot Reason Persistence
- **Mechanism:** The bootloader control block in `/dev/block/by-name/para` (`misc`) may retain a persistent bootloader flag from previous fastboot transitions.
- **Proposed Test:** Check and clear the `misc` BCB message block.

---

## 4. Subsystem Comparison: AxionOS 16 vs BlazeAOSP 17

| Subsystem | AxionOS 2.9 (Android 16 - Verified Working) | BlazeAOSP (Android 17 - In Progress) |
| :--- | :--- | :--- |
| **Kernel Used** | `boot_oc_2.1ghz_android16.img` (Linux 4.14.141 @ 2.1 GHz) | Same (`boot_oc_2.1ghz_android16.img` / `boot_gsi.img`) |
| **Ramdisk** | DFE Patched Initramfs (`791,547` bytes) | Same DFE Initramfs |
| **Target API** | API Level 36 (`BP4A.251205.006`) | API Level 37 (`CP2A.260605.016`) |
| **Treble Shims** | Included phh `rw-system.sh`, `treble-overlay-*` | Pure AOSP / Lineage GSI (Requires explicit init shims) |
| **`netbpfload.rc`** | No `reboot_on_failure` | Stripped and patched to match AxionOS |
| **`apexd.rc`** | Standard AOSP | Retains `reboot,bootloader,bootstrap-apexd-failed` |

---

## 5. Rollback Baseline Readiness

To ensure zero downtime and absolute device safety, the verified working Android 16 baseline is fully prepared and staged for immediate deployment upon explicit user confirmation:

- **System Image:** [`Axion-2.9-patched.img`](file:///home/atilla/Desktop/Via%20S48/Axion-2.9-patched.img) (`3,070,558,208 bytes`, SHA-256: `40cd5fa1e4e2e8ed407a89907ad1c797ae409bb272127fab904e96787154aba1`)
- **Boot Image:** [`boot_oc_2.1ghz_android16.img`](file:///home/atilla/Desktop/Via%20S48/boot_oc_2.1ghz_android16.img) (`11,079,680 bytes`, SHA-256: `879da6cfc5ce2e84cc5d54a21c69bbee28bf14278743dfe2ae1a711418fcbef9`)
- **VBMeta:** [`vbmeta_disabled.img`](file:///home/atilla/Desktop/Via%20S48/vbmeta_disabled.img) (`256 bytes`, SHA-256: `53ea4aee4a0f0fd0dc0dc4556bf3143f39ae7799aa55f35dbe8eb9ee482873f3`)
- **Vendor & Product Partitions:** 100% stock, unmodified, and preserved.
