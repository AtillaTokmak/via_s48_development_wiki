# Android 17 (Cinnamon Bun / API 37) Investigation & Kernel Compatibility Audit

- **Target Device:** Casper VIA S48 (`MT8168` / `F716`)
- **Active Kernel Baseline:** Linux 4.14.141 (`SMP PREEMPT #10`, 2.1 GHz Overclocked)
- **Target OS:** BlazeAOSP Android 17 (`CP2A.260605.016`, API Level 37.0)
- **Defconfig:** `arch/arm64/configs/tb8168p1_64_bsp_f716_defconfig`
- **Status:** **CLEAN REBUILD VALIDATED — DISASSEMBLY & CRYPTOGRAPHIC AUDIT COMPLETE**

---

## 1. Objective & Empirical Framework

This document records the empirical investigation into running Android 17 (API 37) on the Casper VIA S48 tablet powered by MediaTek MT8168 and Linux kernel 4.14.141.

> [!IMPORTANT]
> **Methodological Principle:**
> We strictly distinguish between **confirmed empirical facts** (verified by binary disassembly, cryptographic signature checks, filesystem inspection, or live execution logs) and **untested runtime hypotheses**. No hardcoded ARM64 binary patches or speculative configuration changes are permitted without independent proof.

---

## 2. Confirmed Root Causes of Attempt 1 & Verified Corrections

### 2.1 VNDK 29 APEX (`com.android.vndk.v29.apex`) — **CONFIRMED REQUIRED & VERIFIED**
- **Fact:** Casper VIA S48 vendor specifies `ro.vndk.version=29`. `BlazeAOSP-GSI/system.img` ships `com.android.vndk.v30.apex` through `v35`, omitting `v29`.
- **Cryptographic & Structural Verification:**
  - Extracted `com.android.vndk.v29.apex` (`103,755,776` bytes, SHA-256: `3a0aadbdaa0cb1177160f47ecb4458083e2db2c74150499a43ecdfa9c4c98433`) from `Axion-2.9-GSI_treble_arm64-ab-VANILLA-20261017.img`.
  - Outer APK v2/v3 signature certificate (`CN=com.android.vndk`, SHA-256: `ae0a71129a4e7ac2e0fe7b8a124732bc01668d5820da8af97f246696dc50752c`) and inner `apex_pubkey` (`SHA-256: 19d2131d80d4238098af9969233ab821199f0cfd059d817bc01c028ddf03e2f1`) are **100% identical** to `BlazeAOSP`'s `com.android.vndk.v31.apex`.
  - `avbtool verify_image --image apex_payload.img --key apex_pubkey` passes (`SHA256_RSA4096`, Root Digest: `dbdde006184ad1b4504298b092d3f2f6c57b83068a27edd29544e99a90bc6b22`).
  - All 20 LLNDK libraries in `/etc/llndk.libraries.29.txt` exist in `BlazeAOSP`, and `linkerconfig` symbol audit (`BuildVndkNamespace` at `0x252c8`, `GetVndkProvider` at `0x27838`) confirms full VNDK 29 compatibility.

### 2.2 `fstab.mt8168` Bind-Mount Ordering (`on early-fs`) & Stock Content — **CONFIRMED FIX**
- **Fact:** In `BlazeAOSP` `/system/etc/init/hw/init.rc` (lines 567–575), `on late-init` fires `trigger early-fs` (line 568) strictly before `trigger fs` (line 574) and `trigger post-fs` (line 575). Stock `/vendor/etc/init/hw/init.mt8168.rc` executes `mount_all /vendor/etc/fstab.mt8168` inside `on fs` (line 108).
- **Previous Defect:** Attempt 1 placed the bind-mount inside `on post-fs` (after `mount_all` had already failed on the encrypted stock `fstab.mt8168`) and used a synthetic `fstab.mt8168` missing `/nvram`, `/misc`, `/seccfg`, `/tee1`, `/tee2`.
- **Correction:** Replaced `/system/etc/fstab.mt8168` (`3,661` bytes, SHA-256: `eed78be91dfd421dc59d63f19f08c6643c27f6c4e13be6f228a42db2d35eb53c`) with an exact copy of stock `/vendor/etc/fstab.mt8168` with only `,checkpoint=block,fileencryption=aes-256-xts` removed from `/data`, and moved the bind-mount in `/system/etc/init/casper_compat.rc` to `on early-fs`.

### 2.3 Restoration of Original Android 17 `netbpfload.rc` — **CONFIRMED FIX**
- **Fact:** In Android 17, `on load-bpf-programs` (`exec_start bpfloader`, `wait_for_prop bpf.progs_loaded 1`, `exec_start netd1shot`, `start netd`) resides in `/system/etc/init/netbpfload.rc`, which also starts with `# 2026 2 37 0 0 #` (parsed by `netbpfload` at `0xa5fc`).
- **Previous Defect:** Attempt 1 replaced `/system/etc/init/netbpfload.rc` with a 2-line stub setting `bpf.progs_loaded=1` in `early-init`, deleting `on load-bpf-programs` and the `# 2026 2 37 0 0 #` header.
- **Correction:** Restored the original 69-line `/system/etc/init/netbpfload.rc` from `BlazeAOSP-GSI/system.img` (`SHA-256: 3e486e9f3be12e07d0879cdcd4ac3779e526977270454f3ffebbde7c361c8ea5`), removing only line 68 (`reboot_on_failure reboot,netbpfload-missing`).

---

