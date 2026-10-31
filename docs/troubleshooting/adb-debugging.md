# Early ADB Debugging over USB

To ensure ADB enumerates during early boot stages:

1. **Verify Vendor Init Script:** Ensure `init.mt8168.usb.rc` sets `sys.usb.controller=11201000.usb`.
2. **Kernel Command Line:** Verify `buildvariant=userdebug` is present in `BOARD_KERNEL_CMDLINE`.
3. **Property Trigger:** Set `persist.sys.usb.config=adb` in `device.mk`.
4. **Host Rules:** Verify host `/etc/udev/rules.d/51-android.rules` contains MediaTek VID:
   ```
   SUBSYSTEM=="usb", ATTR{idVendor}=="0e8d", MODE="0666", GROUP="plugdev"
   ```
