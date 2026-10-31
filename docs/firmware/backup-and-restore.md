# Firmware Backup & Restore Procedures

## 1. Complete Factory Restoration via SP Flash Tool

If the tablet enters a hard brick or bootloader loop:

1. Launch **SP Flash Tool v5.1924** or newer on Linux / Windows.
2. Load the scatter file:
   `Casper_VIA_S48_0_20220606_C1.03_[WwW-(TimurGsmViP)-Com]/MT8168_Android_scatter.txt`
3. Select **Download Only** mode (do NOT select Format All + Download to avoid losing calibration/NVRAM).
4. Power off tablet completely.
5. Hold `Volume Down` and connect the USB cable.
6. Wait for the green checkmark icon.

---

## 2. Partition Restoration via Fastboot

To restore stock boot and system partitions via command line:

```bash
# 1. Flash stock boot and recovery
fastboot flash boot stock_boot.img
fastboot flash recovery stock_recovery.img

# 2. Flash stock vbmeta
fastboot flash vbmeta stock_vbmeta.img

# 3. Enter Fastbootd
fastboot reboot fastboot

# 4. Flash stock super image or individual partitions
fastboot flash super stock_super.img

# 5. Format userdata
fastboot erase userdata
fastboot reboot
```
