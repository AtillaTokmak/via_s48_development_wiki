# Kernel Cross-Compilation: Linux 4.14

Repository: `https://github.com/AtillaTokmak/kernel_casper_mt8168` (Branch: `android16-axionos-compat`)

---

## 1. Toolchain Setup

The MediaTek MT8168 BSP compiles with GNU GCC 9.3 aarch64 cross-compiler or Android Clang:
```bash
sudo apt install -y gcc-aarch64-linux-gnu gcc-arm-linux-gnueabi libssl-dev bc
```

---

## 2. Compilation Procedure
```bash
cd kernel_casper_mt8168
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
export CROSS_COMPILE_ARM32=arm-linux-gnueabi-

# 1. Generate configuration
make tb8168p1_64_bsp_f716_defconfig

# 2. Compile kernel Image.gz and Device Tree Blobs
make -j$(nproc) Image.gz-dtb
```
The output kernel binary will be generated at `arch/arm64/boot/Image.gz-dtb`.
