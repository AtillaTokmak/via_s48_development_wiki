# Hardware Subsystem Compatibility Status

Status verified on Casper VIA S48 running AxionOS Android 16 (API 36):

| Subsystem | Driver / HAL | Operational Status | Notes |
| :--- | :--- | :--- | :--- |
| **CPU Complex** | Linux CPUFreq | **VERIFIED WORKING** | 4 cores scaling 500 MHz to 2100 MHz |
| **GPU / 3D** | Mali-G52 MP1 | **VERIFIED WORKING** | Hardware acceleration active, 60 FPS UI |
| **Touchscreen** | Silead GSL1680 | **VERIFIED WORKING** | 10-point multitouch responsive |
| **Display** | MediaTek DSI | **VERIFIED WORKING** | 1280x800 resolution, backlight scaling |
| **Audio** | MT8168 Audio HAL 5.0 | **VERIFIED WORKING** | MP2 decoder removed to prevent abort |
| **Storage** | eMMC 5.1 / EXT4 | **VERIFIED WORKING** | 24 GB usable userdata mounted via DFE |
| **Wi-Fi** | MT6631 Dual-Band | **VERIFIED WORKING** | 2.4 GHz and 5 GHz AP association |
| **Bluetooth** | MT6631 BT 5.0 | **VERIFIED WORKING** | BLE and A2DP operational |
| **Power / Battery** | MT6357 PMIC | **VERIFIED WORKING** | Fuel gauge reading capacity and charging current |
| **USB** | MTK USB OTG | **VERIFIED WORKING** | Bound to controller `11201000.usb` |
| **Sensors** | MTK Sensor HAL 2.0 | **FUNCTIONAL** | Accelerometer auto-rotation working |
| **Camera** | MTK Camera HAL3 | **PARTIAL** | Preview functional |
| **Security / TEE** | TrustKernel MTEE | **DEGRADED (DFE)** | Keymaster 4.0 bypassed |
| **SELinux** | Platform Policy | **PERMISSIVE** | Permissive mode during bring-up |
