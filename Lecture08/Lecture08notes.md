# Lecture 08 - Power BI İleri DAX Fonksiyonları ve Sipariş Analizi

## DAX Çeşitleri ve İleri Fonksiyonlar

### DAX 1: DIVIDE Fonksiyonu
**Amaç:** IF koşulu belirtmeden koşul ifadesi içerme

```dax
ort_siparis_degeri =
DIVIDE(
    [toplam_satis], 
    DISTINCTCOUNT(Sheet1[ShipName])
)
```

**Açıklama:**
- `DIVIDE` fonksiyonu otomatik olarak sıfıra bölme hatalarını yönetir
- IF koşulu yazmadan güvenli bölme işlemi yapar
- `DISTINCTCOUNT` ile benzersiz müşteri sayısını alır

### DAX 2: CALCULATE + DATESBETWEEN
**Amaç:** Belirli tarih aralığında filtreleme

```dax
satislar_2024 =
CALCULATE(
    [toplam_satis],
    DATESBETWEEN(tarih_tablosu[Date], DATE(2024,1,1), DATE(2024,12,31))
)
```

**Açıklama:**
- `CALCULATE` ile filtrelenmiş hesaplama
- `DATESBETWEEN` ile özel tarih aralığı belirleme
- `DATE()` fonksiyonu ile tarih oluşturma

### DAX 3: DATESINPERIOD - Son Belirli Periyot
**Amaç:** Son 30 gün / belli bir periyot içindeki dataları belirleme

#### Son 30 Gün Satışları
```dax
son30gun_satis =
CALCULATE(
    [toplam_satis],
    DATESINPERIOD(
        tarih_tablosu[Date], 
        MAX(tarih_tablosu[Date]),
        -30,
        DAY
    )
)
```

#### Derste Soru: Son 6 Ayın Toplam Satış Adedi Nedir?
```dax
son6ay_satis_adedi =
CALCULATE(
    [toplam_adet],
    DATESINPERIOD(
        tarih_tablosu[Date],
        MAX(tarih_tablosu[Date]),
        -6,
        MONTH
    )
)
```

**Açıklama:**
- `DATESINPERIOD` ile dinamik tarih aralığı
- `MAX()` ile en son tarihi referans alma
- Negatif değer ile geriye doğru gitme
- `DAY`, `MONTH`, `YEAR` gibi zaman birimleri

### DAX 4: DATEDIFF ve AVERAGEX
**Amaç:** İki tarih arası ortalama değer hesaplama

```dax
ort_teslimat_gunu =
AVERAGEX(
    Sheet1,
    DATEDIFF(Sheet1[RequiredDate], Sheet1[ShippedDate], DAY)
)
```

**Soru:** Ortalama teslimat günü ne kadar? (Önceden teslimat yapıyor muyuz?)

**Açıklama:**
- `DATEDIFF` fonksiyonu sayı döndürür
- İki tarih arasındaki farkı hesaplar
- `AVERAGEX` ile satır bazlı ortalama alır
- Teslimat performansı analizi için kullanılır

## Fonksiyon Detayları

### DIVIDE Fonksiyonu
- **Avantajları:** Otomatik hata yönetimi
- **Kullanım Alanları:** Oran hesaplamaları, performans göstergeleri
- **Alternatifi:** IF(ISERROR()) yapısı

### DATESBETWEEN vs DATESINPERIOD
- **DATESBETWEEN:** Sabit başlangıç ve bitiş tarihleri
- **DATESINPERIOD:** Dinamik, son X gün/ay/yıl

### AVERAGEX Fonksiyonu
- **Amaç:** Satır bazlı ortalama hesaplama
- **Farkı:** AVERAGE ile temel farkı
- **Kullanım:** Karmaşık hesaplamalarda satır seviyesi işlemler

## Veri Seti Özellikleri

### turkce_siparis_veri_seti_50000_v3.xlsx
- **Kayıt Sayısı:** 50,000 sipariş
- **Veri Türü:** Türkçe sipariş verileri
- **Sürüm:** v3 (güncellenmiş)

