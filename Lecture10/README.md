# Lecture 10 - Power BI Hızlı Bilgiler ve Pratik DAX Örnekleri

Bu derste Power BI'da sık kullanılan DAX fonksiyonları ve pratik örnekler üzerinde durulmuştur. Özellikle CALCULATE, SUMMARIZE, TOPN ve tarih fonksiyonları detaylandırılmıştır.

## Pratik DAX Örnekleri

### 1. Kategoriye Göre Ürün Sayısı Bulma

#### Soru: Products tablosunda CategoryID'si 2 olan kaç tane ürün var?

```dax
cat_id_2_olanlar = CALCULATE(
    COUNTA(products[ProductID]), 
    products[CategoryID] = 2
)
```

#### Açıklama:
- **CALCULATE**: Filtrelenmiş hesaplama yapar
- **COUNTA**: Boş olmayan değerleri sayar
- **Filtre Koşulu**: `products[CategoryID] = 2` ile belirli kategoriyi filtreler

#### Kullanım Alanları:
- Kategori bazlı stok analizi
- Ürün portföyü değerlendirmesi
- İnventory yönetimi

---

### 2. Ülke Bazında Freight Toplamı Tablosu

#### Soru: Ülke bazında Freight toplamını getiren bir tablo oluşturun?

```dax
ulkeler_freight = SUMMARIZE(
    orders,
    orders[ShipCountry], 
    "freight toplam", SUM(orders[Freight])
)
```

#### Açıklama:
- **SUMMARIZE**: Belirtilen kolonlara göre gruplandırma yapar
- **SUM**: Freight değerlerini toplar
- **Gruplandırma**: ShipCountry bazında gruplama

#### Kullanım Alanları:
- Lojistik maliyet analizi
- Ülke bazlı operasyonel giderler
- Coğrafi performans değerlendirmesi

---

### 3. Müşteri Bazında İşlem Sayısı (Top 10)

#### Soru: Her müşteri ID'ye göre işlem sayısı tablosu oluşturun (orders)

```dax
musteriye_gore_islem_say = TOPN(
    10, 
    SUMMARIZE(
        orders, 
        orders[CustomerID], 
        "islem sayisi", COUNT(orders[CustomerID])
    ), 
    [islem sayisi], 
    DESC
)
```

#### Açıklama:
- **TOPN**: En yüksek N değeri getirir
- **COUNT**: Belirtilen kolondaki kayıt sayısını hesaplar
- **DESC**: Azalan sırala (en çok işlem yapandan başlayarak)
- **Kombinasyon**: SUMMARIZE + TOPN birlikte kullanımı

#### Kullanım Alanları:
- VIP müşteri analizi
- Müşteri segmentasyonu
- Satış performansı değerlendirmesi

---

### 4. Tarih Fonksiyonları ve String Birleştirme

#### Problematik Kod:
```dax
yil_ay = ENDOFYEAR(orders[OrderDate]) & "-" & orders[OrderDate]  // ❌ Hatalı
```

#### Doğru Yaklaşımlar:

**Yıl-Ay Formatı Oluşturma:**
```dax
yil_ay = YEAR(orders[OrderDate]) & "-" & FORMAT(orders[OrderDate], "MM")
```

**Yıl Sonu Tarihi ile Kombinasyon:**
```dax
yil_sonu_formatli = FORMAT(ENDOFYEAR(orders[OrderDate]), "YYYY") & 
                   "-" & 
                   FORMAT(orders[OrderDate], "MM")
```

**Alternatif Tarih Formatları:**
```dax
// Yıl-Ay (2024-01)
tarih_yil_ay = FORMAT(orders[OrderDate], "YYYY-MM")

// Tam Tarih (2024-01-15)
tarih_tam = FORMAT(orders[OrderDate], "YYYY-MM-DD")

// Türkçe Format (Ocak 2024)
tarih_turkce = FORMAT(orders[OrderDate], "MMMM YYYY")
```

## Fonksiyon Detayları

### CALCULATE Fonksiyonu
```dax
CALCULATE(<expression>, <filter1>, <filter2>, ...)
```
- **Amaç**: Filter context'i değiştirerek hesaplama yapar
- **Avantajları**: Dinamik filtreleme imkanı
- **Dikkat**: AND mantığı ile çalışır

