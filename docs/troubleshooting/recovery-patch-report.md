# Casper VIA S48 recovery yaması: nasıl yapıldı?

## Sonuç ve kapsam

Gönderilen `recovery(1).img` açıldı, içindeki stok recovery ve ADB bileşenleri incelendi. ZIP paketinin kriptografik imza kontrolünü atlayan ve recovery içinde root USB ADB erişimi hedefleyen bir imaj üretildi.

Çıktı: `recovery-s48-adbroot-nosig.img`

Bu işlem TWRP derlemesi veya Android sistemine Magisk kurulumu değildir. Değişiklikler recovery imajının içindedir. İmaj cihazda çalıştırılmadı. Dolayısıyla açılış, gerçek USB bağlantısı, root shell ve ZIP kurulumu henüz cihaz üzerinde doğrulanmadı.

## 1. Orijinal imajın incelenmesi

İmajın Android boot image biçiminde olduğu görüldü. Başlık ve bölüm boyutları Python ile okundu.

| Alan | Değer |
| --- | --- |
| Cihaz | Casper VIA S48 / MT8168 |
| Recovery tabanı | Android 10, API 29 |
| Boot image başlık sürümü | 2 |
| Sayfa boyutu | 2048 bayt |
| Toplam imaj boyutu | 33.554.432 bayt, 32 MiB |
| Kernel boyutu | 9.206.542 bayt |
| Orijinal sıkıştırılmış ramdisk | 6.419.982 bayt |
| Recovery DTBO boyutu | 27.346 bayt |
| DTB boyutu | 69.074 bayt |
| Ramdisk sıkıştırması | gzip |
| Ramdisk arşivi | CPIO newc |
| Orijinal AVB algoritması | SHA256_RSA2048 |

Başlıktaki komut satırında `bootopt=64S3,32N2,64N2 buildvariant=user` bulunuyordu. Bu komut satırı değiştirilmedi.

Ramdisk gzip ile açıldı. CPIO girdilerinin adları, içerikleri, izinleri, sahiplik bilgileri ve bağlantı bilgileri kaydedildi. Analiz için normal dosyalar ayrı dizine çıkarıldı. Yeniden paketleme sırasında asıl CPIO kayıtları kullanıldı, böylece sembolik bağlantılar ve diğer dosya türleri korundu.

## 2. ZIP imza doğrulamasının bulunması

`system/bin/recovery`, ARM64 ELF biçiminde ve sembolleri temizlenmiş bir programdı. Bu yüzden yalnızca fonksiyon adına bakarak yama yapılmadı.

Capstone ve pyelftools ile makine kodu incelendi. Şu metinlere yapılan kod referansları takip edildi:

- `Verifying update package...`
- `Failed to verify package`
- `failed to verify whole-file signature`
- `footer is wrong`

İncelenen akışta `0x344E0` adresindeki fonksiyon sertifikaları yüklüyor, `0x35EC0` adresindeki bütün-dosya doğrulama rutinini çağırıyor ve sonucu boolean olarak döndürüyordu. Alt doğrulayıcı sıfır döndürdüğünde üst fonksiyon başarı için `1` döndürüyordu.

Üst fonksiyonun ilk sekiz baytı, doğrudan başarı döndürecek şekilde değiştirildi.

| Özellik | Değer |
| --- | --- |
| Dosya | `system/bin/recovery` |
| ELF sanal adresi | `0x344E0` |
| Dosya içi ofset | `0x344E0` |
| Eski baytlar | `ff0302d1f72300f9` |
| Yeni baytlar | `20008052c0035fd6` |

Yeni ARM64 komutları:

```asm
mov w0, #1
ret
```

Böylece fonksiyon, sertifika ve ZIP imzasını incelemeden başarı döndürüyor. Orijinal fonksiyonun gövdesi dosyada duruyor, ancak girişten artık çalıştırılmıyor.

Bu değişiklik ZIP içindeki kurulum betiğini, cihaz kontrollerini veya paket uyumluluğunu düzeltmez. İmzasız bir paket bu aşamayı geçse de başka bir nedenle kurulamayabilir.

Adresler yalnızca incelenen dosya için geçerlidir. Başka bir recovery sürümüne aynı ofsetler uygulanmamalıdır.

## 3. Root ADB için yapılan değişiklikler

### ADB programındaki kontrol

Ramdisk içinde `system/bin/adbd`, shell ve gerekli kütüphaneler zaten vardı. Başka cihazdan ADB programı eklenmedi.

