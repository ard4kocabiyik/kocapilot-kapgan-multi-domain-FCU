# 🛡️ Kocapilot Kapgan Multi-Domain FCU







**Kocapilot Kapgan Multi-Domain FCU**, hava (İHA / Multikopter / Sabit Kanat), kara (İKA) ve su üstü/altı (İDA) insansız otonom sistemlerde görev yapmak üzere **Kocapilot Otonom Sistemler** tarafından geliştirilmiş, endüstriyel koruma katmanlarına sahip hibrit bir uçuş ve hareket kontrol ünitesidir (**Flight Control Unit**).

Tasarım; düşük gecikmeli gerçek zamanlı kontrol döngülerini yürüten **STM32F405RGTx (ARM® Cortex®-M4F)** mikrodenetleyicisi ile yüksek seviyeli görev planlama, görüntü işleme ve otonom karar algoritmalarını koşturan **Raspberry Pi Zero 2W** yardımcı bilgisayarını (**Companion Computer**) tek bir 4 katmanlı donanım mimarisinde birleştirir. Bağımsız SPI veriyolları üzerinde koşan **çift yedekli IMU (BMI270 + MPU-6500)** yapısı, **74LVC1G125** tabanlı ters besleme (**back-powering**) izolasyonu, tüm dış arayüzlerde çok katmanlı **TVS/ESD koruması** ve **kilitlenebilir DB25** endüstriyel ana konnektör mimarisi ile zorlu saha koşullarında yüksek sinyal bütünlüğü ve donanımsal güvenilirlik sunar.

## ✨ Öne Çıkan Özellikler

* **Hibrit Kontrol Mimarisi:** Gerçek zamanlı kontrol için **STM32F405RGTx** ve yüksek seviyeli otonomi (ROS 2 / MAVLink) için doğrudan kart üzerine entegre **40-pin Raspberry Pi Zero 2W** soketi (`J3`).
* **Tam Güç İzolasyonu (Zero Back-Powering):** MCU (`3.3V_DIGI`) ve Raspberry Pi (`PI_3V3`) güç bölgeleri arasında `2N7002` MOSFET (`Q1`) ve 3 adet `74LVC1G125` (`U5`, `U6`, `U7`) tri-state tampon kapısı ile otomatik lojik izolasyon.
* **Çift Yedekli (Dual-Redundant) IMU & Barometre:** Ayrı SPI hatlarında çalışan birincil **Bosch BMI270** (`SPI1`) ve ikincil **TDK InvenSense MPU-6500** (`SPI2`) 6 eksenli IMU sensörleri ile **Bosch BMP280** (`I2C1`) barometrik basınç sensörü.
* **Donanımsal RC Sinyal Evirici (Smart RC Inverter):** `74LVC1G86` (`U11`) XOR kapısı ve `JP1` lehim köprüsü ile hem **SBUS** (evrilmiş lojik) hem de **CRSF / IBUS / PPM** (normal lojik) alıcı protokollerine donanımsal uyumluluk.
* **Endüstriyel ESD, EMI ve Aşırı Akım Koruması:** Çift PTC sigortalı güç girişi (`2A` ana giriş + `1A` USB), `SMAJ6.0A` (400W TVS), `ESDA6V1-5SC6`, `USBLC6-2SC6`, `PRTR5V0U2X`, `PESD2CAN` ve `PESD5V0S1UL` koruma dizileri.
* **Gürültüden Arındırılmış Analog/Sensör Beslemesi:** MCU analog referansı (`VDDA`), her iki IMU, barometre ve MicroSD kart için 5 adet ayrı **Murata BLM18EG601SN1D** ferrit boncuk (`FB1–FB5`) ile filtrelenmiş yerel güç adaları.

## 🔧 Donanım Özellikleri (Hardware Specifications)

### 1. İşlemci ve Yardımcı Bilgisayar (MCU & Companion Computer)

