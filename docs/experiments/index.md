# ROM Experiments Journal: Casper VIA S48

This journal provides an engineering record of custom ROM, kernel, and hardware integration experiments performed on the Casper VIA S48 tablet.

---

## Active Hardware & Kernel Verification Registry

| Experiment ID | Date | Target OS | ROM / Kernel Tested | Final Result | Key Finding / Feature |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [**`EXP-004`**](android16/stock-kernel-test.md) | 2026-10-31 | Android 16 (API 36) | Stock 2.0 GHz Kernel Baseline | **SUCCESS** | Booted with 2.0 GHz stock CPU ceiling. |
| [**`EXP-005`**](android16/oc-2.1ghz-test.md) | 2026-10-31 | Android 16 (API 36) | Overclocked 2.1 GHz Kernel Baseline | **SUCCESS** | All 4 cores dynamically scaling to 2,100,000 kHz. Daily driver standard. |
| [**`EXP-008`**](android16/hardware-fixes-test.md) | 2026-11-08 | Android 16 (API 36) | 2.1 GHz Kernel + HW Fixes Final | **SUCCESS** | Front camera, LED191 flash/torch, and capacitive touchkeys verified on physical hardware. |

---

## Archived GSI Bring-Up Experiments

Generic System Image (GSI) experiments from earlier bring-up phases have been archived in [`docs/archived/`](../archived/):
* `EXP-001`: Android 14 GSI Initial Bring-Up ([Archived](../archived/exp-001-android14-gsi.md))
* `EXP-002`: Android 15 crDroid Bootloop Investigation ([Archived](../archived/exp-002-android15-crdroid-gsi.md))
* `EXP-003`: AxionOS 2.9 GSI Initial Bring-Up ([Archived](../archived/exp-003-android16-axionos-gsi.md))
* `EXP-006`: Android 17 BlazeAOSP Evaluation ([Archived](../archived/exp-006-android17-blazeaosp-gsi.md))
* `EXP-007`: Android 16 Evolution X Kernel Panic Remediation ([Archived](../archived/exp-007-android16-evox-panic.md))
* `EXP-PixelOS`: PixelOS GSI Adaptation Audit ([Archived](../archived/pixelos-android16.md))

