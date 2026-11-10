
# Kocapilot Kapgan — Donanım Senkronizasyonlu Uçuş Kontrol Kartı

Kapgan; açık kaynak otopilot yazılımını çalıştıran mikrodenetleyicili uçuş kontrolcüsünü ve Linux tabanlı yardımcı bilgisayarı (YB) tek kart yığınında birleştiren, iki saati ayrılmış bir donanım senkron hattıyla ilişkilendiren bir uçuş kontrol kartıdır. Amaç, İHA'da kaydedilen video karelerini uçuş telemetrisiyle (konum, irtifa, yönelim) milisaniyenin altına yakın doğrulukla eşlemek ve bu doğruluğu kamera görüş alanındaki bir LED ile ölçerek raporlamaktır.

Bu depo, TÜBİTAK 2209-A (2026 yılı 1. dönem) başvurusuna konu olan projenin tasarım dosyalarını, devre benzetim modellerini ve teknik raporlarını içerir.

> **Durum (Kasım 2026):** Rev-A şeması ve 4 katmanlı PCB yerleşimi tamamlandı; güç ağı ve SPI sinyal bütünlüğü benzetimleri yapıldı. Rev-A henüz üretilmedi. Ölçüm ve uçuş sonuçları proje süresince bu depoya eklenecektir.

---

## Depo içeriği

| Klasör / dosya | İçerik |
|---|---|
| `.kicad_pro`, `.kicad_sch`, `.kicad_pcb` | KiCad proje, şema ve PCB dosyaları (Rev-A) |
| `ibom.html` | Etkileşimli malzeme listesi; tarayıcıda açılır, dizgi sırasında kullanılır |
| `kocapilot Kapgan.step` | Kartın 3B modeli |
| `kapgan_case.step` | Kartın kutu modeli |
| `Ltspice/Kapgan_Guc_Agi.asc` | Güç ağı benzetimi (5 V giriş, 3,3 V regülatör açılışı) |
| `Ltspice/Kapgan_SPI_SI.asc` | Mikrodenetleyici–IMU SPI hattı sinyal bütünlüğü benzetimi |
| `Docs/Kocapilot_Kapgan_SCH_Raporu.pdf` | Şema tasarım ve inceleme raporu |
| `Docs/Kapgan_Guc_Agi_Simulasyon_Raporu.pdf` | Güç ağı benzetim raporu |
| `Docs/Kapgan_SPI_SI_Simulasyon_Raporu.pdf` | SPI sinyal bütünlüğü benzetim raporu |
| `Docs/Literatür-Taraması.pdf` | Literatür taraması |

## Kart özeti

- 168 MHz, FPU'lu ARM Cortex-M4F çekirdekli 32-bit mikrodenetleyici
- Ayrı SPI hatlarında, farklı üreticilere ait iki adet 6 eksenli MEMS IMU
- I2C arayüzlü MEMS barometrik basınç sensörü
- SPI üzerinden microSD kayıt, CAN alıcı-verici, motor çıkışları
- Linux tabanlı tek kartlı bilgisayara (YB) takılan 40 pinli genişleme bağlantısı
- 4 katmanlı baskı devre kartı
- Besleme: harici 5 V ve USB-C girişleri diyotla birleştirilir; sensör beslemeleri ferrit boncuklarla dijital hattan ayrılır
- YB ile kart arasındaki seri hat ve senkron hattı tampon entegrelerinden geçer; YB kapalıyken bu hatlar yüksek empedansa geçer
- Uçuş kontrolü ayrı mikrodenetleyicide çalıştığı için kart, YB takılı olmadan da otopilot olarak çalışır

## Senkronizasyon yöntemi

1. Mikrodenetleyici, otopilot yazılımının kamera tetikleme işlevi üzerinden saniyede bir senkron darbesi üretir.
2. Aynı darbe, kesme destekli bir geri besleme girişine geri bağlanır; otopilot yazılımı darbenin kendi saatine göre zamanını kaydeder ve MAVLink `CAMERA_FEEDBACK` mesajıyla YB'ye gönderir.
3. YB, darbenin yükselen kenarını Linux GPIO karakter aygıtı arayüzüyle kendi saatine göre zaman damgalar.
4. Darbe çiftlerinden iki saat arasındaki fark ve kayma (sabit fark + doğrusal kayma modeli) kestirilir; her video karesinin zamanı uçuş kontrolcüsü saatine çevrilerek konum ve yönelim ara değerlemeyle bulunur.

Otopilot yazılımının kaynak kodu değiştirilmez; yalnızca kart tanım dosyaları ve parametreler kullanılır.

Eşleme doğruluğu üç yöntemle karşılaştırılır: **K1** telemetri mesajının YB'ye varış zamanı, **K2** yalnızca MAVLink zaman eşleme protokolü, **K3** protokol + donanım senkron darbesi (önerilen yöntem). Gerçek değer, kamera görüş alanındaki ve pin kenarından zaman damgalanan bir LED ile ölçülür.

## Benzetim sonuçları (özet)

| Benzetim | Sonuç |
|---|---|
| Güç ağı | 3,3 V hattı aşım olmadan yaklaşık 3 ms'de oturuyor; açılış tepe akımları sigorta tutma akımlarının altında |
| SPI sinyal bütünlüğü | Örneklenen tüm bitler lojik eşikleri sağlıyor |

Ayrıntılar `Docs/` klasöründeki raporlardadır. Benzetim sonuçları Rev-A üzerinde osiloskop ölçümleriyle karşılaştırılacaktır.

## Yol haritası

| İş paketi | Kapsam |
|---|---|
| İP1 | Rev-A üretimi, dizgi, ilk enerji; kart tanım dosyaları ve otopilot yazılımının yüklenmesi |
| İP2 | YB kayıt servisi, senkron darbesi, saat modeli ve eşleme yazılımı |
| İP3 | Elektriksel ölçümler, tetik gecikmesi ölçümü, masaüstü senkron deneyleri (K1–K3), Rev-B tasarımı |
| İP4 | Rev-B üretimi, uçuş platformunun kurulumu, pervanesiz ve iple bağlı testler |
| İP5 | Pilot kontrolünde, görerek yapılan uçuşlar ve uçuş içi senkron ölçümleri |
| İP6 | İstatistiksel analiz ve veri setinin yayımlanması |

## Ekip

- Arda Kocabıyık — proje yürütücüsü
- Ali Demirci
- Efe Haskuka

Akademik danışman: Sibel Yenikaya — Bursa Uludağ Üniversitesi, Elektrik-Elektronik Mühendisliği Bölümü

## Atıf

A. Kocabıyık, A. Demirci ve E. Haskuka, "Uçuş kontrol kartı tasarım dosyaları, devre benzetim modelleri ve teknik raporlar," GitHub deposu, 2026. https://github.com/ard4kocabiyik/kocapilot-kapgan-multi-domain-FCU
