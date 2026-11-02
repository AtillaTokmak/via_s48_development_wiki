# Kernel Version Technical Comparison Matrix (`4.14` – `5.15`)

- **Device:** Casper VIA S48 (`MT8168` / `F716`)
- **Active Baseline:** `Linux 4.14.141` (`kernel_casper_mt8168`, branch `android-16` / `oc-2.1ghz-experimental`)
- **Vendor Baseline:** Stock Android 10 (`VNDK 29`, `target-level="4"`)
- **Status:** **COMPLETE COMPARATIVE EVALUATION**

---

## 1. Candidate Kernel Families Overview

This document evaluates five candidate Linux LTS kernel families for the Casper VIA S48 (`MT8168` / `F716`):
- **Linux 4.14.x** (Current MediaTek ALPS Android 10 BSP baseline — `4.14.141`)
- **Linux 4.19.x** (MediaTek ALPS Android 10/11 BSP generation — `4.19.127` / `4.19.191`)
- **Linux 5.4.x** (Android 11/12 Common Kernel & early MediaTek IoT Yocto `MT8365` BSP)
- **Linux 5.10.x** (Android 12 GKI 2.0 & MediaTek IoT Yocto `MT8365` / Genio 350 BSP)
- **Linux 5.15.x** (Android 13/14 GKI 2.0 & MediaTek Genio 350 / Mainline `MT8365` hybrid)

---

## 2. Master Feasibility Matrix

