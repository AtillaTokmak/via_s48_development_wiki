# Experiment EXP-006: BlazeAOSP Android 17 (Cinnamon Bun / API 37) Bring-Up

- **Device:** Casper VIA S48 (`MT8168` / `F716`)
- **OS Version:** Android 17 (`26Q2` / API Level 37.0 / `CP2A.260605.016`)
- **Target GSI:** `BlazeAOSP-GSI/system.img` (`3,408,465,920` bytes, SHA-256: `4d1a5aee6a407bed3a3fe1fd571ab0db3b5577ebe31535c52b67b65d39843f03`)
- **Kernel Baseline:** Linux 4.14.141 (`#10 SMP PREEMPT`, 2.1 GHz Overclocked + DFE Ramdisk, `boot_oc_2.1ghz_android16.img`)
- **Status:** **CLEAN REBUILD VALIDATED — READY FOR CONTROLLED DIAGNOSTIC BOOT TEST**

---

## 1. Executive Summary & Methodological Rules

Following the successful bring-up of AxionOS Android 16 (QPR2) on the Casper VIA S48 (`Linux 4.14.141`), an initial attempt to boot `BlazeAOSP` Android 17 returned to `LK` fastboot. Rather than applying speculative binary patches or upgrading the kernel prematurely, we conducted an exhaustive forensic comparison between our working `AxionOS` Android 16 system image, live tablet runtime state, and `BlazeAOSP` Android 17 using symbol extraction and ARM64 disassembly (`gdb` multi-arch).

All modifications in the rebuilt test image (`BlazeAOSP-patched.img`) are restricted to **independently verified, minimal corrections** (3 added files and 2 minimally edited `.rc` files removing `reboot_on_failure`), with **zero binary modifications**.

---

## 2. Independently Verified Corrections & Disassembly Proofs

### 2.1 VNDK 29 APEX Verification (`com.android.vndk.v29.apex`) — **CONFIRMED REQUIRED**
Casper VIA S48 vendor specifies `ro.vndk.version=29`. `BlazeAOSP-GSI/system.img` ships `com.android.vndk.v30.apex` through `v35`, omitting `v29`. Before injecting `com.android.vndk.v29.apex` from `Axion-2.9-GSI_treble_arm64-ab-VANILLA-20261017.img`, we verified:
- **File Size & Digest:** `103,755,776` bytes, SHA-256: `3a0aadbdaa0cb1177160f47ecb4458083e2db2c74150499a43ecdfa9c4c98433`
- **Protobuf Manifest (`apex_manifest.pb`):** `name: "com.android.vndk.v29"`, `version: 1` (`0a14636f6d2e616e64726f69642e766e646b2e7632391001`) — identical schema to `BlazeAOSP`'s `com.android.vndk.v31.apex`.
- **Outer APK Signature v2 (`0x7109871a`) & v3 (`0xf05368c0`):** Signed with the **exact same** X.509 certificate as `BlazeAOSP`'s `com.android.vndk.v31.apex` (`Subject: CN=com.android.vndk`, Serial `12E3A3541A784BB4ED05456D57319A1A8364779C`, Certificate SHA-256: `ae0a71129a4e7ac2e0fe7b8a124732bc01668d5820da8af97f246696dc50752c`, Public Key SHA-256: `d0568c78a94d3abe3fb3db03852b1fc56de0554f9e5c2d0b75f07346d4c38b13`).
- **Inner `apex_pubkey` & `dm-verity` Hashtree (`apex_payload.img`):** `apex_pubkey` (`1,032` bytes, SHA-256: `19d2131d80d4238098af9969233ab821199f0cfd059d817bc01c028ddf03e2f1`) is byte-for-byte identical to `BlazeAOSP`'s VNDK APEX key. `avbtool verify_image --image apex_payload.img --key apex_pubkey` passes (`SHA256_RSA4096`, Root Digest: `dbdde006184ad1b4504298b092d3f2f6c57b83068a27edd29544e99a90bc6b22`).
- **Filesystem & SELinux Labels:** `e2fsck -fn` reports `clean, 532/544 files, 25102/25110 blocks` (`255` 64-bit `.so` files in `/lib64`, `255` 32-bit `.so` files in `/lib`, `5` `.29.txt` config files in `/etc`). Outer file labeled `u:object_r:system_file:s0`; inner shared libraries labeled `u:object_r:system_lib_file:s0`.
- **LLNDK Dependencies & `linkerconfig` Integration:** All 20 libraries listed in `/etc/llndk.libraries.29.txt` exist in `BlazeAOSP` `/system/lib64/` and `/apex/com.android.runtime/lib64/bionic/`. Symbol table audit of `BlazeAOSP` `/system/bin/linkerconfig` (`BuildVndkNamespace` at `0x252c8`, `GetVndkProvider` at `0x27838`, `GetVendorVndkVersion` at `0x32af4`) confirms identical VNDK namespace generation to `AxionOS`.

