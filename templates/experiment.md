# Experiment Report Template

## Metadata
* **Experiment ID:** `EXP-XXX`
* **Date:** `YYYY-MM-DD`
* **Author:** `Atilla Tokmak`
* **Android Target Version:** `e.g. Android 16 (API 36)`
* **ROM Source:** `e.g. AxionOS 2.9 (Build BP4A.251205.006)`
* **Kernel Version:** `e.g. Linux 4.14.141 (#10 SMP PREEMPT 2026)`
* **Kernel Git Commit:** `e.g. 100f94623`
* **Boot Image Version:** `e.g. boot_oc_2.1ghz_android16.img`
* **Vendor Baseline:** `Casper_VIA_S48_0_20220606_C1.03`

---

## 1. Objective & Hypothesis
* **Objective:** Clearly state what this experiment intends to prove, test, or implement.
* **Hypothesis:** State the technical theory being verified.

---

## 2. Prerequisites & Device State
* Initial state of partitions (`boot`, `system`, `vendor`, `userdata`, `vbmeta`).
* Connected peripherals and USB status.

---

## 3. Exact Operations & Commands Executed
Document every command sequentially:
```bash
# Example
fastboot flash boot test_boot.img
fastboot reboot
```

---

## 4. File Modifications & Checksums
| File Name | Original SHA-256 | Modified SHA-256 | Description of Change |
| :--- | :--- | :--- | :--- |
| `example.xml` | `hash1...` | `hash2...` | Removed buggy entry |

---

## 5. Observed Behavior & Telemetry
* Behavior on display.
* Serial / ADB logs observed.
* Failure stage (e.g., Bootloader, Kernel, Init, SurfaceFlinger, Boot Completed).

---

## 6. Diagnostic Log Excerpt
```
[Log snippet showing crash or success]
```

---

## 7. Result & Lessons Learned
* **Final Result:** `PASS` / `FAIL` / `INCONCLUSIVE`
* **Root Cause Identified:** Explain why it behaved this way.
* **Recovery Action Taken:** How device was restored.
* **Links to Commits & Issues:** `[ISSUE-XX](link)`
