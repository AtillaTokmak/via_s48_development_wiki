# Technical Glossary

* **AVB (Android Verified Boot):** Google's cryptographic boot verification system enforcing partition integrity via hash trees.
* **DFE (Disabled File-Based Encryption):** An fstab configuration that omits `fileencryption` and `checkpoint` flags, allowing `/data` to mount as plain unencrypted ext4/f2fs.
* **DTB (Device Tree Blob):** Compiled Open Firmware hardware description passed to the Linux kernel during boot.
* **FCM (Framework Compatibility Matrix):** An XML specification defining required and supported HAL versions between Android system and vendor partitions.
* **FUSE-BPF:** An acceleration mechanism routing userspace FUSE filesystem operations directly through kernel eBPF bytecode.
* **GSI (Generic System Image):** A standardized Android system partition image built to run on any Treble-compliant device.
* **Fastbootd:** Userspace fastboot running inside recovery that supports dynamic partition flashing (`liblp`).
* **OPP (Operating Performance Point):** Kernel device tree table specifying frequency and voltage pairs for CPU scaling.
* **UBSAN (Undefined Behavior Sanitizer):** Compiler instrumentation checking for integer overflows, memory alignment issues, and undefined operations at runtime.
