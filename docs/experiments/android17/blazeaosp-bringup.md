# Experiment EXP-006: BlazeAOSP Android 17 (Baklava) Bring-Up

- **Device:** Casper VIA S48 (`MT8168` / `F716`)
- **OS Version:** Android 17 (Baklava / API Level 37)
- **Target GSI:** `BlazeAOSP Android 17 Generic System Image` (`CP2A.260605.016`)
- **Kernel:** Linux 4.14.141 @ 2.1 GHz Overclocked + DFE Ramdisk (`boot_oc_2.1ghz_android16.img`)
- **Status:** **PREPARED & VALIDATED FOR FASTBOOTD FLASHING**

---

## 1. Context & Motivation

Following the successful bring-up of AxionOS Android 16 (QPR2) on the Casper VIA S48, the next milestone is verifying Treble compatibility on Android 17 (Baklava / API 37).
The target image is located at `BlazeAOSP-GSI/system.img` (3,408,465,920 bytes raw ext4).

---

## 2. Forensic Analysis & Root Cause Identification

Inspecting `BlazeAOSP-GSI/system.img` revealed key incompatibilities with legacy MediaTek MT8168 Android 10 vendor HALs:

1. **VINTF FCM Level 4 Missing (`CRITICAL BLOCKER`):**
   - Android 17 GSI only supplied framework compatibility matrices for levels 7, 8, 202404, 202504, and 202604.
   - Casper VIA S48 vendor manifest declares `target-level="4"` (Android 10). Without `compatibility_matrix.4.xml`, Android `init` / `libvintf` immediately aborts early boot.
   - **Resolution:** Injected `compatibility_matrix.4.xml` into `/system/etc/vintf/compatibility_matrix.4.xml`.

2. **FUSE-BPF Kernel Incompatibility (`CRITICAL BLOCKER`):**
   - Android 17 `vold` relies on FUSE-BPF by default.
   - Linux kernel 4.14.141 lacks upstream FUSE-BPF subsystem backports, causing `vold` to crash on mounting storage.
   - **Resolution:** Appended `persist.sys.fuse.bpf.override=false` and `persist.sys.phh.bpf=false` to `/system/build.prop`.

3. **HIDL Service Manager Disabled (`BLOCKER`):**
   - In `build.prop`, `hwservicemanager.always_sets_disabled=true` was set by default in modern AOSP.
   - Casper VIA S48 vendor HALs (audio@5.0, graphics.allocator@2.0, graphics.composer@2.1, camera.provider@2.4) communicate via HIDL through `hwservicemanager`. If disabled, vendor HALs cannot initialize.
   - **Resolution:** Modified `hwservicemanager.always_sets_disabled=false` and added startup triggers.

4. **MediaTek MP2 Codec UBSAN Crash:**
   - Android 17 `libstagefright` aborts on `OMX.MTK.AUDIO.DECODER.MP2` due to integer overflow checks on legacy MediaTek declarations.
   - Since BlazeAOSP lacks phh's `rw-system.sh`, an init script `/system/etc/init/casper_compat.rc` was created to bind-mount a sanitized `media_codecs_mediatek_audio.xml` over `/vendor/etc/media_codecs_mediatek_audio.xml` at `post-fs`.

5. **Keymaster 4.0 FBE Attestation Deadlock:**
   - Disabled File-Based Encryption (DFE) by bind-mounting unencrypted `fstab.mt8168` onto `/vendor/etc/fstab.mt8168` at `post-fs`.

6. **USB Device Controller (UDC) Binding:**
   - Assigned `sys.usb.controller=11201000.usb` to bind ConfigFS gadget to MT8168 hardware controller.

---

## 3. Dynamic Partition Geometry & Super Feasibility

