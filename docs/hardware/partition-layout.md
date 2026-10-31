# Casper VIA S48 Partition Layout & Flash Geometry

The internal flash storage is a 32 GB eMMC 5.1 device organized according to the MediaTek Android scatter format (`MT8168_Android_scatter.txt`).

---

## 1. Physical Partition Table

| Index | Partition Name | File Name | Start Address | Physical Size (Hex) | Size (Bytes) | Description |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PRELOADER** | `preloader` | `preloader_tb8168p1_64_bsp.bin` | Boot1 | `0x00040000` | 256 KB | First-stage hardware loader |
| **SYS0** | `pgpt` | `GPT.img` | `0x00000000` | `0x00080000` | 512 KB | Primary GUID Partition Table |
| **SYS17** | `lk` | `lk-verified.img` | `0x06000000` | `0x00100000` | 1 MB | Little Kernel (2nd stage bootloader) |
| **SYS19** | `boot` | `boot.img` | `0x07400000` | `0x02000000` | 32 MB | Linux Kernel, DTB & Ramdisk |
| **SYS20** | `recovery` | `recovery.img` | `0x09400000` | `0x02000000` | 32 MB | Recovery Ramdisk & Kernel |
| **SYS25** | `dtbo` | `dtbo-verified.img` | `0x0c400000` | `0x00800000` | 8 MB | Device Tree Blob Overlay |
| **SYS26** | `tee` | `tee-verified.img` | `0x0cc00000` | `0x00500000` | 5 MB | TrustZone / TrustKernel TEE OS |
| **SYS28** | `vbmeta` | `vbmeta.img` | `0x0d380000` | `0x00100000` | 1 MB | AVB 2.0 Root Signature |
| **SYS29** | `vbmeta_system` | `vbmeta_system.img` | `0x0d480000` | `0x00100000` | 1 MB | System Partition AVB Hash Tree |
| **SYS30** | `vbmeta_vendor` | `vbmeta_vendor.img` | `0x0d580000` | `0x00100000` | 1 MB | Vendor Partition AVB Hash Tree |
| **SYS33** | `super` | `super.img` | `0x12a80000` | `0x15f1b0000` | **5,890,572,288** | Dynamic Partition Container |
| **SYS34** | `userdata` | `userdata.img` | `0x171c30000` | Variable | ~24.3 GB | User Data & Application Storage |

---

## 2. Dynamic Partition Container (`super.img`)

The `super` partition contains logical partitions managed by Linux `device-mapper` through Android `liblp`:

* **Total Super Size:** `5,890,572,288` bytes (5617 MB / 0x15f1b0000)
* **Metadata Reserved Space:** 4,194,304 bytes (4 MB)
* **Dynamic Partition Group Name:** `main`
* **Maximum Group Size:** `5,886,377,984` bytes

### Logical Partition Allocation (Stock C1.03 Baseline)
```
Block device: super (Size: 5,890,572,288 bytes)
├── vendor   : Extents: 2048 .. 621280    (619,232 sectors =   317,046,784 bytes)
├── product  : Extents: 622592 .. 2866864  (2,244,272 sectors = 1,149,067,264 bytes)
└── system   : Extents: 2867200 .. 5350056 (2,482,856 sectors = 1,271,222,272 bytes)
```

In Android 16, custom dynamic partition images (such as `Axion-2.9-patched.img`) replace the stock `system` logical partition while preserving `vendor` and `product`.