### Olası Veri Yapısı
- **ShipName:** Müşteri/Firma adları
- **RequiredDate:** Talep edilen teslimat tarihi
- **ShippedDate:** Gerçek sevkiyat tarihi
- **Sipariş Tutarları:** Satış miktarları
- **Sipariş Adeti:** Ürün adetleri

## Uygulama Senaryoları

### 1. Müşteri Analizi
```dax
// Müşteri başına ortalama sipariş
musteri_basina_ort_siparis = DIVIDE([toplam_satis], DISTINCTCOUNT(Sheet1[ShipName]))

// Aktif müşteri sayısı (son 3 ay)
aktif_musteri_sayisi = 
CALCULATE(
    DISTINCTCOUNT(Sheet1[ShipName]),
    DATESINPERIOD(tarih_tablosu[Date], MAX(tarih_tablosu[Date]), -3, MONTH)
)
```

### 2. Teslimat Performansı
```dax
// Zamanında teslimat oranı
zamaninda_teslimat_orani = 
DIVIDE(
    COUNTROWS(FILTER(Sheet1, Sheet1[ShippedDate] <= Sheet1[RequiredDate])),
    COUNTROWS(Sheet1)
)

// Ortalama gecikme gün sayısı
ort_gecikme_gun = 
AVERAGEX(
    FILTER(Sheet1, Sheet1[ShippedDate] > Sheet1[RequiredDate]),
    DATEDIFF(Sheet1[RequiredDate], Sheet1[ShippedDate], DAY)
)
```

### 3. Dönemsel Analizler
```dax
// Yıldan yıla büyüme
yoy_buyume = 
VAR gecen_yil = CALCULATE([toplam_satis], SAMEPERIODLASTYEAR(tarih_tablosu[Date]))
VAR bu_yil = [toplam_satis]
RETURN DIVIDE(bu_yil - gecen_yil, gecen_yil)

// Aylık trend analizi
aylik_degisim = 
VAR gecen_ay = CALCULATE([toplam_satis], DATEADD(tarih_tablosu[Date], -1, MONTH))
VAR bu_ay = [toplam_satis]
RETURN DIVIDE(bu_ay - gecen_ay, gecen_ay)
```

## Önemli Kavramlar

### 1. Tarih Fonksiyonları Karşılaştırması
- **DATESBETWEEN:** Sabit aralık
- **DATESINPERIOD:** Dinamik aralık
- **DATEDIFF:** İki tarih arası fark
- **DATE:** Tarih oluşturma

### 2. Hesaplama Kontekstleri
- **Row Context:** AVERAGEX, SUMX gibi X fonksiyonları
- **Filter Context:** CALCULATE ile değiştirme
- **Time Intelligence:** Tarih tablosu gerekliliği

### 3. Performans İpuçları
- `DISTINCTCOUNT` yerine `VALUES` kullanımı (uygun durumlarda)
- Tarih tablosu ile ilişki kurmanın önemi
- CALCULATE'in filter context'i değiştirme özelliği

### 4. Hata Yönetimi
- `DIVIDE` ile otomatik sıfıra bölme koruması
- `IFERROR` ile manuel hata yönetimi
- `BLANK()` ile boş değer döndürme

## Dashboard Önerileri

### KPI Kartları
1. **Toplam Satış:** [toplam_satis]
2. **Ortalama Sipariş Değeri:** [ort_siparis_degeri]
3. **Son 30 Gün Satış:** [son30gun_satis]
4. **Ortalama Teslimat Süresi:** [ort_teslimat_gunu]

### Grafikler
1. **Zaman Serisi:** Aylık satış trendi
2. **Müşteri Analizi:** Top müşteriler tablosu
3. **Teslimat Performansı:** Gecikme analizi
4. **Dönemsel Karşılaştırma:** YoY, MoM değişimler

Bu ders, Power BI'da ileri düzey tarih fonksiyonları ve sipariş analizi konularında derinlemesine bilgi sağlamaktadır.
