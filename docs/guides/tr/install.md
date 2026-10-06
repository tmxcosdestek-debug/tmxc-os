# TMXC OS - Donanım Kurulum Kılavuzu

## ⚠️ Güvenlik Uyarısı

**ÖNEMLİ:** TMXC OS'u fiziksel donanıma yüklemek, cihaz yazılımını değiştirmeyi, önyükleyicilerin kilidini açmayı ve özel işletim sistemi imajları flaşlamayı içerir. Bu işlemler şunlara neden olabilir:

- Cihaz üreticisi garantisini geçersiz kılabilir
- Yanlış yapılırsa cihazı brick edebilir (kullanılamaz hale getirebilir)
- Uygun önlemler alınmazsa cihazı güvenlik risklerine maruz bırakabilir
- Kalıcı veri kaybına neden olabilir

**Kendi riskinizle ilerleyin.** TMXC OS geliştirme ekibi, cihazınıza gelen herhangi bir hasardan veya veri kaybından sorumlu değildir. İlerlemeden önce verilerinizi yedekleyin.

## Ön Koşullar

### Donanım Gereksinimleri
- TMXC OS uyumlu cihaz (bkz. [DEVICE_COMPATIBILITY.md](../../DEVICE_COMPATIBILITY.md))
- Bilgisayara bağlantı için USB veri kablosu
- İnternet bağlantısı olan bilgisayar
- Hedef cihazda en az %50 pil şarjı

### Yazılım Gereksinimleri
- TMXC OS aktivasyon anahtarı (tmxc.os.destek@gmail.com adresinden alın)
- Bilgisayarınız için Platform araçları (ADB/Fastboot)
- TMXC OS cihaz imajı (.img veya .bin dosyası)
- Cihaza özel önyükleyici kilidi açma aracı (üreticiye göre değişir)

## Kurulum Süreci

### Adım 1: Aktivasyon Anahtarı Alma

TMXC OS'u yüklemek önce geçerli bir aktivasyon anahtarı almalısınız:

1. **tmxc.os.destek@gmail.com** adresine e-posta gönderin
2. Cihaz modelinizi ve seri numaranızı ekleyin
3. Aktivasyon anahtarı teslimini bekleyin (genellikle 24-48 saat)
4. Aktivasyon anahtarınızı güvenli tutun - ilk önyüklemede gerekecek

### Adım 2: Bilgisayarınızı Hazırlayın

**Windows:**
```powershell
# Android Developer web sitesinden platform-tools indirin
# C:\platform-tools konumuna çıkarın
# Sistem PATH'e ekleyin
```

**Linux:**
```bash
sudo apt-get install android-tools-adb android-tools-fastboot
```

**macOS:**
```bash
brew install android-platform-tools
```

### Adım 3: Geliştirici Seçeneklerini Etkinleştirin

1. **Ayarlar** → **Telefon Hakkında** bölümüne gidin
2. Geliştirici Seçeneklerini etkinleştirmek için **Derleme Numarası**'na 7 kez dokunun
3. **Ayarlar** → **Geliştirici Seçenekleri** bölümüne geri gidin
4. **USB Hata Ayıklama**'yı etkinleştirin
5. **OEM Kilidini Açma**'yı etkinleştirin (varsa)

### Adım 4: Önyükleyici Kilidini Açın

**⚠️ UYARI:** Bu işlem cihazınızdaki tüm verileri silecektir.

#### Çoğu Android Cihaz İçin:

```bash
# Önyükleyiciye yeniden başlat
adb reboot bootloader

# Önyükleyici kilidini açın (komut üreticiye göre değişir)
fastboot oem unlock
# VEYA
fastboot flashing unlock
```

#### Cihaza Özel Talimatlar:

**Samsung Cihazları:**
- Odin aracını indirin
- Samsung önyükleyici kilidi açma paketini indirin
- Üreticiye özel kilidi açma sürecini izleyin

**Xiaomi Cihazları:**
- Mi Unlock web sitesinden kilidi açma izni isteyin
- Mi Flash Unlock aracını kullanın
- Ekrandaki talimatları izleyin

**Google Pixel:**
- Geliştirici Seçeneklerinde OEM kilidini açmayı etkinleştirin
- `fastboot flashing unlock` komutunu kullanın

**OnePlus:**
- Geliştirici Seçeneklerinde Gelişmiş Yeniden Başlatmayı etkinleştirin
- `fastboot oem unlock` komutunu kullanın

### Adım 5: Fastboot Moduna Girin

```bash
# Kapalı durumdan
# Hacim Azaltma + Güç tuşuna aynı anda basılı tutun
# VEYA ADB kullanın
adb reboot bootloader
```