| Parametre | Teknik Özellik / Bileşen | Açıklama ve Tasarım Detayı |
| :--- | :--- | :--- |
| **Ana Mikrodenetleyici (`U2`)** | **STM32F405RGTx** (LQFP-64) | 32-bit ARM® Cortex®-M4F (FPU), 168 MHz, 1 MB Flash, 192 KB SRAM |
| **Saat Kaynağı (`Y1`)** | **ABM8-8.000MHZ-10-B2-T** | 8.000 MHz SMD Kristal (3.2×2.5 mm), `12 pF` yük kapasitörleri (`C33`, `C38`) ve `100 Ω` seri direnç (`R57`) |
| **Yardımcı Bilgisayar (`J3`)** | **Raspberry Pi Zero 2W** Header | 2×20 (40-Pin, 2.54 mm) dikey dişi konnektör; `5V1` besleme, `USART1` ve `GPIO17` (`GPIO_ZERO`) bağlantılı |
| **Lojik İzolasyon (`U5–U7`, `Q1`)** | **3× 74LVC1G125** + **2N7002** | `I_off` kısmi güç kapatma korumalı tamponlar ile MCU ↔ RPi arasında çift yönlü back-powering koruması |
| **Reset & Boot Kontrolü** | **SW1** (KMR2) & **JP2** | `SW1` SMD reset butonu (`10 kΩ` pull-up + `100 nF` filtre) ve `JP2` lehim jumper'ı ile `BOOT0` (DFU) seçimi |

### 2. Güç Katı ve Voltaj Regülasyonu (Power Architecture)

| Güç Hattı / Eleman | Bileşen / Değer | Görev ve Koruma Mekanizması |
| :--- | :--- | :--- |
| **Ana Harici Giriş (`5V1`)** | **DB25 Pin 1, 2, 3** | Harici regüleli 5.1V giriş; **F1 (2A PTC)** sigorta, **D1 (SMAJ6.0A 400W TVS)** ve **C46 (100 µF / 16V)** filtre |
| **USB Güç Girişi (`VBUS`)** | **USB Type-C (`J6`)** | **F2 (1A PTC)** sıfırlanabilir sigorta ve `4.7 µF + 100 nF` dekuplaj; `CC1/CC2` üzerinde `5.1 kΩ` UFP/Sink dirençleri |
| **Güç Birleştirme (OR-ing)** | **2× SS34** (`D10`, `D12`) | `5V1` ve `VBUS` kaynaklarını kesintisiz birleştiren `40V 3A` SMA Schottky diyotlar (`+5V` iç veriyolu) |
| **3.3V Ana Regülatör (`U10`)** | **LM1084-3.3** (TO-263-3) | `+5V` hattından `3.3V_DIGI` üreten **5A** kapasiteli LDO regülatör (`10 µF` giriş/çıkış + çoklu `100 nF` dekuplaj) |
| **Ferrit Boncuk İzolasyonu** | **5× BLM18EG601SN1D** | `FB1` (MicroSD), `FB2` (BMP280), `FB3` (MPU-6500), `FB4` (BMI270) ve `FB5` (`STM32 VDDA`) için ayrı gürültü filtreleri |
| **Durum LED'leri** | **4× 0603 SMD LED** | `D13` (Amber: `VBUS`), `D14` (Kırmızı: `+5V`), `D5` (Yeşil: `3.3V_DIGI`), `D6` (Mavi: `PB4 / MCU_LED` durum) |
| **Test Noktaları** | **TP1 – TP4** (`Ø1.0 mm`) | `TP1` (`+5V`), `TP2` (`3.3V_DIGI`), `TP3` (`5V1`), `TP4` (`GND`) donanımsal doğrulama pedleri |

### 3. Sensörler (Sensors & Blackbox Storage)

