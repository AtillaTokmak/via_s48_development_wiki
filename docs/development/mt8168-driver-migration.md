# MT8168 / F716 Hardware Driver Inventory & Migration Audit

- **Device:** Casper VIA S48 (`MT8168` / `F716` Board Revision)
- **SoC Family:** MediaTek MT8168 (Consumer Tablet) / MT8365 (AIoT / Genio 350 Equivalent)
- **Active Kernel Tree:** `https://github.com/AtillaTokmak/kernel_casper_mt8168` (`Linux 4.14.141`)
- **Base SoC Device Tree:** `arch/arm64/boot/dts/mediatek/mt8168.dtsi`
- **Board Overlay Device Tree:** `arch/arm64/boot/dts/mediatek/tb8168p1_64_bsp_f716.dts`
- **Kernel Configuration:** `arch/arm64/configs/tb8168p1_64_bsp_f716_defconfig`

---

## 1. Executive Summary

To evaluate whether the Casper VIA S48 (`MT8168` / `F716`) can migrate from `Linux 4.14.141` to a newer kernel family (`4.19`, `5.4`, `5.10`, or `5.15`), every hardware driver required to reach the Android launcher and support core tablet peripherals must be audited.

> [!CAUTION]
> **Critical Architectural Constraint: Kernel-to-Userspace Vendor Blob ABI Coupling**
> Even where a modern mainline or MediaTek IoT (`MT8365`) kernel driver exists for a hardware block (such as DRM/KMS for display or Panfrost for the Mali-G52 GPU), the Casper VIA S48 relies on **Android 10 (VNDK 29) proprietary vendor HALs** (`hwcomposer.mt8168.so`, `gralloc.mt8168.so`, `libGLES_mali.so`, `libmtkcam_*.so`, `wmt_loader`).
>
> These closed-source vendor binaries do **not** speak standard mainline Linux DRM/V4L2/cfg80211 APIs; they issue MediaTek-proprietary `ioctl` calls to `/dev/mtkfb`, `/dev/ion`, `/dev/mali0` (`r20p0`), `/dev/stpbt`, `/dev/stpwmt`, and `/dev/camera-isp`. Replacing a 4.14 MediaTek BSP driver with an upstream mainline driver breaks the Android 10 vendor HALs unless the corresponding userspace HAL is also completely replaced from source.

---

## 2. Classification Categories

Every essential F716 driver is classified into one of six migration readiness categories:
1. **Available Upstream:** Supported in mainline Linux (`torvalds/linux`) under `mt8168` or `mt8365`.
2. **Available in Another MediaTek BSP:** Present in MediaTek ALPS `kernel-4.19` or IoT Yocto `mt8365` (`5.4` / `5.10` / `5.15`) BSP trees.
3. **Portable with Small Modifications:** Self-contained C driver (`< 2,000` LOC) using standard Linux I2C/GPIO/Input/Regulator APIs.
4. **Requires Substantial Porting:** Large MediaTek subsystem (`10,000+` LOC) deeply tied to `drivers/misc/mediatek/` (`mt-plat`), VFS/MM internals, or legacy timer/proc APIs.
5. **Proprietary or Source Unavailable:** Shipped only as prebuilt `.ko` or coupled to closed-source userspace blobs with strict ABI checks.
6. **Unknown:** Undocumented hardware interface or calibration data without public specification.

---

## 3. Master Hardware Driver Inventory (`tb8168p1_64_bsp_f716`)

### 3.1 Boot-Critical & Core SoC Infrastructure

