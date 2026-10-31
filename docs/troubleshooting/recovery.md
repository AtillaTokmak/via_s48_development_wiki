# Brick Recovery & Fastboot Rescue

## 1. Entering Bootloader Mode
Hold `Power + Volume Up` while plugging in the USB cable until the MediaTek boot select menu appears. Select `[Fastboot Mode]`.

---

## 2. Emergency Partition Flash
```bash
# Flash known working boot image
fastboot flash boot boot_oc_2.1ghz_android16.img
fastboot --disable-verity --disable-verification flash vbmeta vbmeta_disabled.img

# Enter Fastbootd
fastboot reboot fastboot

# Reflash system if corrupted
fastboot flash system Axion-2.9-patched.img

# Clear userdata
fastboot erase userdata
fastboot reboot
```