| Sensör / Modül | Model / Kılıf | Veri Yolu & MCU Pinleri | Teknik Özellikler ve Tasarım Notları |
| :--- | :--- | :--- | :--- |
| **Birincil IMU (`U4`)** | **Bosch BMI270** (LGA-14) | **SPI1** (`PA4–PA7`) | 6 eksenli düşük gürültülü IMU; çift donanımsal kesme (`PC5: INT1`, `PC4: INT2`) ve `FB4` izole besleme |
| **İkincil IMU (`U1`)** | **InvenSense MPU-6500** (QFN-24) | **SPI2** (`PB12–PB15`) | 6 eksenli yedek IMU; bağımsız SPI veriyolu, donanımsal kesme (`PC0: MPU_INT`) ve `FB3` izole besleme |
| **Barometre (`U3`)** | **Bosch BMP280** (LGA-8) | **I2C1** (`PB6: SCL`, `PB7: SDA`) | Mutlak basınç ve irtifa sensörü; `2.2 kΩ` pull-up (`R53`, `R55`), `FB2` izole besleme |
| **Kara Kutu Kayıt (`J1`)** | **Hirose DM3AT-SF-PEJM5** | **SPI3** (`PC10–PC12`, `PC2: CS`) | 21 MHz optimize MicroSD yuvası; `10 Ω / 22 Ω` seri sönümleme dirençleri, `10 kΩ` pull-up dizisi ve `FB1` filtre |

### 4. Haberleşme Protokolleri ve Çevre Arayüzleri (Interfaces & I/O)

| Arayüz | Kontrolcü / Koruma | MCU Pinleri | Konektör & Protokol Detayları |
| :--- | :--- | :--- | :--- |
| **CAN Bus (`CAN1`)** | **SN65HVD230** (`U8`) + **PESD2CAN** (`CR1`) | `PB8` (RX), `PB9` (TX) | **DB25** (`Pin 15: CAN_P`, `Pin 16: CAN_N`); `2× 60.4 Ω` (`120.8 Ω`) split termination + `4.7 nF` (`C23`) EMI filtre |
| **USB 2.0 (`USB_OTG_FS`)** | **USBLC6-2SC6** (`U9`) + **2× PESD5V0S1UL** | `PA11` (D-), `PA12` (D+) | **USB Type-C** (`J6`, 16-Pin); `22 Ω` empedans dirençleri ve `1 MΩ // 4.7 nF` şasi/shield izolasyonu |
| **Companion UART (`USART1`)** | **2× 74LVC1G125** (`U5`, `U7`) | `PA9` (TX), `PA10` (RX) | **RPi Zero 2W** (`J3` Pin 8 & 10) ile `100 Ω` seri korumalı, izole yüksek hızlı veri hattı |
| **Telemetri UART (`USART2`)** | **ESDA6V1-5SC6** (`D2`, `D7`) | `PA2` (TX), `PA3` (RX), `PA0` (CTS), `PA1` (RTS) | **DB25** (`Pin 9, 10, 11, 12`); tam donanımsal akış kontrolü (RTS/CTS) ve `150 Ω` seri koruma |
| **GPS / Harici UART (`USART3`)** | **ESDA6V1-5SC6** (`D4`, `D7`) | `PB10` (TX), `PB11` (RX) | **DB25** (`Pin 13: TX`, `Pin 14: RX`); `150 Ω / 100 Ω` seri direnç ve `10 kΩ` pull-up destekli |
| **RC Alıcı Girişi (`UART5`)** | **74LVC1G86** (`U11` XOR) + **ESDA6V1** (`D4`) | `PD2` (RX) *(+ opsiyonel `PA8`)* | **DB25** (`Pin 19`); `JP1` ile seçilebilir donanımsal sinyal evirici (SBUS / CRSF / IBUS / PPM) |
| **Harici I2C (`I2C1`)** | **PRTR5V0U2X** (`D3`) | `PB6` (SCL), `PB7` (SDA) | **DB25** (`Pin 17: SDA`, `Pin 18: SCL`); `22 Ω` seri direnç ve ultra düşük kapasitanslı ESD koruması |
| **PWM / Aktüatör (6 Kanal)** | **ESDA6V1-5SC6** (`D2`, `D4`, `D7`) | `PC6–PC9` (`TIM8_CH1–4`), `PB0–PB1` (`TIM3_CH3–4`) | **DB25** (`Pin 20–25`); `150 Ω` seri koruma ve `10 kΩ` güvenlik pull-down dirençli motor/servo çıkışları |
| **Buzzer Çıkışı (`J5`)** | **AO3400A** (`Q2`) + **SS34** (`D11`) | `PC13` (`BUZZER`) | **1×02 (1.27 mm)**; `5.7A` N-MOSFET düşük taraf sürücüsü ve Schottky flyback diyodu |
| **SWD Debug (`J4`)** | Doğrudan Hat + Pull-up/down | `PA13` (SWDIO), `PA14` (SWCLK), `NRST` | **1×05 (2.54 mm)** dikey pin dizisi; `10 kΩ` pull-up (`SWDIO`, `NRST`) ve pull-down (`SWCLK`) |