| Subsystem | Hardware Block / Compatible | Exact Source Path in `kernel_casper_mt8168` | Kconfig Symbol(s) | Migration Classification | Technical Notes & Migration Blockers |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CPU & SMP** | 4x ARM Cortex-A53 (`arm,cortex-a53`, `arm,psci-1.0`) | `arch/arm64/kernel/psci.c`, `mt8168.dtsi` | `CONFIG_ARM_PSCI_FW=y`, `CONFIG_SMP=y` | **Available Upstream** | Standard PSCI 1.0 SMC calls handled by ATF (`atf-reserved-memory@54600000`). Works across all kernels. |
| **CPUFreq & 2.1 GHz Overclock** | MT8168 CPU DVFS + SVS (`mediatek,mt8168-svs`) | `drivers/cpufreq/cpufreq-dt.c`, `drivers/misc/mediatek/base/power/ mt8168/` | `CONFIG_CPUFREQ_DT=y`, `CONFIG_MTK_BASE_POWER=y`, `CONFIG_MTK_UNIFY_POWER=y` | **Portable with Small Modifications** | Uses standard `operating-points-v2` in `mt8168.dtsi` (`850 MHz` – `2.100 GHz`) plus MediaTek power throttle table (`NR_MAX_OPP_TBL=32`). |
| **Interrupts** | ARM GICv3 + MTK SysIRQ + CIRQ (`arm,gic-v3`, `mediatek,mt8168-sysirq`) | `drivers/irqchip/irq-gic-v3.c`, `drivers/irqchip/irq-mtk-sysirq.c`, `drivers/misc/mediatek/cirq/` | `CONFIG_ARM_GIC_V3=y`, `CONFIG_MTK_SYSIRQ=y`, `CONFIG_MTK_GIC_V3_EXT=y` | **Available Upstream** | Upstream `irq-mtk-sysirq.c` and `irq-mtk-cirq.c` exist in 4.19–5.15 (`mt8365.dtsi`). |
| **System Clocks (CCF)** | TOPCKGEN, INFRACFG, APMIXEDSYS, MMSYS, CAMSYS, VDECSYS, VENCSYS | `drivers/clk/mediatek/clk-mt8168*.c` | `CONFIG_COMMON_CLK_MT8168=y` (+ `_APU`, `_CAMSYS`, `_MFGCFG`, `_MMSYS`, `_VDECSYS`, `_VENCSYS`) | **Available Upstream / MTK BSP** | Mainline 5.15+ has `clk-mt8365*.c`; ALPS 4.19 has `clk-mt8168*.c`. Note: DT clock IDs in `dt-bindings/clock/mt8168-clk.h` differ between vendor 4.14 and mainline `mt8365-clk.h`. |
| **Pinctrl & GPIO** | MT8168 Pin Controller (`mediatek,mt8168-pinctrl`) | `drivers/pinctrl/mediatek/pinctrl-mt8168.c`, `pinctrl-mtk-common.c` | `CONFIG_PINCTRL_MTK=y`, `CONFIG_PINCTRL_MT8168=y` | **Available Upstream / MTK BSP** | Mainline uses `pinctrl-mt8365.c` (Paris/Moore v2 architecture), whereas 4.14 uses `pinctrl-mt8168.c` (v1). DT pinmux macros require translation if moving to 5.x. |
| **Power Domains (SCPSYS)** | MT8168 SCPSYS (`mediatek,mt8168-scpsys`) | `drivers/soc/mediatek/mtk-scpsys.c`, `mtk-scpsys-mt8168.c` | `CONFIG_MTK_SCPSYS=y`, `CONFIG_MTK_SCPSYS_MT8168=y` | **Available Upstream / MTK BSP** | Controls power domains for `DISP`, `AUDIO`, `CAM`, `VDEC`, `VENC`, `MFG` (GPU), `CONN`, and `DSP`. |
| **PMIC Wrapper (PWRAP)** | MT8168 PWRAP SPI (`mediatek,mt8168-pwrap`) | `drivers/soc/mediatek/mtk-pmic-wrap.c`, `drivers/misc/mediatek/pmic_wrap/` | `CONFIG_MTK_PMIC_WRAP=y`, `CONFIG_MTK_PMIC_WRAP_HAL=y` | **Available Upstream / MTK BSP** | Bridges SoC to MT6357 PMIC at `0x1000d000`. Supported in mainline `mtk-pmic-wrap.c`. |
| **PMIC Core, Regulators & RTC** | MediaTek MT6357 PMIC (`mt6357-pmic`) | `drivers/mfd/mt6358-core.c`, `drivers/misc/mediatek/pmic/mt6357/`, `drivers/rtc/rtc-mt6358.c` | `CONFIG_MFD_MT6358=y`, `CONFIG_MTK_PMIC_NEW_ARCH=y`, `CONFIG_RTC_DRV_MT6358=y` | **Available Upstream / MTK BSP** | Mainline 5.4+ supports `mt6357` via `drivers/mfd/mt6397-core.c` and `drivers/regulator/mt6357-regulator.c`, but regulator node names in `tb8168p1_64_bsp_f716.dts` (`mt_pmic_vsim1_ldo_reg`, `mt_pmic_vemc_ldo_reg`) follow MediaTek 4.14/4.19 conventions. |
| **IOMMU (M4U) & SMI** | MT8168 M4U & SMI LARB0..3 (`mediatek,mt8168-m4u`, `mediatek,mt8168-smi-larb`) | `drivers/iommu/mtk_iommu.c`, `drivers/memory/mtk-smi.c`, `drivers/misc/mediatek/pseudo_m4u/` | `CONFIG_MTK_IOMMU=y`, `CONFIG_MTK_SMI=y`, `CONFIG_MTK_PSEUDO_M4U=y` | **Requires Substantial Porting** | While `mtk_iommu.c` and `mtk-smi.c` exist upstream, MediaTek's 4.14 camera/display/ION stack depends on `pseudo_m4u` (`mediatek,mtk-pseudo-m4u`) and custom M4U port bindings (`dt-bindings/memory/mt8168-larb-port.h`). |
| **CMDQ / GCE Mailbox** | Global Command Engine (`mediatek,gce`, `mediatek,mailbox-gce`) | `drivers/mailbox/mtk-cmdq-mailbox.c`, `drivers/misc/mediatek/cmdq/v3/` | `CONFIG_MTK_CMDQ_MBOX=y`, `CONFIG_MTK_CMDQ=y` | **Requires Substantial Porting** | Used by Display (`dispsys`), MDP, and Video codecs to batch register writes on VSYNC. Vendor `hwcomposer.mt8168.so` and `libdpframework.so` issue custom ioctls to `/dev/mtk_cmdq`. |