---

### 2.2 `fstab.mt8168` Bind-Mount Ordering (`on early-fs`) & Content — **CONFIRMED FIX**
- **Previous Defect #1 (Init Ordering):** In Attempt 1, `casper_compat.rc` placed `mount none /system/etc/fstab.mt8168 /vendor/etc/fstab.mt8168 bind` inside `on post-fs`. In `BlazeAOSP` `/system/etc/init/hw/init.rc` (lines 567–575), `on late-init` triggers:
  1. `trigger early-fs` (line 568)
  2. `trigger fs` (line 574) — where stock `/vendor/etc/init/hw/init.mt8168.rc` (line 108) runs `mount_all /vendor/etc/fstab.mt8168`!
  3. `trigger post-fs` (line 575) — too late! `mount_all` had already executed against the unpatched stock `/vendor/etc/fstab.mt8168` (`checkpoint=block,fileencryption=aes-256-xts`).
- **Previous Defect #2 (Incomplete Synthetic `fstab.mt8168`):** The earlier synthetic `fstab.mt8168` omitted stock MediaTek entries (`/nvram`, `/proinfo`, `/lk`, `/para`, `/misc`, `/seccfg`, `/tee1`, `/tee2`, `/scp1`, `/sspm_1`, `/spmfw`, `/mcupm_1`, `zram0`).
- **Verified Correction:**
  - `/system/etc/fstab.mt8168` (`3,661` bytes, SHA-256: `eed78be91dfd421dc59d63f19f08c6643c27f6c4e13be6f228a42db2d35eb53c`) is now an exact copy of stock `/vendor/etc/fstab.mt8168` with **only** `,checkpoint=block,fileencryption=aes-256-xts` removed from line 134 (`/data`).
  - The bind-mount in `/system/etc/init/casper_compat.rc` now runs in `on early-fs`. Both `/system` and `/vendor` are already mounted in `first_stage_init` via the boot ramdisk `/fstab.mt8168` before `second_stage_init` parses `on early-fs`.

---

### 2.3 Restoration of Original Android 17 `netbpfload.rc` & BPF Chain Disassembly
- **Previous Defect:** Attempt 1 replaced `/system/etc/init/netbpfload.rc` with a 2-line stub (`on early-init` / `setprop bpf.progs_loaded 1`), deleting:
  1. Line 1 (`# 2026 2 37 0 0 #`), which `/apex/com.android.tethering/bin/netbpfload` parses via `fscanf(f, "# %d %d %d %d %d #", ...)` at `0xa5fc` (exiting with status `16` if missing).
  2. The `on load-bpf-programs` trigger block (lines 38–59: `exec_start bpfloader`, `wait_for_prop bpf.progs_loaded 1`, `exec_start netd1shot`, `start netd`), which Android 17 moved out of `/system/etc/init/hw/init.rc` and into `/system/etc/init/netbpfload.rc`.
  3. The base `service bpfloader ... updatable` declaration (lines 64–69) required by `perform_apex_config` when overriding `bpfloader` from `/apex/com.android.tethering/etc/netbpfload.35rc`.