### 5. DB25 Ana Konnektör (`J2`) Pin Haritası

| DB25 Pin | Sinyal Adı | MCU Pini | Yön / Tip | Koruma ve Hat Yapısı |
| :---: | :--- | :---: | :--- | :--- |
| **1, 2, 3** | `5V1_IN` | — | Güç Girişi | `F1` (2A PTC Sigorta) + `D1` (SMAJ6.0A TVS) |
| **4, 5, 6, 7, 8** | `GND` | — | Toprak | 4 katmanlı ortak GND referans düzlemi |
| **9** | `USART2_TX` | `PA2` | Çıkış (UART) | `150 Ω` (`R19`) + `10 kΩ` pull-up + `D2` (`ESDA6V1-5SC6`) |
| **10** | `USART2_RX` | `PA3` | Giriş (UART) | `150 Ω` (`R11`) + `10 kΩ` pull-up + `D2` (`ESDA6V1-5SC6`) |
| **11** | `USART2_RTS` | `PA1` | Çıkış (Akış Kontrol) | `150 Ω` (`R61`) + `10 kΩ` pull-up + `D7` (`ESDA6V1-5SC6`) |
| **12** | `USART2_CTS` | `PA0` | Giriş (Akış Kontrol) | `150 Ω` (`R62`) + `10 kΩ` pull-up + `D7` (`ESDA6V1-5SC6`) |
| **13** | `USART3_TX` | `PB10` | Çıkış (UART) | `150 Ω` (`R71`) + `10 kΩ` pull-up + `D7` (`ESDA6V1-5SC6`) |
| **14** | `USART3_RX` | `PB11` | Giriş (UART) | `100 Ω` (`R24`) + `10 kΩ` pull-up + `D4` (`ESDA6V1-5SC6`) |
| **15** | `CAN_P` (CANH) | `PB8 / PB9` | Diferansiyel Çift | `SN65HVD230` (`U8`) + `CR1` (`PESD2CAN`) + `60.4 Ω` split term. |
| **16** | `CAN_N` (CANL) | `PB8 / PB9` | Diferansiyel Çift | `SN65HVD230` (`U8`) + `CR1` (`PESD2CAN`) + `60.4 Ω` split term. |
| **17** | `I2C1_SDA` | `PB7` | Çift Yönlü (I2C) | `22 Ω` (`R30`) + `2.2 kΩ` pull-up + `D3` (`PRTR5V0U2X`) |
| **18** | `I2C1_SCL` | `PB6` | Çıkış/Saat (I2C) | `22 Ω` (`R14`) + `2.2 kΩ` pull-up + `D3` (`PRTR5V0U2X`) |
| **19** | `RC_IN` | `PD2` (`PA8`) | Giriş (RC Alıcı) | `150 Ω` (`R17`) + `10 kΩ` pull-down + `U11` (`74LVC1G86`) + `D4` |
| **20** | `TIM8_CH1` | `PC6` | Çıkış (PWM / DShot) | `150 Ω` (`R16`) + `10 kΩ` pull-down + `D4` (`ESDA6V1-5SC6`) |
| **21** | `TIM8_CH2` | `PC7` | Çıkış (PWM / DShot) | `150 Ω` (`R13`) + `10 kΩ` pull-down + `D2` (`ESDA6V1-5SC6`) |
| **22** | `TIM8_CH3` | `PC8` | Çıkış (PWM / DShot) | `150 Ω` (`R15`) + `10 kΩ` pull-down + `D2` (`ESDA6V1-5SC6`) |
| **23** | `TIM8_CH4` | `PC9` | Çıkış (PWM / DShot) | `150 Ω` (`R48`) + `10 kΩ` pull-down + `D2` (`ESDA6V1-5SC6`) |
| **24** | `TIM3_CH3` | `PB0` | Çıkış (PWM / Servo) | `150 Ω` (`R51`) + `10 kΩ` pull-down + `D7` (`ESDA6V1-5SC6`) |
| **25** | `TIM3_CH4` | `PB1` | Çıkış (PWM / Servo) | `150 Ω` (`R72`) + `10 kΩ` pull-down + `D7` (`ESDA6V1-5SC6`) |
| **SHIELD (0)** | `DB25_SHIELD` | — | Gövde / Şasi | `R1` (`1 MΩ`) // `C1` (`4.7 nF`) ile GND'ye izole bağlantı |