- **Physical `super` Partition:** 5,890,572,288 bytes (5.48 GiB)
- **Dynamic Group `main` Capacity:** 5,886,377,984 bytes
- **Partition Allocation:**
  - `system`: 3,408,465,920 bytes (BlazeAOSP patched)
  - `vendor`: 332,238,848 bytes (Stock Android 10)
  - `product`: 1,204,490,240 bytes (Stock Android 10)
  - **Total Dynamic Size:** 4,945,195,008 bytes (~4.95 GB)
  - **Remaining Headroom:** ~941 MB of free space inside `super`

---

## 4. Generated Artifacts

- **Patched System Image:** `BlazeAOSP-patched.img`
  - Size: `3,408,465,920 bytes`
  - SHA-256: `819f1770d747d267afdd521fd5bea210cda1980bf17cae7951493356a304583e`
  - Filesystem Integrity: `e2fsck -f -p` passed with 0 errors.
- **Flashing Scripts:**
  - `flash_blazeaosp_android17.sh` (Linux / macOS)
  - `flash_blazeaosp_android17.bat` (Windows)

---

## 5. Execution Procedure (Fastbootd)

```bash
# Enter Fastbootd
fastboot reboot fastboot

# Flash components
fastboot --disable-verity --disable-verification flash vbmeta vbmeta_disabled.img
fastboot flash boot boot_oc_2.1ghz_android16.img
fastboot flash system BlazeAOSP-patched.img

# Clean format userdata (DFE)
fastboot erase userdata
fastboot erase cache

#Reboot to Android 17
fastboot reboot
```

---

## 6. Post-Flash Boot Failure Root Cause Analysis (Confirmed Findings vs. Superseded Hypotheses)

> [!NOTE]
> **Historical Record & Superseded Hypotheses:**
> When `BlazeAOSP-patched.img` first dropped back into `LK` fastboot during early boot, initial hypotheses considered whether Linux 4.14.141 lacked mandatory Android 17 kernel syscalls (`SIGSYS`), `binderfs`, or cgroup v2 freezer support. Subsequent empirical forensics against our live booted AxionOS Android 16 system (`Linux 4.14.141 #10 SMP PREEMPT`) **superseded** those kernel-blame hypotheses and identified three concrete userspace/init root causes in `BlazeAOSP-patched.img`:

1. **Missing `com.android.vndk.v29.apex` in `BlazeAOSP` (`CONFIRMED BLOCKER`):**
   - Casper VIA S48 vendor specifies `ro.vndk.version=29`. While `Axion-2.9-patched.img` includes `com.android.vndk.v28.apex`, `v29` (`103.7 MB`), `v30`, `v31`, `v32`, `v33`, and `v34`, `BlazeAOSP` only shipped `v31`–`v34`. Without `v29` mounted at `/bootstrap-apex/com.android.vndk.v29`, `linkerconfig` could not provide VNDK-29 libraries (`libhidlbase.so`, `libutils.so`) to early vendor HALs (`boot@1.0`, `health@2.0`, `keymaster@4.0`).
2. **Late `fstab.mt8168` Bind-Mount Timing (`on post-fs` vs `on early-fs`) (`CONFIRMED BLOCKER`):**
   - Stock `/vendor/etc/init/hw/init.mt8168.rc` executes `mount_all /vendor/etc/fstab.mt8168` during `on fs`. Because `casper_compat.rc` bind-mounted the unencrypted `fstab.mt8168` during `on post-fs` (which runs *after* `on fs`), `mount_all` used the stock FBE-encrypted `fstab.mt8168`, causing `vold` to fail on `/data` and trigger `reboot,vold-failed`.
3. **Missing `on load-bpf-programs` Init Trigger (`CONFIRMED BLOCKER`):**
   - Overwriting Android 17's `netbpfload.rc` with Android 16's 4-line service definition removed the `on load-bpf-programs` trigger (which Google moved out of `init.rc` in Android 17), preventing `bpf.progs_loaded=1` and `netd` startup.

See full details in:
- [Android 17 Minimum Kernel Requirements](../../development/android17-minimum-kernel-requirements.md)
- [Kernel Upgrade Feasibility & Strategic Recommendation](../../development/kernel-upgrade-feasibility.md)