---

### 3.2 Storage, Memory, and USB (Launcher-Critical)

| Subsystem | Hardware Block / Compatible | Exact Source Path in `kernel_casper_mt8168` | Kconfig Symbol(s) | Migration Classification | Technical Notes & Migration Blockers |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **eMMC 5.1 & MicroSD Storage** | `mmc0@11230000` (eMMC HS200 + HW CQ), `mmc1@11240000` (SD UHS) | `drivers/mmc/host/mtk-sd.c`, `drivers/mmc/core/block.c` | `CONFIG_MMC_MTK=y`, `CONFIG_MTK_EMMC_HW_CQ=y`, `CONFIG_MTK_BLOCK_TAG=y` | **Available Upstream / MTK BSP** | `drivers/mmc/host/mtk-sd.c` is upstream (`mediatek,mt8168-mmc` / `mt8365-mmc`), though 4.14 includes MediaTek's custom software Command Queue (`CONFIG_MTK_EMMC_HW_CQ=y`) and `blocktag` telemetry. |
| **USB 2.0 OTG / Gadget / Host** | MediaTek MTU3 + XHCI + T-PHY (`usb@11201000`, `usb-phy@11cc0000`) + Roco OTG | `drivers/usb/mtu3/`, `drivers/usb/host/xhci-mtk.c`, `drivers/phy/mediatek/phy-mtk-tphy.c` | `CONFIG_USB_MTU3=y`, `CONFIG_USB_MTU3_DUAL_ROLE=y`, `CONFIG_USB_XHCI_MTK=y`, `CONFIG_PHY_MTK_TPHY=y`, `CONFIG_ROCO_OTG_EN=y`, `CONFIG_EXTCON_USB_CHG=y` | **Available Upstream (Core) + Portable (Board OTG)** | Core `mtu3`, `xhci-mtk`, and `phy-mtk-tphy` are upstream. F716 board uses custom `roco,otgen` (`CONFIG_ROCO_OTG_EN`) and `linux,extcon-usb-chg` (`GPIO 39` ID, `GPIO 80` OTG EN, `GPIO 40` VBUS) which are small C drivers. |
| **Memory Allocator (ION)** | MediaTek ION (`mtk_ion`) | `drivers/staging/android/ion/`, `drivers/staging/android/ion/mtk/` | `CONFIG_ION=y`, `CONFIG_MTK_ION=y` | **Requires Substantial Porting (Blocker on 5.10+)** | Linux 5.10+ replaced staging `ION` with `DMA-BUF Heaps` (`CONFIG_DMABUF_HEAPS`). However, Casper VIA S48's Android 10 vendor `gralloc.mt8168.so`, `libGLES_mali.so`, and camera HALs open `/dev/ion` and issue `ION_IOC_CUSTOM` (`ION_CMD_MULTIMEDIA`) ioctls! Moving to 5.10+ requires forward-porting MediaTek's legacy `drivers/staging/android/ion/mtk/` intact. |