| Evaluation Criterion | Linux 4.14.141 (Current Baseline) | Linux 4.19.x (MediaTek ALPS BSP) | Linux 5.4.x (MT8365 IoT / ACK) | Linux 5.10.x (MT8365 IoT / GKI) | Linux 5.15.x (Genio 350 / Mainline) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. MT8168 Platform Availability** | **Full Native Support** (`CONFIG_MACH_MT8168=y` in `kernel_casper_mt8168`) | **High** (Public MT8168/MT8163/MT6765 ALPS `kernel-4.19` trees exist from OEMs like Amazon/Lenovo) | **Partial (`MT8365` AIoT)** (MediaTek Pumpkin/IoT BSP targets industrial `MT8365`, not tablet `MT8168`) | **Partial (`MT8365` AIoT)** (MediaTek IoT Yocto SDK supports `MT8365` EVK) | **Partial (`MT8365` Mainline/IoT)** (`arch/arm64/boot/dts/mediatek/mt8365.dtsi` upstreamed by BayLibre/MTK) |
| **2. Critical Hardware Driver Readiness (`F716`)** | **100% Verified Working** (eMMC, Display, Touch, GPU, Wi-Fi/BT, Audio, USB, PMIC, 2.1 GHz OC) | **Moderate** (Requires porting F716 DTS, `jd936x` LCM, `gslX680` touch, and finding/building 4.19 `gen4m` `.ko` modules) | **Low** (Uses DRM instead of `mtkfb`; breaks stock Android 10 HWC2, Audio, Camera, and `.ko` modules) | **Very Low** (`ION` removed in favor of `DMABUF_HEAPS`; breaks stock `gralloc.mt8168.so`, `libGLES_mali.so`, HWC2, Wi-Fi) | **Very Low** (Pure DRM/V4L2/Mainline architecture; completely incompatible with Android 10 VNDK-29 vendor blobs) |
| **3. Android 17 Capability Coverage** | **High with Userspace Shims** (Boots Android 16 live; supports mount ns, dm-verity, PSI, uclamp, hybrid cgroups; needs userspace BPF/FUSE-BPF overrides) | **High with Userspace Shims** (Adds cgroup v2 freezer backports in later 4.19 ACK, but **still** lacks Linux 5.8+ `BPF_MAP_TYPE_RINGBUF` / BTF for `netbpfload`!) | **Moderate-High** (Has native cgroup v2 freezer & `clone3`, but still requires backports for `BPF_MAP_TYPE_RINGBUF` and `FUSE-BPF`) | **High** (Native `BPF_MAP_TYPE_RINGBUF`, BTF, cgroup v2 freezer, modern syscalls) | **Native GKI Compliance** (Supports all Android 16/17 eBPF, cgroup v2, and MGLRU features natively) |
| **4. Kernel Source Availability** | **100% Available in Workspace** (`kernel_casper_mt8168`) | **Third-Party OEM Tarballs Only** (No official Casper F716 4.19 source; connectivity `.ko` sources missing) | **IoT Yocto BSP Only** (Lacks Android `mtkfb`, `sensors-1.0`, `cameraisp`, and `CONSYS_8168` Android wrappers) | **IoT Yocto BSP Only** (Designed for Yocto Linux/Wayland, not Android 10 Treble vendor HALs) | **Mainline + IoT Yocto** (Designed for mainline Linux, not Android 10 Treble) |
| **5. Mali-G52 GPU Compatibility** | **100% Compatible** (`mali bifrost r20p0` + `mt8168-gpufreq` + `ged` + `mtk_ion` matches `/vendor/lib64/egl/libGLES_mali.so`) | **Port Required** (`mali-r20p0` must be forward-ported to 4.19 timer/mm APIs while keeping `r20p0` ioctl ABI) | **Major Port Required** (`mali-r20p0` does not compile on 5.4 `mm_struct` / `vm_fault_t` / `dma_buf` without heavy backports) | **Critical Blocker** (`mali-r20p0` depends on staging `ION`, which was removed in 5.10/5.11; Panfrost cannot load `libGLES_mali.so`) | **Critical Blocker** (Incompatible with `r20p0` blob and `/dev/ion` gralloc) |
| **6. MT6631 Connectivity (`wmt_drv` / `wlan_drv_gen4m`)** | **100% Working** (Prebuilt 4.14.141 `.ko` modules embedded via `CONFIG_VIA_S48_CONNSYS_PRELOAD=y`) | **Critical Blocker** (4.14.141 `.ko` blobs cannot load on 4.19; requires full out-of-tree `gen4m` + `wmt_drv` source code) | **Critical Blocker** (4.14.141 `.ko` blobs cannot load on 5.4; out-of-tree source unavailable) | **Critical Blocker** (4.14.141 `.ko` blobs cannot load on 5.10; out-of-tree source unavailable) | **Critical Blocker** (4.14.141 `.ko` blobs cannot load on 5.15; out-of-tree source unavailable) |
| **7. Display & Touchscreen (`EK79029` / `JD9365DA` + `GSL1680`)** | **100% Working** (Repaired F716 gamma/timing `d60cc9645` and 5247-record GSL1680 fw `20189f473`) | **Portable** (Uses same `mtkfb`/`lcm` & `mtk_tpd` architecture in ALPS 4.19) | **Incompatible Architecture** (5.4 IoT BSP uses DRM/KMS instead of `mtkfb`, breaking `hwcomposer.mt8168.so`) | **Incompatible Architecture** (Uses DRM/KMS; breaks stock Android 10 `hwcomposer.mt8168.so`) | **Incompatible Architecture** (Uses DRM/KMS; breaks stock Android 10 `hwcomposer.mt8168.so`) |
| **8. Device Tree (`mt8168.dtsi` + `f716.dts`) Compatibility** | **100% Native** (`Image.gz-dtb` with appended `mt8168_dtbo` + `tb8168p1_64_bsp_f716` overlay) | **Moderate Effort** (Clock/pinctrl/larb bindings mostly match ALPS 4.19, minor property updates needed) | **Incompatible** (`mt8365.dtsi` uses different clock IDs, pinctrl v2 macros, and DRM display nodes) | **Incompatible** (Complete DT rewrite required; breaks LK bootloader DTBO overlay fixups!) | **Incompatible** (Complete DT rewrite required; breaks LK bootloader DTBO overlay fixups!) |
| **9. Bootloader (`LK`) & Boot Image Requirements** | **100% Native** (Boot Header v2, base `0x40000000`, `Image.gz-dtb` + DTBO overlay) | **Compatible** (Same LK `Image.gz-dtb` + DTBO overlay mechanism) | **High Risk** (LK applies runtime FDT overlay fixups onto `__symbols__` expected from ALPS `mt8168.dtsi`) | **High Risk** (LK FDT overlay fixups fail if base DTB node paths/symbols change) | **High Risk** (LK FDT overlay fixups fail if base DTB node paths/symbols change) |
| **10. Estimated Porting Complexity** | **Low** (Targeted backports & userspace shims only) | **High** (Full BSP bring-up + missing Wi-Fi/BT module source) | **Extreme** (Requires forward-porting all of `drivers/misc/mediatek/` from 4.14 to 5.4) | **Infeasible without Full Vendor Source** (Requires replacing Android 10 vendor blobs or porting legacy ION/mtkfb/r20p0 to 5.10) | **Infeasible without Full Vendor Source** |
| **11. Recommended Priority** | **PRIORITY 1 (Primary Recommended Target)** | **PRIORITY 2 (Secondary Reference / Cherry-Pick Donor)** | **Not Recommended for Full Port** (Use as backport donor only) | **Not Recommended for Full Port** (Use as backport donor only) | **Not Recommended** |

---

## 3. Detailed Evaluation by Kernel Family

