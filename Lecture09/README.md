# Lecture 09 - SUMMARIZE Fonksiyonu ve Veri Modelleme


### Veri Modelleme Yaklaşımları
- **Snowflake Modeli**: Normalleştirilmiş yaklaşım
- **Star Şema Modeli**: Merkezi fact tablo etrafında dimension tabloları
- **Semantik Model**: Her iki modelin birlikte kullanımı

## SUMMARIZE Fonksiyonu

### Önemli Notlar
- ⚠️ **Sadece tablo görünümde yazılabilir**
- 💾 **SUMMARIZE tabloları ekstra yer kaplamaz**

### Pratik Örnekler

#### Q2: Kategori Bazında Ürün Sayıları
```dax
summarize2 = SUMMARIZE(
    products,
    categories[CategoryName],
    "Ürün Sayısı", COUNTROWS(products)
)
```

#### Q3: Yıllara Göre Sipariş Sayısı
```dax
summarize3 = SUMMARIZE(
    orders,
    tarih_tablosu[Date].[Year], 
    "Sipariş Sayısı", COUNTROWS(orders)
)
```

#### Q4: Ülke Bazında Freight Toplamları
**Temel Versiyon:**
```dax
summarize4 = SUMMARIZE(
    orders, 
    orders[ShipCountry], 
    "Freight Toplamı", SUM(orders[Freight])
)
```

**Filtrelenmiş Versiyon (>2000):**
```dax
summarize4 = FILTER(
    SUMMARIZE(
        orders, 
        orders[ShipCountry], 
        "Freight Toplamı", SUM(orders[Freight])
    ), 
    [Freight Toplamı] > 2000
)
```

#### Q5: Sipariş Bazında Ciro Hesaplama
**Temel Versiyon:**
```dax
summarize5 = SUMMARIZE(
    order_details, 
    order_details[OrderID], 
    "Ciro", SUMX(order_details, order_details[UnitPrice]*order_details[Quantity])
)
```

**Filtrelenmiş Versiyon (>10000):**
```dax
summarize5 = FILTER(
    SUMMARIZE(
        order_details, 
        order_details[OrderID], 
        "Ciro", SUMX(order_details, order_details[UnitPrice]*order_details[Quantity])
    ), 
    [Ciro] > 10000
)
```

## Dashboard Ölçüleri 👍

### 4 Temel Zaman Bazlı Ölçü

#### 1. Son 6 Ay Ciro
```dax
son6ayciro = CALCULATE(
    [toplam_ciro], 
    DATESINPERIOD(
        tarih_tablosu[Date],
        LASTDATE(tarih_tablosu[Date]), 
        -6, 
        MONTH
    )
)
```

#### 2. Son 7 Gün Ciro
```dax
son7gunciro = CALCULATE(
    [toplam_ciro], 
    DATESINPERIOD(
        tarih_tablosu[Date],
        LASTDATE(tarih_tablosu[Date]), 
        -7, 
        DAY
    )
)
```

#### 3. İlk 3 Ay Ciro
```dax
ilk3ayciro = CALCULATE(
    [toplam_ciro],
    DATESINPERIOD(
        tarih_tablosu[Date],
        FIRSTDATE(tarih_tablosu[Date]), 
        3, 
        MONTH
    )
)
```

#### 4. Belirli Tarih Aralığı (7-27 Haziran 1997)
```dax
yedionyediciro = CALCULATE(
    [toplam_ciro],
    DATESBETWEEN(
        tarih_tablosu[Date],
        DATE(1997,6,7),
        DATE(1997,6,27)
    )
)
```

## Veri Gruplama (Binning)

### Sayısal Grupların Gruplanması
- **Bins'lemek**: Sayısal grupların gruplanması olarak adlandırılır
- **Uygulama**: Data'da yer alan bir değeri grafiğe ekleyip "..." kısmından "Edit Group" diyerek gruplama yapılır
- **Amaç**: Görselleştirmeleri anlamlı şekle getirmek

### Edit Group Özelliği
1. Grafikteki bir değeri seçin
2. "..." (üç nokta) menüsüne tıklayın
3. "Edit Group" seçeneğini kullanın
4. Anlamlı gruplar oluşturun