`adbd_main` içindeki akış incelendi. `0x2D10` adresindeki yardımcı fonksiyonun sonucu, root yetkisini düşürme ve ADB kimlik doğrulamasına ilişkin özelliklerin kullanılmasını kontrol ediyordu. Bu fonksiyonun başlangıcı da doğrudan `1` döndürecek şekilde yamalandı.

| Özellik | Değer |
| --- | --- |
| Dosya | `system/bin/adbd` |
| ELF sanal adresi | `0x2D10` |
| Dosya içi ofset | `0x2D10` |
| Eski baytlar | `ff0302d1f44f06a9` |
| Yeni baytlar | `20008052c0035fd6` |

Bu, bootloader kilidini açmaz. Recovery içindeki adbd programının ilgili kararını değiştirir.

### Sistem özellikleri

`prop.default` içinde şu değişiklikler yapıldı:

```diff
-ro.secure=1
+ro.secure=0
-ro.adb.secure=1
+ro.adb.secure=0
-ro.debuggable=0
+ro.debuggable=1
```

Dosyanın sonuna şu satır eklendi:

```properties
service.adb.root=1
```

Hedef, adbd'nin başlangıçta root yetkisini koruması ve recovery ekranında RSA onayı gerektirmeden USB bağlantısı kabul etmesidir. Bu recovery açıkken USB erişimi olan bilgisayarın root ADB bağlantısı kurabilmesi amaçlanmıştır.

Mevcut `init.rc` zaten `ro.debuggable=1` olduğunda `sys.usb.config=adb` ayarlayan ve USB ADB servisini başlatan kuralları içeriyordu. Bunlar korundu.

### SELinux bağlamı

ADB servisinin komut satırından şu argüman kaldırıldı:

```text
--root_seclabel=u:r:su:s0
```

Servis satırı şu hale geldi:

```text
service adbd /system/bin/adbd --device_banner=recovery
```

Servisin mevcut `seclabel u:r:adbd:s0` satırı korundu. Böylece root için ayrıca `su` alanına geçiş yapılması hedeflenmedi.

## 4. SELinux politikasının düzenlenmesi

`sepolicy`, libsepol ile okunarak metin biçiminde incelendi. Politika sürümü 30'du. ADB'nin shell çalıştırırken `shell` alanına geçiş kuralı da görüldü.

Yalnızca iki alan permissive yapıldı:

```text
permissive adbd;
permissive shell;
```

Bu imajdaki tür numaraları:

| Alan | Tür numarası |
| --- | --- |
| adbd | 424 |
| shell | 2316 |

İkili politika içindeki boş permissive bitmap, bu iki türü içerecek şekilde yeniden oluşturuldu. Sistemin tamamına `setenforce 0` uygulanmadı. Diğer alanların politikası değiştirilmedi.

Değişiklikten önceki ve sonraki politika libsepol ile ayrı ayrı okunup metne çevrildi. Metinler karşılaştırıldı. Tek farkın yukarıdaki iki permissive bildirimi olduğu doğrulandı.

## 5. Ramdisk ve boot imajının yeniden oluşturulması

Toplam beş dosya değişti:

```text
init.rc
prop.default
sepolicy
system/bin/adbd
system/bin/recovery
```

CPIO arşivi orijinal girdi sırası ve metadata bilgileri korunarak yeniden yazıldı. Değişen içeriklerin boyut alanları güncellendi. CPIO kayıtları dört bayta, arşivin sonu 512 bayta hizalandı. Ardından gzip ile sıkıştırıldı.

Boot image yeniden oluşturulurken:

1. Orijinal kernel aynen alındı.
2. Yeni ramdisk yerleştirildi.
3. Recovery DTBO ve DTB aynen korundu.
4. Ramdisk boyutu ve recovery DTBO ofseti başlıkta güncellendi.
5. Bileşenler 2048 baytlık sayfa sınırlarına hizalandı.
6. Başlıktaki SHA-1 kimliği yeniden hesaplandı.

Başlık SHA-1 hesabında sırasıyla kernel, ramdisk, boş second bileşeni, recovery DTBO ve DTB kullanıldı. Her bileşenin ardından little-endian 32 bit boyutu hash hesabına eklendi.

Kernel SHA-256:

```text
588d40706ae486acb25874ee7487382377c585290901821bb5770ef64f94eb37
```

## 6. AVB metadatasının yeniden oluşturulması

Orijinal üretici imzası, içerik değiştikten sonra geçerli kalamaz. Üreticinin özel anahtarı kullanılmadı ve üretici imzası yeniden üretilmedi.

