# Diagnosing Boot Animation Hangs

When an Android port hangs at the boot animation, the kernel, display hardware, and SurfaceFlinger are already operating. The hang occurs because `system_server` or core Android services failed to initialize.

---

## Diagnostic Steps

1. Connect USB cable to host.
2. Check if ADB is alive:
   ```bash
   adb devices
   ```
3. Capture live logcat:
   ```bash
   adb logcat -v time > boot_crash.log
   ```
4. Search for fatal exceptions:
   ```bash
   grep -E "FATAL|SIGABRT|Abort message|AndroidRuntime: FATAL" boot_crash.log
   ```
5. If `AudioCapabilities: Unsupported mediaType audio/mpeg-L2` appears, verify [ISSUE-01](../issues/system-server-mp2-crash.md).
6. If `vold: FUSE-BPF` appears, verify [ISSUE-04](../issues/fuse-bpf.md).