### 6. PCB Katman Yapısı, Fiziksel Boyutlar ve Empedans Kuralları

| Özellik | Değer / Parametre | Mühendislik Detayı |
| :--- | :--- | :--- |
| **Kart Boyutları** | **70.0 mm × 57.0 mm** | `3.0 mm` köşe radyuslu dikdörtgen form faktörü (`Edge.Cuts`: X=`50–120 mm`, Y=`50–107 mm`) |
| **Montaj Delikleri (`H1–H4`)** | **4× M2 (`Ø2.2 mm`)** | **63.0 mm × 50.0 mm** merkezden merkeze montaj deseni; her delik çevresinde `5×5 mm` çok katmanlı bakır keepout alanı |
| **Katman Sayısı & Kalınlık** | **4 Katman (`1.6 mm`)** | `F.Cu` (35 µm) – `Prepreg` (0.1 mm) – `In1.Cu` (35 µm) – `Core` (1.24 mm) – `In2.Cu` (35 µm) – `Prepreg` (0.1 mm) – `B.Cu` (35 µm) |
| **Dielektrik Malzeme** | **FR4** ($
arepsilon_r = 4.5$) | Kayıp tanjantı $	an\delta = 0.02$; empedans kontrollü dış katman yönlendirmesi |
| **Diferansiyel Empedans** | **USB 90 Ω** / **CAN 120 Ω** | `USB_Diff_90R` (`0.16 mm` genişlik / `0.15 mm` boşluk), `CAN_Diff_120R` (`0.15 mm` genişlik / `0.25 mm` boşluk) |
| **Güç Hatları (Track Widths)** | **0.25 mm – 1.20 mm** | `5V1_2A`: `1.2 mm` \| `5V_1A25`: `0.6 mm` \| `3V3_VBUS_1A`: `0.45 mm` \| `PI_3V3`: `0.35 mm` \| `Sub_250m`: `0.25 mm` |
| **Bileşen & Serim Yoğunluğu** | **176 Elektronik Bileşen** | Toplam 184 ayak izi (footprint), `2.550` bakır yol segmenti ve `631` via; tüm ERC/DRC testlerinden geçmiştir |

## 📂 Repo İçeriği (Repository Structure)

Bu depo, **Kocapilot Kapgan Multi-Domain FCU** kartının üretim, montaj, mekanik entegrasyon ve akademik doğrulama süreçleri için gereken tüm kaynak dosyaları içerir:

