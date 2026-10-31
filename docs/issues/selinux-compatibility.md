# ISSUE-05: SELinux Vendor Policy Compatibility

## Metadata
* **Issue ID:** `ISSUE-05`
* **Affected Component:** `init` / SELinux Policy Engine / Vendor Domains
* **Target Setting:** `BOARD_KERNEL_CMDLINE += androidboot.selinux=permissive`
* **Status:** **MITIGATED (Development Permissive Mode)**

---

## 1. Problem Statement
Running Android 16 platform policy in Enforcing mode against Android 10 MediaTek vendor HALs causes numerous AVC denials. Essential services (such as `fuelgauged`, `wmt_loader`, and `camerahalserver`) are denied access to dev nodes, sysfs entries, and binder IPC endpoints.

---

## 2. Bring-Up vs. Production Approach
* **Bring-Up (Current):** `androidboot.selinux=permissive` is passed via kernel command line to prevent services from being killed while diagnosing bring-up issues.
* **Production Roadmap:**
  1. Capture AVC denial logs: `adb shell "dmesg | grep 'avc: denied'"` or extract `/data/misc/audit/audit.log`.
  2. Run `audit2allow -p` against target sepolicy.
  3. Author vendor-specific policy rules in `device/casper/via_s48/sepolicy/vendor/`.
  4. Test Enforcing mode once all denials are resolved.