---

### 3.3 Display, Touchscreen, and GPU (UI / Launcher-Critical)

| Subsystem | Hardware Block / Compatible | Exact Source Path in `kernel_casper_mt8168` | Kconfig Symbol(s) | Migration Classification | Technical Notes & Migration Blockers |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Display Pipeline (`dispsys` / `mtkfb`)** | OVL0, OVL0_2L, RDMA0/1, WDMA0, COLOR0, CCORR0, AAL0, GAMMA0, DITHER0, DSI0 (`14000000.dispsys`) | `drivers/misc/mediatek/video/mt8168/dispsys/`, `drivers/misc/mediatek/video/mt8168/videox/` | `CONFIG_MTK_FB=y`, `CONFIG_MTK_LCM=y` (`# CONFIG_DRM_MEDIATEK is not set`) | **Requires Substantial Porting (CRITICAL BLOCKER)** | Our 4.14 kernel does **NOT** use mainline `CONFIG_DRM_MEDIATEK`! It uses MediaTek's legacy `mtkfb` / `videox` / `lcm` architecture. Vendor `hwcomposer.mt8168.so` (`android.hardware.graphics.composer@2.1-service`) communicates exclusively with `/dev/mtkfb` via `MTKFB_IOCTL_*`. Mainline/5.x kernels only support `CONFIG_DRM_MEDIATEK` (`/dev/dri/card0`), which is 100% incompatible with our stock `hwcomposer.mt8168.so` and LK bootloader handoff. |
| **LCD Panel (LCM)** | Fitipower `EK79029` / Jadard `JD9365DA` 800x1280 WXGA MIPI-DSI IPS (`kd,mid-lcm`) | `drivers/misc/mediatek/lcm/jd936x_wxga_dsi_vdo/jd936x_wxga_dsi_vdo.c`, `f716_fx_ek79029_boe_wxga_ips_8.h` | `CONFIG_CUSTOM_KERNEL_LCM="jd936x_wxga_dsi_vdo"` | **Portable with Small Modifications (within MTK LCM) / Rewrite for DRM** | Uses custom F716 GPIO sequencing (`gpio_lcd_pwr` = GPIO 135, `gpio_lcd_rst` = GPIO 20, `gpio_lcd_vdd` = GPIO 133, `vsim1` 1.8V LDO) and a custom 210-register gamma/timing table (`f716_fx_ek79029_boe_wxga_ips_8.h`) that we specifically repaired in commit `d60cc9645`. |
| **Touchscreen** | Silead GSL1680 (`mediatek,cap_touch` on I2C2 `@0x40`) | `drivers/input/touchscreen/mediatek/mtk_tpd.c`, `drivers/input/touchscreen/mediatek/gslX680/mt8167/mtk_gslX680.c`, `f716_gsl1680_wxga.h` | `CONFIG_TOUCHSCREEN_MTK=y`, `CONFIG_TOUCHSCREEN_MTK_GSLX68X=y` | **Portable with Small Modifications** | Relies on the 5,247-record `F716_GSL1680_WXGA` microcode array (`f716_gsl1680_wxga.h`) and Silead `gsl_DataInit` / `SL_Adjust_Freq` algorithm (`gsl_point_id_point_active_pen_no_gesture.c`, commit `20189f473`). Can be ported to any kernel, though it currently wraps MediaTek's `tpd` framework (`mtk_tpd.c`). |
| **GPU (3D Graphics)** | ARM Mali-G52 MC1 (`mali@13040000`, `mediatek,mt8168-gpufreq`) | `drivers/misc/mediatek/gpu/gpu_mali/mali_bifrost/mali-r20p0/`, `drivers/misc/mediatek/base/power/gpufreq_v1/mt8168/` | `CONFIG_MTK_GPU_SUPPORT=y`, `CONFIG_MTK_GPU_VERSION="mali bifrost r20p0"` | **Requires Substantial Porting (CRITICAL ABI LOCK)** | Stock vendor userspace blob `/vendor/lib64/egl/libGLES_mali.so` (and 32-bit `/vendor/lib/egl/libGLES_mali.so`) is compiled for **ARM Mali Bifrost DDK `r20p0`**. The kernel driver **must** be `mali-r20p0` (or an ABI-identical `r20p0` kbase port) with MediaTek `ged` (GPU Extension Device, `/proc/ged`) and `mt8168-gpufreq` hooks. Newer Mali DDKs (`r25p0`, `r32p0`) or mainline `panfrost` will reject the `r20p0` userspace blob on version handshake! |

