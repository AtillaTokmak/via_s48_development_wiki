# Table of contents

* [Introduction](README.md)

## Hardware
* [Hardware Overview](docs/hardware/overview.md)
* [MediaTek MT8168 SoC](docs/hardware/mt8168.md)
* [Partition Layout](docs/hardware/partition-layout.md)
* [Kernel Architecture](docs/hardware/kernel.md)
* [Display & Touchscreen](docs/hardware/display-touch.md)

## Firmware
* [Stock Android 10 Baseline](docs/firmware/stock-android10.md)
* [Backup and Restore](docs/firmware/backup-and-restore.md)
* [Android Verified Boot (AVB)](docs/firmware/avb.md)
* [Dynamic Partitions Geometry](docs/firmware/dynamic-partitions.md)

## Experiments Journal
* [Experiment Index](docs/experiments/index.md)
* [EXP-004: Stock Kernel Compatibility Test](docs/experiments/android16/stock-kernel-test.md)
* [EXP-005: 2.1 GHz Overclocked Kernel Verification](docs/experiments/android16/oc-2.1ghz-test.md)
* [EXP-008: Hardware Fixes (Camera, Flash, Keys)](docs/experiments/android16/hardware-fixes-test.md)

## Issues & Root Causes
* [Issue Tracker Overview](docs/issues/index.md)
* [ISSUE-01: System Server MP2 Codec UBSAN Crash](docs/issues/system-server-mp2-crash.md)
* [ISSUE-02: File-Based Encryption (FBE) Deadlock](docs/issues/fbe-encryption.md)
* [ISSUE-03: VINTF FCM Level 4 Integration](docs/issues/vintf-fcm4.md)
* [ISSUE-05: SELinux Vendor Policy Compatibility](docs/issues/selinux-compatibility.md)
* [ISSUE-07: Android 16 VNDK Deprecation & Vendor Dependencies](docs/issues/vndk-deprecation.md)

## Archived Experiments & Diagnostics
* [EXP-001: Android 14 GSI Bring-Up](docs/archived/exp-001-android14-gsi.md)
* [EXP-002: Android 15 crDroid Bootloop](docs/archived/exp-002-android15-crdroid-gsi.md)
* [EXP-003: Android 16 AxionOS GSI Bring-Up](docs/archived/exp-003-android16-axionos-gsi.md)
* [EXP-006: Android 17 BlazeAOSP Evaluation](docs/archived/exp-006-android17-blazeaosp-gsi.md)
* [EXP-007: Android 16 Evolution X Kernel Panic Remediation](docs/archived/exp-007-android16-evox-panic.md)
* [EXP-PixelOS: PixelOS Android 16 GSI Audit](docs/archived/pixelos-android16.md)
* [ISSUE-04: Legacy FUSE and eBPF Storage](docs/archived/issue-04-fuse-bpf.md)
* [ISSUE-06: USB UDC Gadget Binding](docs/archived/issue-06-usb-adb-init.md)

## Patches & Modifications
* [Patch Overview](docs/patches/index.md)
* [Full Patch Inventory](docs/patches/patch-inventory.md)

## Development & Building
* [Build Environment Setup](docs/development/environment.md)
* [Device Tree Architecture](docs/development/device-tree.md)
* [Vendor Tree & Extraction](docs/development/vendor-tree.md)
* [Kernel Cross-Compilation](docs/development/kernel-build.md)
* [ROM Compilation](docs/development/rom-build.md)
* [Boot Image Packaging](docs/development/boot-image-packaging.md)
* [Source Tree Integration](docs/development/source-integration.md)
* [Android 17 Kernel Compatibility Audit](docs/development/android17_kernel_audit.md)
* [Android 17 Minimum Kernel Requirements](docs/development/android17-minimum-kernel-requirements.md)
* [Kernel Upgrade Feasibility](docs/development/kernel-upgrade-feasibility.md)
* [MT8168 Hardware Driver Migration Audit](docs/development/mt8168-driver-migration.md)
* [Kernel Version Comparison Matrix](docs/development/kernel-version-comparison.md)

## Testing & Validation
* [Hardware Subsystem Status](docs/testing/hardware-status.md)
* [Performance & CPU Frequency Scaling](docs/testing/performance.md)
* [Pre-Flash Validation Suite](docs/testing/boot-validation.md)

## Troubleshooting
* [Diagnosing Boot Animation Hangs](docs/troubleshooting/boot-animation-hang.md)
* [Unbricking & SP Flash Tool Recovery](docs/troubleshooting/recovery.md)
* [Early ADB Debugging over USB](docs/troubleshooting/adb-debugging.md)

## References
* [Official Project Repositories](docs/references/repositories.md)
* [Upstream Sources & Toolchains](docs/references/upstream-sources.md)
* [Technical Glossary](docs/references/glossary.md)

## Templates
* [Experiment Template](templates/experiment.md)
* [Issue Template](templates/issue.md)
* [Patch Template](templates/patch.md)
* [Test Report Template](templates/test-report.md)