## 3. Refuted Hypotheses & Disassembly Audits

### 3.1 Alleged `apexd` Loop-Device Race & Hardcoded Binary Patch — **REFUTED**
- **Audit:** Disassembled `android::apex::loop::WaitForLoopDevice(int)` (`0xb4cf8`) and `CreateLoopDevice` (`0xb3740`) in `BlazeAOSP` `/system/bin/apexd` (`blaze_apexd`).
- **Finding:** When `access("/dev/block/loopN", F_OK)` (`0xb4e30`) returns `ENOENT`, `WaitForLoopDevice` immediately calls `CreateLoopDeviceNode` (`0xb4e40` -> `0xba040`), which calls `mknod("/dev/block/loopN", S_IFBLK | 0600, makedev(7, N))` (`0xba0cc`). Furthermore, offsets `0x46794` / `0x46924` in `blaze_apexd` were inside an unrelated `std::__1::__hash_table` method (`0x46738`), not `CreateLoopDevice`.
- **Resolution:** Restored `/system/bin/apexd` to **100% unmodified stock `BlazeAOSP`** (`SHA-256: 9c1810e3900a3657ff79db74723c2f16c64217b629a7226e13e6c01bc612f6d7`). Only the 2 `reboot_on_failure` lines in `/system/etc/init/apexd.rc` are removed for diagnostics.

### 3.2 `cgroups.json` `Cgroups2` Freezer Failure Hypothesis — **REFUTED**
- **Audit:** Compared `/system/etc/cgroups.json` and disassembled `ActivateControllers` in `/system/lib64/libprocessgroup_setup.so` across `AxionOS` and `BlazeAOSP`.
- **Finding:** `/system/etc/cgroups.json` (`SHA-256: ab2ed667ff45958843fb0c6ee953a5512def0ae87470c4358aa9576a6a4b2e22`) is byte-for-byte identical between `AxionOS` and `BlazeAOSP`, and `ActivateControllers` (`0x6fd0..0x7398`) logs a non-fatal error when `+freezer` fails on `cgroup2` and continues without aborting.
- **Resolution:** Kept `/system/etc/cgroups.json` **100% unmodified**.

### 3.3 Static Disassembly Findings on the Android 17 BPF Chain (Pending Live Diagnostic Confirmation)
Disassembly of `BlazeAOSP`'s BPF chain identified three static version/flag checks on `Linux 4.14.141`:
1. `blaze_netbpfload` (`/apex/com.android.tethering/bin/netbpfload`): checks `kernelVersion() <= 0x412FFFF` (`< 4.19`) at `0xa2bc` (`cmp w8, #0x412`) and exits with code `5` (and exits with `6`/`7`/`38` on `< 5.4`/`< 5.10` BPF load failures), whereas `AxionOS`'s `netbpfload` logs warnings and proceeds.
2. `blaze_bpfloader` (`/system/bin/bpfloader`): Entry 6 (`/system/etc/bpf/bpfRingbufProg.bpf`) in the 10-entry descriptor table (`0x14e10..0x151cf`) has `flags = 0x0101` (`fail_on_error = 1`, `ignore_on_user_build = 1`) at offset `0x15000`, which is skipped on `ro.build.type=user` (`AxionOS`) but attempted on `ro.build.type=userdebug` (`BlazeAOSP`), triggering `panic_fmt` at `0xd4cc`.
3. `blaze_libnetd_updatable.so` (`/apex/com.android.tethering/lib64/libnetd_updatable.so`): `BpfHandler::init()` (`0x8944`) rejects `< 4.19` at `0x9218`, causing `libnetd_updatable_init` (`0x127ec`) to call `abort()`.

These three static disassembly findings will be validated against live runtime logs captured in `/cache/android17_boot/` during the controlled first boot before any BPF binary compatibility shim is applied.

---

## 4. Rebuilt Image & Rollback Baseline Checksums

| Artifact | Path | Size (Bytes) | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| **Unmodified Android 17 GSI** | [`BlazeAOSP-GSI/system.img`](file:///home/atilla/Desktop/Via%20S48/BlazeAOSP-GSI/system.img) | `3,408,465,920` | `4d1a5aee6a407bed3a3fe1fd571ab0db3b5577ebe31535c52b67b65d39843f03` |
| **Clean Rebuilt Android 17 Image** | [`BlazeAOSP-patched.img`](file:///home/atilla/Desktop/Via%20S48/BlazeAOSP-patched.img) | `3,460,825,088` | `1fe071403521c75cb9e65ad542d4b1ae85056619e11873a7e715b9f1368e0cdc` |
| **Verified Working Android 16 System** | [`Axion-2.9-patched.img`](file:///home/atilla/Desktop/Via%20S48/Axion-2.9-patched.img) | `3,070,558,208` | `40cd5fa1e4e2e8ed407a89907ad1c797ae409bb272127fab904e96787154aba1` |
| **Verified Working Boot Image** | [`boot_oc_2.1ghz_android16.img`](file:///home/atilla/Desktop/Via%20S48/boot_oc_2.1ghz_android16.img) | `11,079,680` | `879da6cfc5ce2e84cc5d54a21c69bbee28bf14278743dfe2ae1a711418fcbef9` |
| **Disabled VBMeta Image** | [`vbmeta_disabled.img`](file:///home/atilla/Desktop/Via%20S48/vbmeta_disabled.img) | `256` | `53ea4aee4a0f0fd0dc0dc4556bf3143f39ae7799aa55f35dbe8eb9ee482873f3` |