- **Verified Restoration:** Restored the original 69-line `/system/etc/init/netbpfload.rc` from `BlazeAOSP-GSI/system.img`, removing only line 68 (`reboot_on_failure reboot,netbpfload-missing`).
- **Static ARM64 Disassembly Findings on the Android 17 BPF Chain (Documented for Runtime Validation):**
  Disassembly of the BPF execution chain in `BlazeAOSP` vs `AxionOS` identified three exact instructions where stock Android 17 binaries enforce kernel version checks on `Linux 4.14.141`:
  1. **`/apex/com.android.tethering/bin/netbpfload` (`blaze_netbpfload`):**
     - In `main()` (`0xa110..0xae90`), `blaze_netbpfload` checks `kernelVersion()` at `0xa2bc` (`cmp w8, #0x412`; `0x4120000` = `4.18.65535`). On `< 4.19`, it branches to `0xa454`, logs `"Unsupported kernel version ("`, and exits with code `5` (`mov w20, #0x5`). It also exits with code `6` on `< 5.4` (`0xa2f4`), code `7` on `< 5.10`/`< 5.15` (`0xa324`/`0xa34c`), and code `38` (`0xa9e0`) when `5.4+`/`5.10+` tethering BPF programs fail to load.
     - By contrast, `AxionOS`'s `/apex/com.android.tethering/bin/netbpfload` (`axion_netbpfload`) is compiled with Treble GSI `< 4.19` compatibility: it logs warnings instead of exiting, parses `# 2026 2 37 0 0 #` (`0xcf90`), creates `/sys/fs/bpf/tethering/map_kernel_bugs` (`0xd678`) and `/sys/fs/bpf/netd_shared/mainline_done` (`0xd8c8`), chains `uprobestatsbpfload` -> `/system/bin/bpfloader`, and sets `bpf.progs_loaded=1` (`0xcc48`) when re-invoked as `netbpfload done`.
  2. **`/system/bin/bpfloader` (`blaze_bpfloader`):**
     - Disassembly of `blaze_bpfloader` (`0xad48..0xd564`) shows 10 static 72-byte BPF file descriptors at `0x14e10..0x151cf`. When loading a `.bpf` object fails, `0xd0bc` checks `byte[64]` (`fail_on_error`): if `0`, it logs a warning and continues; if `1`, it calls `core::panicking::panic_fmt` (`0xd4cc`).
     - 9 of the 10 descriptors have `byte[64] = 0`. Only Entry 6 (`/system/etc/bpf/bpfRingbufProg.bpf`, testing Linux 5.8+ `BPF_MAP_TYPE_RINGBUF`) has `flags = 0x0101` (`byte[64] = 1` = `fail_on_error`, `byte[65] = 1` = `ignore_on_user_build`) at file offset `0x15000`. At `0xafe4..0xb060`, `byte[65] == 1` is skipped only when `ro.build.type == "user"`. Because `BlazeAOSP` has `ro.build.type=userdebug`, `bpfRingbufProg.bpf` is attempted and panics `bpfloader` on `Linux 4.14.141`.
  3. **`/apex/com.android.tethering/lib64/libnetd_updatable.so` (`blaze_libnetd_updatable.so`):**
     - In `libnetd_updatable_init.cfi` (`0x125b8..0x127ec`), if `BpfHandler::init()` (`0x8944`) returns an error (such as its `< 4.19` check at `0x9218`), `0x12758..0x127ec` logs at `FATAL` severity and calls `abort()`.
  - **Policy:** Per strict engineering discipline, **none** of these binary modifications are applied in `BlazeAOSP-patched.img` prior to capturing live runtime evidence from the first controlled boot test via `/cache/android17_boot/`.

---

### 2.4 `apexd` Loop-Device Race Audit — **HYPOTHESIS REFUTED (BINARY RESTORED UNMODIFIED)**
- **Investigation:** We extracted symbol tables (`blaze_apexd.sym.elf`) and disassembled `android::apex::loop::WaitForLoopDevice(int)` (`0xb4cf8`) and `CreateLoopDevice` (`0xb3740`) in `BlazeAOSP` `/system/bin/apexd`.
- **Disassembly Proof:**
  - When `WaitForLoopDevice` checks for `/dev/block/loopN` via `access()` (`0xb4e30`) and receives `ENOENT`, it calls `android::apex::loop::(anonymous namespace)::CreateLoopDeviceNode` (`0xb4e40` -> `0xba040`), which computes `makedev(7, N)` (`0xba0ac: orr w2, w8, #0x700`) and executes `mknod("/dev/block/loopN", S_IFBLK | 0600, dev)` (`0xba0cc: bl mknod@plt`) directly!
  - Furthermore, the earlier hardcoded offsets (`0x46794` / `0x46924`) did not reside in `CreateLoopDevice` at all; they were inside an unrelated `std::__1::__hash_table` template instantiation (`0x46738`).
