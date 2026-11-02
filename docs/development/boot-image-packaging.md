# Android Boot Image Packaging (v2)

To package a standalone Android Boot Image v2 containing the 2.1 GHz kernel, DTB, and DFE ramdisk:

```bash
mkbootimg \
    --kernel kernel_binary \
    --ramdisk ramdisk_dfe.cpio.gz \
    --dtb mt8168.dtb \
    --cmdline "bootopt=64S3,32N2,64N2 buildvariant=user androidboot.selinux=permissive" \
    --base 0x40000000 \
    --kernel_offset 0x00080000 \
    --ramdisk_offset 0x04000000 \
    --tags_offset 0x00000100 \
    --dtb_offset 0x00000100 \
    --pagesize 2048 \
    --header_version 2 \
    --os_version 10.0.0 \
    --os_patch_level 2022-06 \
    -o boot_oc_2.1ghz_android16.img
```

Automated by [`scripts/pack_boot_android16.sh`](https://github.com/AtillaTokmak/kernel_casper_mt8168/blob/android-16/scripts/pack_boot_android16.sh) in the kernel repository.
