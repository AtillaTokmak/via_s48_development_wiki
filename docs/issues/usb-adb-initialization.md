# ISSUE-06: USB Controller UDC Gadget Binding

## Metadata
* **Issue ID:** `ISSUE-06`
* **Affected Component:** `adbd` / USB ConfigFS Gadget / MediaTek USB OTG Controller
* **Target Setting:** `sys.usb.controller=11201000.usb`
* **Status:** **RESOLVED**

---

## 1. Problem Statement
When early boot failures occur, ADB debugging does not enumerate on the host computer. `adbd` waits indefinitely for a USB gadget controller to be attached to `/config/usb_gadget/g1/UDC`.

---

## 2. Root Cause Analysis
The MediaTek MT8168 USB OTG peripheral controller is mapped in the device tree as `11201000.usb`. Android 16's dynamic gadget daemon requires an explicit system property indicating which hardware controller to bind to.

---

## 3. Resolution
Added to `init.mt8168.usb.rc` and `device.mk`:
```properties
sys.usb.controller=11201000.usb
```
This forces immediate binding of the USB controller as soon as `adbd` launches, providing full ADB debugging during early boot stages.