```text
kocapilot-kapgan-multi-domain-FCU/
├── hardware/
│   ├── kocapilot Kapgan.kicad_pro   # KiCad 9.0 proje ve tasarım kuralları (Net Classes, Stackup)
│   ├── kocapilot Kapgan.kicad_sch   # Ana şematik (MCU, IMU'lar, Barometre, RPi, CAN, USB, Güç)
│   ├── Kapgan.kicad_sch             # Alt şematik (MicroSD, DB25, RPi Lojik İzolasyon, ESD)
│   ├── kocapilot Kapgan.kicad_pcb   # 4 katmanlı PCB serim dosyası (70x57 mm, Rev v2.1)
│   └── gerber/                      # Üretime hazır Gerber ve Excellon delik (Drill) dosyaları
├── 3d-models/
│   └── kocapilot-kapgan-fcu.step    # Mekanik şasi ve kabin entegrasyonu için STEP 3D CAD modeli
├── docs/
│   └── academic-reports/            # Akademik raporlamalar, sinyal bütünlüğü ve tasarım dokümanları
├── ibom/
│   └── ibom.html                    # Tarayıcı tabanlı İnteraktif Malzeme Listesi (Interactive HTML BOM)
└── README.md                        # Proje teknik dokümantasyonu
```

* **📐 STEP 3D Model Dosyaları:** İnsansız hava, kara ve deniz araçlarının mekanik gövde/şasi tasarımlarında doğrudan kullanılabilmesi, konnektör (`DB25`, `USB-C`, `MicroSD`, `RPi Zero 2W`) erişim paylarının ve `63.0 mm × 50.0 mm` (M2) montaj eksenlerinin doğrulanabilmesi için tam ölçekli `.step` 3D katı model dosyaları sunulmuştur.
* **📑 Akademik Raporlamalar ve Tasarım Dokümanları:** Kartın mimari tasarım kararlarını, güç bütçesi ve termal analizlerini, yedekli sensör seçim kriterlerini, empedans eşleştirme hesaplamalarını ve ERC/DRC/Timer doğrulama test sonuçlarını içeren kapsamlı akademik raporlar ve mühendislik dokümanları yer almaktadır.
* **🔍 İnteraktif BOM (iBOM) Dosyası:** Elle dizgi, lehimleme, kalite kontrol ve arıza tespiti süreçlerini hızlandırmak amacıyla oluşturulan interaktif HTML BOM (`ibom.html`) dosyası üzerinden kart üzerindeki 176 bileşenin tamamı değer, kılıf ve referans kodu (`U1–U11`, `R1–R81`, `C1–C46`, `D1–D14`) bazında görsel olarak filtrelenebilir.

## 📱 Donanım Doğrulama ve QR Kodu

Üretilen **Kocapilot Kapgan Multi-Domain FCU** kartlarının arka yüzeyinde (`B.SilkS` katmanı, `Kocapilot:kapgan_qr` ayak izi) basılı olan **QR kod**, saha testleri ve donanım doğrulama süreçlerinde teknik dokümanlara, pin haritasına ve interaktif BOM dosyasına anında erişim sağlamak üzere doğrudan bu GitHub deposuna ([github.com/ard4kocabiyik/kocapilot-kapgan-multi-domain-FCU](https://github.com/ard4kocabiyik/kocapilot-kapgan-multi-domain-FCU)) yönlendirmektedir.

## 📬 İletişim & Destek

* **Teknik Referanslar:** Bileşen yerleşimi, lehimleme kılavuzu ve kapsamlı devre analizleri için lütfen depo içerisindeki **İnteraktif BOM (iBOM)** dosyasını ve **Akademik Raporlamalar** dizinini inceleyin.
* **İletişim & İş Birliği:** Projeyle ilgili tüm teknik soru, görüş, öneri ve Ar-Ge iş birliği talepleriniz için **[ard4kocabiyik@gmail.com](mailto:ard4kocabiyik@gmail.com)** e-posta adresi üzerinden iletişime geçebilirsiniz.