### SUMMARIZE Fonksiyonu
```dax
SUMMARIZE(<table>, <groupBy_columnName>, [<name>, <expression>])
```
- **Amaç**: Veri gruplandırma ve özetleme
- **Kullanım**: Pivot tablo benzeri işlemler
- **Not**: Sadece tablo görünümde kullanılabilir

### TOPN Fonksiyonu
```dax
TOPN(<n_value>, <table>, <orderBy_expression>, [<order>])
```
- **Amaç**: En üst N kaydı getirme
- **Sıralama**: ASC (artan) veya DESC (azalan)
- **Kombinasyon**: SUMMARIZE ile sık kullanılır

### COUNTA vs COUNT
- **COUNTA**: Boş olmayan tüm değerleri sayar
- **COUNT**: Sadece sayısal değerleri sayar
- **COUNTROWS**: Satır sayısını getirir

### Tarih Fonksiyonları
- **ENDOFYEAR**: Yılın son günü
- **YEAR**: Yıl değeri
- **MONTH**: Ay değeri
- **FORMAT**: Tarih formatlanması

## Uygulama Örnekleri

### 1. Gelişmiş Kategori Analizi
```dax
// Tüm kategorilerde ürün sayıları
kategori_urun_sayilari = SUMMARIZE(
    products,
    categories[CategoryName],
    "Ürün Sayısı", COUNTA(products[ProductID]),
    "Ortalama Fiyat", AVERAGE(products[UnitPrice])
)

// En çok ürünü olan kategori
en_cok_urunlu_kategori = TOPN(
    1,
    [kategori_urun_sayilari],
    [Ürün Sayısı],
    DESC
)
```

### 2. Müşteri Segmentasyonu
```dax
// Müşteri aktivite seviyesi
musteri_aktivite = SUMMARIZE(
    orders,
    orders[CustomerID],
    "Toplam Sipariş", COUNT(orders[OrderID]),
    "Toplam Tutar", SUM(orders[Freight]),
    "Son Sipariş", MAX(orders[OrderDate])
)

// Aktif müşteriler (Son 90 günde sipariş verenler)
aktif_musteriler = FILTER(
    [musteri_aktivite],
    [Son Sipariş] >= TODAY() - 90
)
```

### 3. Zaman Bazlı Analizler
```dax
// Aylık sipariş trendi
aylik_trend = SUMMARIZE(
    orders,
    "Yıl", YEAR(orders[OrderDate]),
    "Ay", MONTH(orders[OrderDate]),
    "Sipariş Sayısı", COUNT(orders[OrderID]),
    "Toplam Freight", SUM(orders[Freight])
)

// Yıllık büyüme oranı
yillik_buyume = DIVIDE(
    CALCULATE([Toplam Freight], YEAR(orders[OrderDate]) = 1998),
    CALCULATE([Toplam Freight], YEAR(orders[OrderDate]) = 1997)
) - 1
```

## Best Practices

### 1. CALCULATE Kullanımı
- Filtre koşullarını açık ve net yazın
- Birden fazla filtre kullanırken AND mantığını unutmayın
- Performance için basit filtreler tercih edin

### 2. SUMMARIZE Optimizasyonu
- Gereksiz kolonları gruplamaya eklemeyin
- Büyük tablolarda dikkatli kullanın
- Mümkünse önceden filtrelenmiş tablolar kullanın

### 3. TOPN Stratejileri
- N değerini makul tutun (genelde 10-50 arası)
- ORDER BY expression'ını optimize edin
- Büyük veri setlerinde performansa dikkat edin

### 4. Tarih İşlemleri
- FORMAT fonksiyonunu dikkatli kullanın
- String concatenation'da veri tipine dikkat edin
- Tarih tablosu kullanımını tercih edin

## Hata Ayıklama İpuçları

### Yaygın Hatalar
1. **Tarih + String birleştirme**: Veri tipi uyumsuzlukları
2. **SUMMARIZE syntax**: Kolon isimleri ve expression'lar
3. **TOPN ordering**: DESC/ASC kullanımı
4. **CALCULATE filtreler**: Boolean expression'lar


