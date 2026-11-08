# ISSUE-04: Legacy FUSE and eBPF Storage Virtualization

## Metadata
* **Issue ID:** `ISSUE-04`
* **Affected Component:** `vold` / FUSE Storage Daemon / eBPF Kernel Subsystem
* **Target Properties:** `persist.sys.fuse.bpf.override=false`, `persist.sys.phh.bpf=false`
* **Status:** **RESOLVED**

---

## 1. Problem Statement
Android 15 and 16 default to kernel-accelerated FUSE-BPF for virtualized storage (`/sdcard`, `/storage/emulated/0`). On legacy kernels lacking in-tree FUSE-BPF upstream modules, `vold` fails to attach BPF programs, resulting in crashes and unmountable shared storage.

---

## 2. Root Cause Analysis
The stock Linux 4.14.141 kernel supports general eBPF (`CONFIG_BPF_SYSCALL=y`), but does not contain the Android-specific FUSE-BPF hooks introduced in Linux 5.10 / 5.15.

---

## 3. Resolution
Rather than backporting hundreds of complex upstream FUSE-BPF patches into Linux 4.14, Android provides a userspace fallback to classic FUSE daemon operation.
We configured:
```properties
persist.sys.fuse.bpf.override=false
persist.sys.phh.bpf=false
```
Integrated permanently into `device_casper_via_s48/device.mk` under `PRODUCT_SYSTEM_PROPERTIES`.