### 3.1 Linux 4.14.x (`4.14.141` Current Baseline — `kernel_casper_mt8168`)
- **Strengths:**
  - 100% of Casper VIA S48 (`F716`) hardware works out-of-the-box, including our custom 2.1 GHz CPU overclock (`d6ce48d16`, `84ff56e2a`), repaired `EK79029`/`JD9365DA` LCD panel timing (`d60cc9645`), 5,247-record `GSL1680` touchscreen firmware (`20189f473`), and in-kernel `via_s48_connsys_preload` (`f4abafafd`) for Wi-Fi/BT/GPS.
  - Already includes critical backports from Linux 4.20–5.4: **PSI (`CONFIG_PSI=y`)**, **Task & Group Utilization Clamping (`CONFIG_UCLAMP_TASK=y`, `CONFIG_UCLAMP_TASK_GROUP=y`)**, **Speculative Page Fault (`CONFIG_SPECULATIVE_PAGE_FAULT=y`)**, **Cgroup v2 unified hierarchy (`cgroup2`)**, and **Cgroup BPF (`CONFIG_CGROUP_BPF=y`)**.
  - **Empirically proven** to boot AxionOS Android 16 (API 36) to the launcher with working Wi-Fi, display, touch, audio, and 2.1 GHz scaling.
- **Weaknesses:**
  - Lacks Linux 5.8+ `BPF_MAP_TYPE_RINGBUF` and BTF (requiring userspace `netbpfload` bypass or backport).
  - Lacks unified cgroup v2 `cgroup.freeze` (uses cgroup v1 `CONFIG_CGROUP_FREEZER=y`).
  - Lacks `CONFIG_FUSE_BPF` (uses standard `/dev/fuse` via `persist.sys.fuse.bpf.override=false`).

### 3.2 Linux 4.19.x (MediaTek ALPS `kernel-4.19`)
- **Technical Reality Check:**
  - Does upgrading from `4.14` to `4.19` eliminate the need for Android 17 userspace compatibility patches? **No.**
  - Linux 4.19 still lacks native `BPF_MAP_TYPE_RINGBUF` (introduced in Linux 5.8) and mainline `FUSE-BPF`, and still fails official Android 16/17 GKI VINTF kernel version checks unless overridden.
  - Meanwhile, migrating to `4.19` immediately breaks our prebuilt `4.14.141` connectivity modules (`wmt_drv.ko`, `bt_drv.ko`, `wmt_chrdev_wifi.ko`, `wlan_drv_gen4m.ko`) because their `vermagic` and symbol CRCs are locked to `4.14.141`.
  - **Best Use of `4.19`:** Use Google's `android-4.19-stable` and MediaTek's `kernel-4.19` trees as **clean backport donors** for isolated features (such as Cgroup v2 `cgroup.freeze`, `clone3`, `faccessat2`, and BPF map helpers) into our working `4.14.141` tree!

### 3.3 Linux 5.4.x, 5.10.x, and 5.15.x (`MT8365` IoT / Mainline)
- **Why `MT8365` 5.x Trees Cannot Drop-In Replace `MT8168` 4.14 on Casper VIA S48:**
  1. **LK Bootloader DTBO Fixup Contract:** When the Casper VIA S48 Little Kernel (`LK`) boots, it loads the base DTB (`mt8168_dtbo`) and dynamically merges the board overlay (`tb8168p1_64_bsp_f716`) using `__fixups__` symbols (`odm`, `pio`, `mt_pmic_vsim1_ldo_reg`, `afe`, `consys`, `kd_camera_hw1`, `ssusb`, `touch`). Mainline/IoT `mt8365.dtsi` does not define these symbol names or node structures, causing `LK` FDT overlay merging to fail before the kernel even starts (`fastboot` fallback in `LK`!).
  2. **Legacy Vendor Blob Lock-In:** Our `/vendor` partition is Android 10 (`VNDK 29`). Its closed-source HALs (`hwcomposer.mt8168.so`, `gralloc.mt8168.so`, `libGLES_mali.so`, `audio.primary.mt8168.so`, `camerahalserver`, `wmt_loader`) are hardwired to MediaTek's 4.14 kernel drivers (`/dev/mtkfb`, `/dev/ion`, `/dev/mali0` `r20p0`, `/dev/stpwmt`). None of these proprietary interfaces exist in mainline or 5.x `MT8365` IoT kernels.

---

## 4. Conclusion & Priority Ranking

1. **Priority 1 — Keep Linux `4.14.141` (`kernel_casper_mt8168`) + Targeted Backports & Userspace Shims:**
   - Preserves 100% hardware functionality, 2.1 GHz overclocking, LK DTBO compatibility, and Wi-Fi/BT module preloading while directly solving Android 17 requirements.
2. **Priority 2 — Use `android-4.19-stable` / `android-5.4` as Cherry-Pick Donors Only:**
   - Backport self-contained kernel features (`cgroup.freeze`, `clone3`, `faccessat2`, `BPF_MAP_TYPE_RINGBUF`) into `4.14.141` on an experimental branch if required by Android 17 services.
3. **Priority 3 — Do Not Attempt Full Migration to `4.19` / `5.4` / `5.10` / `5.15`:**
   - Technically unjustified given the loss of Wi-Fi/BT (`wmt_drv.ko` / `wlan_drv_gen4m.ko`), `mtkfb` HWC2, `mali-r20p0` + `mtk_ion`, and LK DTBO handoff.
