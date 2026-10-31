# Dynamic Partitions & liblp Architecture

The Casper VIA S48 implements Android Dynamic Partitions (first introduced in Android 10).

---

## 1. Physical Geometry

* **Block Device:** `/dev/block/by-name/super` (`SYS33`)
* **Physical Start Address:** `0x12a80000`
* **Physical End Address:** `0x171c30000`
* **Size:** `0x15f1b0000` = `5,890,572,288` bytes
* **Logical Block Size:** 4096 bytes

---

## 2. Dynamic Partition Group

* **Group Name:** `main`
* **Group Max Size:** `5,886,377,984` bytes (Total Super Size minus 4 MB metadata headers).
* **Logical Partitions Included:**
  * `system`: OS framework, platform binaries, shared runtime.
  * `vendor`: MediaTek BSP HALs, proprietary drivers, firmware.
  * `product`: Product-level apps and customizations.

---

## 3. Fastbootd Flashing Protocol

Because dynamic partitions do not have dedicated physical eMMC flash boundaries, they cannot be flashed from the standard Little Kernel bootloader fastboot mode. Flashing requires entering **Fastbootd**:

```bash
# Transition from bootloader to Fastbootd userspace recovery
fastboot reboot fastboot

# Flash dynamic logical partitions
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot flash product product.img
```
