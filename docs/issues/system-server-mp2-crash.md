# ISSUE-01: System Server MP2 Media Codec UBSAN Crash

## Metadata
* **Issue ID:** `ISSUE-01`
* **Affected Component:** `system_server` / `libmedia_codeclist_capabilities.so` / `MediaCodecList`
* **Target File:** `/vendor/etc/media_codecs_mediatek_audio.xml`
* **Status:** **RESOLVED**

---

## 1. Problem Statement
During initial Android 16 startup, the device hangs indefinitely at the boot animation. Logcat inspection reveals that `system_server` crashes with SIGABRT repeatedly every few seconds, preventing the system from ever reaching `sys.boot_completed=1`.

---

## 2. Crash Stack Trace
```
10-31 15:45:10.123  1245  1245 E AudioCapabilities: Unsupported mediaType audio/mpeg-L2
10-31 15:45:10.124  1245  1245 F libc    : Fatal signal 6 (SIGABRT), code -1 (SI_QUEUE) in tid 1245 (system_server)
10-31 15:45:10.128  1300  1300 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
10-31 15:45:10.129  1300  1300 F DEBUG   : Build fingerprint: 'AxionOS/treble_arm64_bvN/treble_arm64_bvN:16/BP4A.251205.006/eng.20261017:userdebug/test-keys'
10-31 15:45:10.129  1300  1300 F DEBUG   : Revision: '0'
10-31 15:45:10.129  1300  1300 F DEBUG   : ABI: 'arm64'
10-31 15:45:10.129  1300  1300 F DEBUG   : Timestamp: 2026-10-31 15:45:10+0300
10-31 15:45:10.130  1300  1300 F DEBUG   : pid: 1245, tid: 1245, name: system_server  >>> system_server <<<
10-31 15:45:10.130  1300  1300 F DEBUG   : uid: 1000
10-31 15:45:10.130  1300  1300 F DEBUG   : signal 6 (SIGABRT), code -1 (SI_QUEUE), fault addr --------
10-31 15:45:10.130  1300  1300 F DEBUG   : Abort message: 'ubsan: sub-overflow'
10-31 15:45:10.130  1300  1300 F DEBUG   :     x0  0000000000000000  x1  00000000000004dd  x2  0000000000000006  x3  0000007fe8124990
10-31 15:45:10.131  1300  1300 F DEBUG   : backtrace:
10-31 15:45:10.131  1300  1300 F DEBUG   :       #00 pc 000000000004e280  /apex/com.android.runtime/lib64/bionic/libc.so (abort+168)
10-31 15:45:10.131  1300  1300 F DEBUG   :       #01 pc 000000000005a110  /system/lib64/libmedia_codeclist_capabilities.so (__ubsan_handle_sub_overflow_minimal_abort+24)
10-31 15:45:10.131  1300  1300 F DEBUG   :       #02 pc 000000000005b42c  /system/lib64/libmedia_codeclist_capabilities.so (android::AudioCapabilities::getDefaultFormat(android::sp<android::AMessage> const&) const+512)
10-31 15:45:10.131  1300  1300 F DEBUG   :       #03 pc 000000000028a1f4  /system/framework/arm64/boot-framework.oat (android.media.MediaCodecList.findDecoderForFormat+212)
10-31 15:45:10.132  1300  1300 F DEBUG   :       #04 pc 0000000000492ab8  /system/framework/arm64/services.odex (com.android.server.StorageManagerService.configureTranscoding+88)
10-31 15:45:10.132  1300  1300 F DEBUG   :       #05 pc 0000000000490f20  /system/framework/arm64/services.odex (com.android.server.StorageManagerService.handleSystemReady+340)
```

---

## 3. Root Cause Analysis
In Android 16, Google compiled `libmedia_codeclist_capabilities.so` with Clang Undefined Behavior Sanitizer (UBSAN) unsigned/signed arithmetic overflow checks enabled with `-fsanitize-minimal-runtime`.
When `StorageManagerService` boots, it calls `configureTranscoding()`, which queries `MediaCodecList` for supported formats.
The legacy MediaTek XML contains:
```xml
<MediaCodec name="OMX.MTK.AUDIO.DECODER.MP2" type="audio/mpeg-L2">
    <Quirk name="needs-discrete-audio-sample-sizes" />
</MediaCodec>
```
`AudioCapabilities::getDefaultFormat()` calculates bitrate upper and lower bounds. Because the legacy XML omitted explicit bitrate ranges for the deprecated MP2 format, an internal subtraction evaluated to a negative value or underflowed, triggering `__ubsan_handle_sub_overflow_minimal_abort`.

---

## 4. Resolution
1. **Runtime Fix:** Added a bind-mount hook in `rw-system.sh` to remove `OMX.MTK.AUDIO.DECODER.MP2`.
2. **Build-Time Fix:** Added a sanitized version of `media_codecs_mediatek_audio.xml` into `device/casper/via_s48/configs/` and declared it in `device.mk`:
   ```makefile
   PRODUCT_COPY_FILES += \
       $(DEVICE_PATH)/configs/media_codecs_mediatek_audio.xml:$(TARGET_COPY_OUT_VENDOR)/etc/media_codecs_mediatek_audio.xml
   ```
This permanently prevents the abort across both GSI and source builds.