Bağlantıyı doğrulayın:
```bash
fastboot devices
```

### Adım 6: TMXC OS İmajını Flaşlayın

```bash
# TMXC OS imajını uygun bölüme flaşlayın
fastboot flash boot tmxc_os_boot.img
fastboot flash system tmxc_os_system.img
fastboot flash vendor tmxc_os_vendor.img

# VEYA birleşik imajı flaşlayın
fastboot flash boot tmxc_os_combined.img
```

### Adım 7: Kurtarma İmajını Flaşlayın (İsteğe Bağlı)

```bash
fastboot flash recovery tmxc_os_recovery.img
```

### Adım 8: Sisteme Yeniden Başlatın

```bash
fastboot reboot
```

### Adım 9: İlk Önyükleme Yapılandırması

1. **TMXC OS önyüklenecek** (ilk önyükleme 5-10 dakika sürebilir)
2. **Dil Seçimi** - Tercih ettiğiniz dili seçin
3. **Ağ Kurulumu** - Wi-Fi veya mobil ağa bağlanın
4. **Aktivasyon** - İstenildiğinde aktivasyon anahtarınızı girin
5. **Cihaz Kurulumu** - İlk yapılandırmayı tamamlayın

## Sorun Giderme

### Cihaz Önyüklemez

**Cihaz bootloop'da takılı kalırsa:**
```bash
# Kurtarmaya yeniden başlat
adb reboot recovery

# Verileri/ fabrika ayarlarına sıfırla
# Önyükleyiciye yeniden başlat
fastboot flash boot tmxc_os_boot.img
```

### Fastboot Komutları Tanınmıyor

- Platform-tools'un PATH'inizde olduğundan emin olun
- Fastboot yürütülebilir dosyasının tam yolunu kullanmayı deneyin
- USB kablosunun veri kablosu olduğunu, sadece şarj kablosu olmadığını kontrol edin

### Önyükleyici Kilidi Açılamadı

- Cihazınızın önyükleyici kilidi açmayı desteklediğini doğrulayın
- Operatör kısıtlamalarının geçerli olup olmadığını kontrol edin
- Gerekirse üreticiden kilidi açma kodu isteyin

### Aktivasyon Anahtarı Geçersiz

- Anahtarı doğru girdiğinizi doğrulayın
- Anahtar geçersiz görünüyorsa tmxc.os.destek@gmail.com ile iletişime geçin
- Kayıtlı cihaz modelinizin eşleştiğinden emin olun

## Kurulum Sonrası

### Kurulumu Doğrulayın

```bash
# TMXC OS sürümünü kontrol et
adb shell tmxc_version

# Sistem durumunu kontrol et
adb shell tmxc_status
```

### Güvenlik Kurulumu

1. Biyometrik kimlik doğrulamayı etkinleştirin
2. Şifreleme kasalarını yapılandırın
3. İsterseniz hayalet modunu ayarlayın
4. Neural güvenlik duvarı ayarlarını yapılandırın

### Sistemi Güncelleyin

```bash
# Güncellemeleri kontrol et
adb shell tmxc_update check

# Güncellemeleri uygula
adb shell tmxc_update install
```

## Kurtarma ve Geri Yükleme

### Orijinal Yazılımı Geri Yükleme

Cihazınızı stok yazılıma geri yüklemeniz gerekiyorsa:

1. Cihazınız için stok firmware'ı indirin
2. Fastboot moduna yeniden başlatın
3. Stok imajları flaşlayın:
```bash
fastboot flash boot stock_boot.img
fastboot flash system stock_system.img
fastboot flash vendor stock_vendor.img
fastboot flash recovery stock_recovery.img
```

### Acil Durum Kurtarma

Cihaz brick olduysa:
- Cihaza özel unbrick araçlarını kullanın
- TMXC OS desteği ile iletişime geçin
- Profesyonel onarım hizmeti düşünün

## Ek Kaynaklar

- [Cihaz Uyumluluk Listesi](../../DEVICE_COMPATIBILITY.md)
- [Güvenlik Dokümantasyonu](../security/activation_protocol.md)
- [Sorun Giderme Kılavuzu](troubleshooting.md)
- [Topluluk Forumu](https://community.tmx-os.org)

## Destek

Kurulum sorunları için:
- **E-posta:** tmxc.os.destek@gmail.com
- **Dokümantasyon:** [docs.tmx-os.org](https://docs.tmx-os.org)
- **Topluluk:** [community.tmx-os.org](https://community.tmx-os.org)

---

**Son Güncelleme:** 2026-07-15  
**TMXC OS Sürümü:** 1.0.0  
**Kılavuz Sürümü:** 1.0
tmxc os ekibi iyi günler diller kurucu:ödül ensar yılmaz