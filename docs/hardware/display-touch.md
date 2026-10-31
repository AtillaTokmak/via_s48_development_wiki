# Display Panel & Touchscreen Subsystems

## 1. Display Panel

* **Interface:** MIPI-DSI 4-Lane high-speed serial interface.
* **Resolution:** 1280 x 800 (WXGA) IPS LCD.
* **Aspect Ratio:** 16:10.
* **Refresh Rate:** 60.0 Hz.
* **Color Depth:** 24-bit True Color (RGB888).
* **Backlight Control:** Managed via MediaTek PWM (Pulse Width Modulation) driver on `mtk_disp`.
* **Hardware Composer:** MediaTek HWComposer 2.1 (`hwcomposer.mt8168.so`) utilizing hardware overlays to bypass GPU composition for UI surfaces.

---

## 2. Touchscreen Controller

* **Controller IC:** Silead GSL1680 Capacitive Touch Controller.
* **Interface:** I2C bus 0 (slave address `0x40`).
* **Kernel Driver:** `CONFIG_TOUCHSCREEN_MTK_GSLX68X=y` (`drivers/input/touchscreen/mediatek/gslX680/`).
* **Firmware Configuration:** The microcode initialization array is compiled directly into the kernel driver source code for the Casper VIA S48 (`F716`) board revision.
* **Capabilities:** Up to 10 simultaneous touch points with hardware noise rejection and wake-on-tap gesture support.