---

### 3.4 Connectivity, Audio, Sensors, Camera & Power Management

| Subsystem | Hardware Block / Compatible | Exact Source Path in `kernel_casper_mt8168` | Kconfig Symbol(s) | Migration Classification | Technical Notes & Migration Blockers |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Wi-Fi / Bluetooth / GPS / FM (CONNSYS)** | MediaTek MT6631 + MT8168 CONNSYS (`consys@18002000`, `wifi@18000000`, `btif@1100c000`) | `drivers/misc/mediatek/connectivity/`, `via_s48_preload/` (`wmt_drv.ko`, `bt_drv.ko`, `wmt_chrdev_wifi.ko`, `wlan_drv_gen4m.ko`) | `CONFIG_MTK_COMBO=y`, `CONFIG_MTK_COMBO_CHIP_CONSYS_8168=y`, `CONFIG_VIA_S48_CONNSYS_PRELOAD=y`, `CONFIG_MTK_BTIF=y` | **Proprietary / Out-of-Tree Source Unavailable (CRITICAL BLOCKER)** | In `kernel_casper_mt8168`, `drivers/misc/mediatek/connectivity/` only contains the in-kernel adapter (`connadp.o`) and our `via_s48_preload` loader (`via_s48_connsys_preload.c`) which embeds prebuilt `4.14.141` `.ko` blobs (`wmt_drv.ko`, `bt_drv.ko`, `wmt_chrdev_wifi.ko`, `wlan_drv_gen4m.ko`)! The full out-of-tree `vendor/mediatek/kernel_modules/connectivity/` source is **not** in our repositories, and the `.ko` files have `vermagic: 4.14.141 SMP preempt mod_unload modversions aarch64`. Any kernel version change (`4.19+` or `5.x`) invalidates these `.ko` binaries completely! |
| **Audio (ASoC)** | MT8168 AFE PCM + MT6357 Codec (`sound`, `11220000.audio-controller`, `mt_soc_codec_63xx`) | `sound/soc/mediatek/mt8168/mt8168-afe-pcm.c`, `mt8168-p1.c`, `sound/soc/codecs/mt6357.c` | `CONFIG_SND_SOC_MT8168=y`, `CONFIG_SND_SOC_MT8168_P1=y`, `CONFIG_MTK_SND_SOC_CODEC="mt6357"` | **Available in MTK BSP (Requires ALSA Control Matching)** | Vendor `audio.primary.mt8168.so` expects exact ALSA kcontrol names (`Audio_Amp_R_Switch`, `Ext_Speaker_Amp_Switch`, `dl1_sram` controls) defined in `mt8168-p1.c` and `mt6357.c`. Mainline `mt8365-afe-pcm.c` uses different kcontrol names and breaks stock Android 10 audio HAL. |
| **Battery, Charger & Fuel Gauge** | MT6357 Linear/Switch Charger & GM3.0 Fuel Gauge (`mediatek,mt6357_charger`, `mediatek,bat_gm30`) | `drivers/power/supply/mediatek/charger/`, `drivers/power/supply/mediatek/battery/` | `CONFIG_MTK_CHARGER=y`, `CONFIG_MTK_GAUGE_VERSION=30`, `CONFIG_MTK_KERNEL_POWER_OFF_CHARGING=y` | **Requires Substantial Porting** | Vendor daemon `/vendor/bin/fuelgauged` communicates with kernel `bat_gm30` via custom netlink socket to compute battery SOC. Without `drivers/power/supply/mediatek/battery/`, battery percentage is inaccurate or stuck. |
| **Thermal Management** | MT8168 Thermal Controller + Power Throttle (`therm_ctrl@1100b000`) | `drivers/misc/mediatek/thermal/mt8168/` | `CONFIG_MTK_THERMAL=y` | **Requires Substantial Porting** | Interfaces with `/vendor/bin/thermal_manager` and `/proc/mtktz/*`. Includes our custom 2.1 GHz `NR_MAX_OPP_TBL=32` fix (commit `84ff56e2a`). |
| **Sensors (Accelerometer & ALS/PS)** | `mir3da` / `mc34xx` / `sc7a20` (GSensor on I2C2 `@0x4c`), `ltr559` (ALS/PS `@0x23`) | `drivers/misc/mediatek/sensors-1.0/accelerometer/`, `alsps/` | `CONFIG_MTK_SENSORS_1_0=y`, `CONFIG_MTK_MIR3DA=y`, `CONFIG_MTK_MC34XX=y`, `CONFIG_MTK_SC7A20=y`, `CONFIG_MTK_LTR559=y` | **Requires Substantial Porting** | Uses MediaTek's `sensors-1.0` (`hwmsen` / `m_acc_misc` / `/dev/m_acc_misc`) framework required by `android.hardware.sensors@2.0-service-mediatek`. |
| **Camera ISP & Sensors** | MT8168 ISP + `gc5025_mipi_raw` (`15000000.ispsys`, `kd_camera_hw1`) | `drivers/misc/mediatek/cameraisp/src/mt8168/`, `drivers/misc/mediatek/imgsensor/src/mt8168/` | `CONFIG_MTK_CAMERA_ISP=y`, `CONFIG_MTK_IMGSENSOR=y`, `CONFIG_CUSTOM_KERNEL_IMGSENSOR="gc5025_mipi_raw"` | **Requires Substantial Porting** | Tightly coupled to `camerahalserver` (`android.hardware.camera.provider@2.4-impl-mediatek.so`). Zero mainline support. |
| **Hardware Video Codecs (VCU / VDEC / VENC)** | MT8168 VCU (`16000000.vcu`, `14004000.mdp_rdma0`) | `drivers/media/platform/mtk-vcu/`, `drivers/media/platform/mtk-vcodec/`, `drivers/misc/mediatek/mdp/` | `CONFIG_VIDEO_MEDIATEK_VCU=y`, `CONFIG_VIDEO_MEDIATEK_VCODEC=y`, `CONFIG_MTK_JPEG=y` | **Requires Substantial Porting** | Vendor `/vendor/bin/vpud` and `libvcodec_oal.so` communicate with `/dev/vcu` and `/dev/mtk_mdp`. |

