# Pre-Flash ROM Validation Suite

The automated validation tool [`scripts/validate_rom_images.sh`](https://github.com/AtillaTokmak/via_s48_development_wiki) runs 6 safety verification gates before flashing images:

* **Gate 1: Boot Image Validation:** Checks Android Boot v2 header, 10.2 MB kernel, 69 KB DTB, and 791 KB DFE ramdisk.
* **Gate 2: Userdata FBE Check:** Ensures `fileencryption` and `checkpoint` are omitted from fstabs.
* **Gate 3: MP2 Media Codec Check:** Verifies that `OMX.MTK.AUDIO.DECODER.MP2` is completely absent from audio codec XMLs.
* **Gate 4: VINTF FCM Matrix Check:** Confirms presence of Target Level 4 matrix.
* **Gate 5: Kernel Overclock Check:** Verifies `opp-2100000000` in DTB and `2100000` ceiling in CPUFreq driver.
* **Gate 6: Device Tree & Vendor Check:** Validates BoardConfig partition geometry (`5890572288` bytes), makefile syntax, and 570 proprietary blobs.

**Validation Status:** **6/6 GATES PASSED.**
