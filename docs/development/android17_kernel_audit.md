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

## 3. Superseded Hypotheses vs. Confirmed Root Causes (Historical Record)

> [!NOTE]
> **Historical Record & Superseded Hypotheses:**
> Earlier revisions of this audit hypothesized that the tablet's return to bootloader fastboot during Android 17 boot might be caused by missing Linux 5.x syscalls (`SIGSYS`), mandatory `binderfs`, or unified `cgroup v2` freezer enforcement. Those hypotheses are **superseded** by live telemetry from our working Android 16 baseline (`android16_cgroup_bpf_mounts.txt`, `android16_boot_logcat.log`) and direct binary/filesystem comparison between `Axion-2.9-patched.img` and `BlazeAOSP-patched.img`.

### 3.1 Confirmed Root Causes of the `BlazeAOSP-patched.img` Boot-to-Fastboot Failure
1. **Absence of `com.android.vndk.v29.apex` in `BlazeAOSP` (`/system/system_ext/apex/`):**
   - `BlazeAOSP` only shipped VNDK APEXes `v31`–`v34`, whereas Casper VIA S48's Android 10 vendor requires `ro.vndk.version=29` (present in `Axion-2.9-patched.img` as `com.android.vndk.v29.apex`, mounted at `/bootstrap-apex/com.android.vndk.v29@1` during `early-init`).
2. **Late `fstab.mt8168` Bind-Mount Ordering (`post-fs` vs `early-fs`):**
   - Stock `/vendor/etc/init/hw/init.mt8168.rc` runs `mount_all /vendor/etc/fstab.mt8168` inside `on fs`. Placing the unencrypted `fstab.mt8168` bind-mount inside `on post-fs` in `casper_compat.rc` meant `mount_all` executed against the unpatched FBE `fstab.mt8168`, triggering `reboot,vold-failed`.
3. **Removal of `on load-bpf-programs` Trigger in `netbpfload.rc`:**
   - In Android 17, `on load-bpf-programs` moved from `/system/etc/init/hw/init.rc` into `/system/etc/init/netbpfload.rc`. Overwriting `netbpfload.rc` with the 4-line AxionOS service definition deleted the `on load-bpf-programs` trigger.

### 3.2 Retained Secondary Investigation Targets (Post-Fix Verification)

The following secondary items remain documented for verification once the three primary blockers above are patched:

#### Secondary Check A: `libprocessgroup_setup` Cgroup v2 Controller Activation
- **Mechanism:** In `/system/etc/cgroups.json`, the `Cgroups2` hierarchy specifies:
  ```json
  "Controllers": [
    { "Controller": "freezer", "Path": "." }
  ]
  ```
- **Observation:** Live `/proc/mounts` on Android 16 confirms `/sys/fs/cgroup` mounts `cgroup2` alongside v1 `/sys/fs/cgroup/freezer`. Adding `"Optional": true` to the `Cgroups2` `freezer` entry in `cgroups.json` ensures `libprocessgroup` never aborts if `+freezer` write to `cgroup.subtree_control` returns `-EINVAL`.

#### Secondary Check B: `apexd-bootstrap` Activation Diagnostics
- **Mechanism:** `/system/etc/init/apexd.rc` contains `reboot_on_failure reboot,bootloader,bootstrap-apexd-failed`. With `com.android.vndk.v29.apex` injected, `apexd-bootstrap` mounts all bootstrap APEXes cleanly using `CLONE_NEWNS` and `loop0..loopN`.

---

## 4. Related Audit Documentation

- [Android 17 Minimum Kernel Requirements](android17-minimum-kernel-requirements.md)
- [Kernel Upgrade Feasibility & Strategic Recommendation](kernel-upgrade-feasibility.md)
- [MT8168 Hardware Driver Migration Audit](mt8168-driver-migration.md)
- [Kernel Version Technical Comparison Matrix (`4.14` – `5.15`)](kernel-version-comparison.md)

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