---

## 4. Critical Migration Blockers Summary

Our audit of the F716 hardware and software stack reveals **four hard migration blockers** that prevent a naive upgrade to Linux `4.19`, `5.4`, `5.10`, or `5.15`:

1. **Connectivity Module Binary Lock (`wmt_drv.ko`, `bt_drv.ko`, `wmt_chrdev_wifi.ko`, `wlan_drv_gen4m.ko`):**
   - Both `/vendor/lib/modules/*.ko` and the preloaded blobs inside `drivers/misc/mediatek/connectivity/via_s48_preload/blobs/*.ko` are prebuilt ELF relocatable kernel modules compiled against `Linux 4.14.141 SMP preempt mod_unload modversions aarch64`.
   - The C source code for `vendor/mediatek/kernel_modules/connectivity/` (specifically `CONSYS_8168` + `MT6631` `gen4m`) is **not** included in `kernel_casper_mt8168`. Without matching `gen4m` / `wmt_drv` source code for a newer kernel, upgrading the kernel version immediately breaks Wi-Fi, Bluetooth, GPS, and FM Radio.
2. **Display Stack Architecture Mismatch (`mtkfb` / `videox` vs. `DRM/KMS`):**
   - Our stock vendor HWC2 (`hwcomposer.mt8168.so`) and `libdpframework.so` require MediaTek's legacy `/dev/mtkfb` and `/dev/mtk_cmdq` interfaces (`drivers/misc/mediatek/video/mt8168/`).
   - MediaTek's newer kernels (`5.4+` and mainline `mt8365`) dropped `mtkfb`/`videox` in favor of `CONFIG_DRM_MEDIATEK`. Forward-porting `drivers/misc/mediatek/video/mt8168/` + `cmdq/v3` + `pseudo_m4u` across major VFS/MM/IOMMU changes into Linux 5.x is a massive undertaking with high risk of display blanking or tearing.