- **Action Taken:** `/system/bin/apexd` is restored to **100% unmodified stock `BlazeAOSP`** (`SHA-256: 9c1810e3900a3657ff79db74723c2f16c64217b629a7226e13e6c01bc612f6d7`). Only the 2 `reboot_on_failure` lines in `/system/etc/init/apexd.rc` are removed so any `apexd` error is logged rather than triggering an immediate blind reboot.

---

### 2.5 `cgroups.json` & `libprocessgroup_setup` Audit — **CONFIRMED COMPATIBLE (RESTORED UNMODIFIED)**
- **Investigation:** Extracted `/system/etc/cgroups.json` and `/system/lib64/libprocessgroup_setup.so` from both `AxionOS` (Android 16) and `BlazeAOSP` (Android 17).
- **Proof:**
  - `/system/etc/cgroups.json` (`804` bytes, SHA-256: `ab2ed667ff45958843fb0c6ee953a5512def0ae87470c4358aa9576a6a4b2e22`) is **100% byte-for-byte identical** between `AxionOS` and `BlazeAOSP`.
  - Disassembly of `ActivateControllers` in `blaze_lpgs.so` (`0x6fd0..0x7398`) and `axion_lpgs.so` (`0x7060..0x7428`) shows identical machine code: when writing `+freezer` to `/sys/fs/cgroup/cgroup.subtree_control` fails on `Linux 4.14.141`, it logs an `ERROR` (`"Failed to activaterender controller..."`) and **continues without failing `SetupCgroups()`** because the legacy v1 `freezer` controller at `/sys/fs/cgroup/freezer` succeeds.
- **Action Taken:** `/system/etc/cgroups.json` is kept **100% unmodified**.

---

## 3. Rebuilt Test Image Specification & Complete File-Level Diff

- **Unmodified Source Image:** `/home/atilla/Desktop/Via S48/BlazeAOSP-GSI/system.img`
  - Size: `3,408,465,920` bytes
  - SHA-256: `4d1a5aee6a407bed3a3fe1fd571ab0db3b5577ebe31535c52b67b65d39843f03`
- **Rebuilt Patched Image:** `/home/atilla/Desktop/Via S48/BlazeAOSP-patched.img`
  - Size: `3,460,825,088` bytes (`844,928` 4KB blocks = `6,759,424` 512-byte sectors)
  - SHA-256: `1fe071403521c75cb9e65ad542d4b1ae85056619e11873a7e715b9f1368e0cdc`
  - Format: Raw `ext4` (`UUID=2fdde14e-7d6c-5725-9b1e-ff1a47ddf836`)
  - Filesystem Check (`e2fsck -fn`): `clean, 4526/4992 files (0.8% non-contiguous), 841872/844928 blocks` (`3,056` free 4KB blocks = `11.94 MiB` free inside image)
- **Live `super` Partition Layout (`lpdump /dev/block/by-name/super` on Active Tablet):**
  - Total `super` size: `5,891,620,864` bytes (`11,507,072` sectors); `main` group max size: `5,890,572,288` bytes (`11,505,024` sectors)
  - `vendor` (readonly, group `main`): `619,232` sectors (`317,046,784` bytes; physical sectors `2048..621280`)
  - `product` (readonly, group `main`): `2,244,272` sectors (`1,149,067,264` bytes; physical sectors `622592..2866864`)
  - `system` (readonly, group `main`): Currently `5,997,184` sectors (`3,070,558,208` bytes; 4 extents ending at physical sector `8862736`), automatically sized by `fastbootd` when `Axion-2.9-patched.img` (`3,070,558,208` bytes) was flashed during rollback
  - **Unallocated Free Space in `super`:** `2,642,288` contiguous sectors (`8862736..11505024`) = **`1,352,851,456` bytes (`1,290.18 MiB`)**
  - **Dynamic Partition Resize Requirement:** Flashing `BlazeAOSP-patched.img` (`3,460,825,088` bytes = `6,759,424` sectors) expands `system` by `762,240` sectors (`390,266,880` bytes) out of the `2,642,288` free sectors at the end of `super`, leaving `1,880,048` sectors (`962,584,576` bytes = `917.99 MiB`) free and leaving `vendor` and `product` completely untouched.

