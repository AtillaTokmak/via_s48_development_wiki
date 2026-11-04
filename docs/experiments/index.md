# ROM Experiments Journal: Casper VIA S48

This journal provides an exhaustive, chronological engineering record of every custom ROM, GSI, and kernel bring-up experiment performed on the Casper VIA S48 tablet.

---

## Experiment Registry

| Experiment ID | Date | Target OS | ROM / Kernel Tested | Final Result | Key Finding / Blocker |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [**`EXP-001`**](android14/gsi-bringup.md) | 2026-10-25 | Android 14 (API 34) | LineageOS 21 GSI (Stock Kernel) | **SUCCESS** | Required `compatibility_matrix.4.xml` and DFE fstab. |
| [**`EXP-002`**](android15/crdroid-bootloop.md) | 2026-10-28 | Android 15 (API 35) | crDroid 11 GSI (Stock Kernel) | **FAILED (Bootloop)** | FUSE-BPF crash during vold initialization. |
| [**`EXP-003`**](android16/axionos-bringup.md) | 2026-10-31 | Android 16 (API 36) | AxionOS 2.9 (Patched GSI) | **SUCCESS** | Identified MP2 audio UBSAN crash in `system_server`. |
| [**`EXP-004`**](android16/stock-kernel-test.md) | 2026-10-31 | Android 16 (API 36) | AxionOS 2.9 (Stock 2.0 GHz Kernel) | **SUCCESS** | Booted with 2.0 GHz CPU ceiling. |
| [**`EXP-005`**](android16/oc-2.1ghz-test.md) | 2026-10-31 | Android 16 (API 36) | AxionOS 2.9 (Overclocked 2.1 GHz Kernel) | **SUCCESS (Verified Working Baseline)** | All 4 cores dynamically scaling to 2,100,000 kHz. Daily driver standard. |
| [**`EXP-006`**](android17/blazeaosp-bringup.md) | 2026-11-01 | Android 17 (API 37) | BlazeAOSP Cinnamon Bun (Candidate V3) | **CONCLUDED (Incompatible)** | Reboots at early boot; Linux 4.14 kernel missing mandatory Android 17 userspace/BPF primitives. |
| [**`EXP-007`**](android16/evox-usb-gadget-kernel-panic.md) | 2026-11-04 | Android 16 (API 36) | Evolution X 11.11 GSI | **SUCCESS** | Isolated MTK 4.14 kernel panic in `mtp_free_inst()` triggered by `twrp.sh` configfs removal. |