3. **GPU DDK Version Lock (`Mali Bifrost r20p0` + `GED` + `MTK ION`):**
   - `/vendor/lib64/egl/libGLES_mali.so` is locked to `mali bifrost r20p0` (`drivers/misc/mediatek/gpu/gpu_mali/mali_bifrost/mali-r20p0/`) and allocates graphics buffers via MediaTek's custom staging ION heaps (`/dev/ion`).
   - Linux `5.4`, `5.10`, and `5.15` substantially refactored `mm_struct`, `vm_operations_struct`, `dma_buf`, and removed staging `ION` in `5.11+`.
4. **Board-Specific F716 Calibrations:**
   - The Casper VIA S48 (`F716`) requires custom panel timings/gamma (`f716_fx_ek79029_boe_wxga_ips_8.h`, commit `d60cc9645`) and the 5,247-line Silead GSL1680 firmware (`f716_gsl1680_wxga.h`, commit `20189f473`). Any external BSP tree lacks these F716 board drivers out-of-the-box.

---

## 5. Risk Assessment & Recommended Next Steps

- **Risk of Migrating to Linux 5.x with Stock Android 10 Vendor:** **CRITICAL (Guaranteed Loss of Wi-Fi/BT and High Risk of Display/GPU Breakage).**
- **Recommended Strategy:** Preserve `Linux 4.14.141` (`kernel_casper_mt8168`) where 100% of F716 drivers, 2.1 GHz overclocking, and preloaded connectivity modules are verified working on Android 16, and apply targeted kernel feature backports + userspace Treble compatibility fixes for Android 17.