### Complete File-Level Diff (`BlazeAOSP-GSI/system.img` vs. `BlazeAOSP-patched.img`)

| Path in Image | Action | Mode / UID:GID | SELinux Context | Size (Bytes) | SHA-256 Checksum | Justification |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/system/system_ext/apex/com.android.vndk.v29.apex` | **Added** | `0644` / `0:0` | `u:object_r:system_file:s0` | `103,755,776` | `3a0aadbdaa0cb1177160f47ecb4458083e2db2c74150499a43ecdfa9c4c98433` | Provides mandatory VNDK 29 namespace for MT8168 Android 10 vendor blobs (`ro.vndk.version=29`). |
| `/system/etc/fstab.mt8168` | **Added** | `0644` / `0:0` | `u:object_r:system_file:s0` | `3,661` | `eed78be91dfd421dc59d63f19f08c6643c27f6c4e13be6f228a42db2d35eb53c` | Exact copy of stock `/vendor/etc/fstab.mt8168` with only `,checkpoint=block,fileencryption=aes-256-xts` removed from `/data`. |
| `/system/etc/init/casper_compat.rc` | **Added** | `0644` / `0:0` | `u:object_r:system_file:s0` | `2,976` | `34e2328c668d8e61733a8f40e563a4be584c3111948de66a4f2434a86b40a743` | Binds `/system/etc/fstab.mt8168` -> `/vendor/etc/fstab.mt8168` on `early-fs` (before `on fs` `mount_all`), enables early ADB, and writes diagnostic logs to `/cache/android17_boot/`. |
| `/system/etc/init/apexd.rc` | **Modified** | `0644` / `0:0` | `u:object_r:system_file:s0` | `1,229` (was `1,334`) | `d9eb6188a043bf04504951741a87a8dae18972ac4ce3c691d26d4df5363eec72` | Removed only the 2 `reboot_on_failure` lines so any `apexd` error is captured by `/cache/android17_boot/` instead of triggering a blind reboot. |
| `/system/etc/init/netbpfload.rc` | **Modified** | `0644` / `0:0` | `u:object_r:system_file:s0` | `2,904` (was `2,952`) | `3e486e9f3be12e07d0879cdcd4ac3779e526977270454f3ffebbde7c361c8ea5` | Restored original 69-line Android 17 `netbpfload.rc` (`# 2026 2 37 0 0 #` and `on load-bpf-programs` preserved), with only `reboot_on_failure reboot,netbpfload-missing` removed. |
| `/system/bin/apexd` | **Unchanged** | `0755` / `0:0` | `u:object_r:apexd_exec:s0` | `1,102,592` | `9c1810e3900a3657ff79db74723c2f16c64217b629a7226e13e6c01bc612f6d7` | Verified 100% identical to original `BlazeAOSP-GSI/system.img`. |
| `/system/bin/bpfloader` | **Unchanged** | `0755` / `0:0` | `u:object_r:bpfloader_platform_exec:s0` | `103,600` | `e1842b6eb1d19216b8c7a47b6f8128583e0faa0d0069a968d7730d2484d7ee38` | Verified 100% identical to original `BlazeAOSP-GSI/system.img`. |
| `/system/etc/cgroups.json` | **Unchanged** | `0644` / `0:0` | `u:object_r:cgroup_desc_file:s0` | `804` | `ab2ed667ff45958843fb0c6ee953a5512def0ae87470c4358aa9576a6a4b2e22` | Verified 100% identical to original `BlazeAOSP-GSI/system.img`. |

---

## 4. Early-Boot Diagnostic Strategy (`/cache/android17_boot/`)