Eski AVB verisi yeni imaja taşınmadı. `avbtool` ile SHA-256 hash tanımlayıcısı içeren, imzasız bir AVB footer oluşturuldu. Algoritma `NONE` olarak ayarlandı. Bölüm adı `recovery`, toplam bölüm boyutu 32 MiB olarak korundu.

Uygulanan komut:

```bash
python3 avbtool.py add_hash_footer \
  --image recovery-s48-adbroot-nosig.img \
  --partition_name recovery \
  --partition_size 33554432 \
  --algorithm NONE \
  --hash_algorithm sha256 \
  --salt 36999f5d4fc87864193a22838237afdbbb266a246e01e0cbf6239c4bb233e217
```

Yeni AVB öncesi imaj boyutu 15.704.064 bayt oldu. Footer ve dolgu eklendikten sonra dosya yine 33.554.432 bayttır.

AVB hash kontrolünün geçmesi, bootloader'ın bu imajı kabul edeceği anlamına gelmez. Kilit ve üst vbmeta doğrulama ayarları cihaz üzerinde belirleyicidir. Bu işlem cihazın vbmeta bölümünü değiştirmedi.

## 7. Yapılan doğrulamalar

| Kontrol | Sonuç |
| --- | --- |
| Yama öncesi sekiz baytın beklenen değerle eşleşmesi | Geçti |
| İki ARM64 yamasının Unicorn ile çalıştırılması | İkisi de `1` döndürdü, stack pointer değişmedi |
| Yeni gzip verisinin tekrar açılması | Geçti |
| CPIO girdi sayısı, sırası ve metadata karşılaştırması | Geçti |
| Yalnızca belirtilen beş dosyanın değişmesi | Doğrulandı |
| Kernel, recovery DTBO ve DTB bayt karşılaştırması | Birebir aynı |
| SELinux politikasının libsepol ile okunması | Geçti |
| Politika metninde yalnızca iki permissive bildiriminin eklenmesi | Doğrulandı |
| Boot header SHA-1 kontrolü | Geçti |
| `avbtool verify_image` ile footer ve SHA-256 kontrolü | Geçti |
| Son dosya boyutu | 32 MiB |
| Cihazda açılış, ADB ve ZIP kurulumu | Yapılmadı |

Unicorn testi yalnızca değiştirilen kısa komut dizilerinin dönüş değerini kontrol etti. Recovery'nin veya adbd'nin tamamını emüle etmedi.

`avbtool verify_image` bölüm adına göre `recovery.img` aradığı için doğrulama sırasında yamalı dosyaya işaret eden bu isimde bir sembolik bağlantı kullanıldı. İlk denemede farklı dosya adı yüzünden alınan dosya-bulunamadı hatası bu şekilde giderildi. Son kontrol başarıyla tamamlandı.

## 8. Dosya kimlikleri

Orijinal `recovery(1).img` SHA-256:

```text
5cc75d36ce7dc7abb42ceeeeb03e070cc0925c464b865912c077e5561ba47dd4
```

Yamalı `recovery-s48-adbroot-nosig.img` SHA-256:

```text
88c4eadd5f59da2cef7a6058c09bc90fb50787f4484313629e172bc000a4295c
```

## 9. Cihaz üzerinde doğrulanması gerekenler

Recovery açıldıktan sonra bilgisayardan:

```bash
adb devices
adb shell id
adb shell getprop ro.debuggable
adb shell getprop ro.secure
adb shell getprop ro.adb.secure
adb shell cat /proc/self/attr/current
```

`adb shell id` için hedef çıktı `uid=0(root)` içermesidir. Özelliklerin sırasıyla `1`, `0`, `0` olması beklenir. ADB zaten root başladığı için ayrıca `adb root` komutuna ihtiyaç olmaması hedeflendi.

Genel `getenforce` çıktısı `Enforcing` kalabilir. Yalnızca adbd ve shell alanları permissive yapıldığı için bu tek başına yamanın çalışmadığını göstermez.

ZIP testi, stok recovery ile uyumlu bir kurulum paketiyle yapılmalıdır. İmza kontrolünün atlanması TWRP'ye özel paketleri, uyumsuz update-binary dosyalarını veya yanlış bölüm varsayımlarını düzeltmez.

## Kullanılan araçlar

Python `struct`, `gzip`, `hashlib` ve `ctypes` modülleri, pyelftools, Capstone, Unicorn, libsepol ve avbtool kullanıldı. Kaynak koddan yeni kernel veya recovery derlenmedi. Mevcut imajın ramdisk içeriği düzenlendi ve imaj yeniden paketlendi.
