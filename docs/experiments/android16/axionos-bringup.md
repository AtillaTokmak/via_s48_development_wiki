# EXP-003: AxionOS Android 16 Initial Bring-Up & MP2 Crash Resolution

## Metadata
* **Experiment ID:** `EXP-003`
* **Date:** `2026-10-31`
* **Target Android Version:** Android 16 (API Level 36)
* **ROM Source:** `Axion-2.9-GSI_treble_arm64-ab-VANILLA-20261017.img` (Build: `BP4A.251205.006`)
* **Kernel Version:** Linux 4.14.141
* **Final Result:** **SUCCESS**

---

## 1. Initial Failure: The Boot Animation Hang
Upon flashing raw AxionOS 2.9, the tablet displayed the boot animation indefinitely.
ADB logs revealed repeated crashes of `system_server`:

```
AudioCapabilities: Unsupported mediaType audio/mpeg-L2
Fatal signal 6 (SIGABRT), code -1 (SI_QUEUE) in tid 1245 (system_server)
Abort message: 'ubsan: sub-overflow'
    #00 pc 000000000004e280  /apex/com.android.runtime/lib64/bionic/libc.so (abort+168)
    #01 pc 000000000005a110  /system/lib64/libmedia_codeclist_capabilities.so (__ubsan_handle_sub_overflow_minimal_abort+24)
    #02 pc 000000000005b42c  /system/lib64/libmedia_codeclist_capabilities.so (android::AudioCapabilities::getDefaultFormat(...)
    #03 pc 000000000028a1f4  /system/framework/arm64/boot-framework.oat (android.media.MediaCodecList.findDecoderForFormat+212)
    #04 pc 0000000000492ab8  /system/framework/arm64/services.odex (com.android.server.StorageManagerService.configureTranscoding+88)
```

---

## 2. Technical Investigation
1. `StorageManagerService.handleSystemReady()` initializes transcoding heuristics.
2. It queries `MediaCodecList` to evaluate whether HEVC / audio transcoding is needed.
3. MediaCodecList parses `/vendor/etc/media_codecs_mediatek_audio.xml`.
4. It encounters `<MediaCodec name="OMX.MTK.AUDIO.DECODER.MP2" type="audio/mpeg-L2">`.
5. In Android 16, UBSAN arithmetic checks are strictly enforced. Calculating bitrate limits on MP2 with missing vendor range data resulted in an integer underflow abort, instantly killing the Android runtime.

---

## 3. Operations & Resolution
1. Modified `rw-system.sh` to execute on post-fs:
   ```bash
   sed -i '/OMX.MTK.AUDIO.DECODER.MP2/d' /mnt/vendor_codecs/media_codecs_mediatek_audio.xml
   sed -i '/audio\/mpeg-L2/d' /mnt/vendor_codecs/media_codecs_mediatek_audio.xml
   mount -o bind /mnt/vendor_codecs/media_codecs_mediatek_audio.xml /vendor/etc/media_codecs_mediatek_audio.xml
   ```
2. Appended FUSE-BPF bypass properties:
   ```properties
   persist.sys.fuse.bpf.override=false
   persist.sys.phh.bpf=false
   ```
3. Rebooted system:
   ```bash
   fastboot reboot
   ```

---

## 4. Final Result
The tablet completed boot animation and launched into AxionOS 2.9 Welcome Activity (`com.android.provision.DefaultActivity`).
Visual confirmation captured in `axion_android16_welcome.png`.