Because `Linux 4.14.141` on F716 does not populate `ramoops`/`pstore` across warm reboots into `LK` fastboot, `/system/etc/init/casper_compat.rc` uses the dedicated `ext4` `/cache` partition (`/dev/block/platform/bootdevice/by-name/cache`, mounted during `on fs`) and the permissive `u:r:su:s0` SELinux domain (verified in `plat_sepolicy.cil`: `(typepermissive su)` and `(allow init su (process (transition)))`) to record persistent milestone telemetry:
1. **`01_post_fs.log` (`on post-fs`):** Synchronously written and `sync`'d immediately after `mount_all /vendor/etc/fstab.mt8168` mounts `/cache` and `/data`. Confirms `early-fs` bind-mount and `on fs` mount success.
2. **`casper_bootlog` Background Service (`on post-fs`):** Continuously streams `/dev/kmsg` (`kmsg_live.log`), `logcat -b all` (`logcat_live.log`), `getprop` (`props_latest.log`), and `/apex` + `/dev/block/loop*` (`apex_latest.log`) to `/cache/android17_boot/` with 1-second `sync`.
3. **`02_post_fs_data.log` (`on post-fs-data`):** Synchronously written after `wait_for_prop apexd.status activated` and `perform_apex_config` complete.
4. **`02b_pre_bpf.log` (`on load-bpf-programs`):** Runs immediately before `netbpfload.rc`'s `on load-bpf-programs` (`exec_start bpfloader`) due to alphabetical `.rc` parse ordering (`casper_compat.rc` before `netbpfload.rc`).
5. **`03_zygote_start.log` & `04_boot.log`:** Capture late-boot transition into `zygote` and `boot`.

If the tablet drops back to fastboot during the test boot, flashing back `Axion-2.9-patched.img` (without erasing `cache`) allows us to read `/cache/android17_boot/*` directly from Android 16 via `adb shell` and see the exact failing process, exit code, and kernel/logcat messages.

---

## 5. Controlled `fastbootd` Test & Rollback Procedure (Awaiting Explicit User Authorization)

> [!CAUTION]
> **DO NOT EXECUTE WITHOUT EXPLICIT USER AUTHORIZATION.**
> Do not erase `cache` or `userdata` during rollback so that `/cache/android17_boot/` diagnostic logs and the user's Android 16 state remain preserved.

### Phase A: Controlled Android 17 Diagnostic Test Boot
```bash
# 1. Clear any stale /cache/android17_boot logs and reboot from Android 16 into userspace fastbootd
HOME=/tmp adb -P 5038 shell "rm -rf /cache/android17_boot && sync"
HOME=/tmp adb -P 5038 reboot fastboot

# 2. Verify userspace fastbootd mode (must return "yes")
fastboot getvar is-userspace

# 3. Explicitly resize logical partition system from 3,070,558,208 to 3,460,825,088 bytes
#    (Allocates 762,240 sectors from the 2,642,288 free sectors at the end of super; vendor and product untouched)
fastboot resize-logical-partition system 3460825088

# 4. Flash only the rebuilt Android 17 system image
#    Preserve existing boot (boot_oc_2.1ghz_android16.img), vendor, product, userdata, and cache
fastboot flash system "/home/atilla/Desktop/Via S48/BlazeAOSP-patched.img"

# 5. Reboot and monitor for 60 seconds
fastboot reboot
```

### Phase B: Immediate Rollback to Verified Android 16 Baseline & Log Retrieval
If the tablet returns to `LK` fastboot (or fails to reach `adb` within 60 seconds):
```bash
# 1. From LK fastboot, enter userspace fastbootd
fastboot reboot fastboot

# 2. Restore the verified working Android 16 system image (3,070,558,208 bytes)
#    SHA-256: 40cd5fa1e4e2e8ed407a89907ad1c797ae409bb272127fab904e96787154aba1
fastboot flash system "/home/atilla/Desktop/Via S48/Axion-2.9-patched.img"

# 3. Reboot back into working Android 16
fastboot reboot

# 4. Once booted, pull the persistent Android 17 early-boot logs from /cache/android17_boot/
HOME=/tmp adb -P 5038 wait-for-device
HOME=/tmp adb -P 5038 pull /cache/android17_boot "/home/atilla/Desktop/Via S48/android17_investigation/boot_logs_attempt2"
```
