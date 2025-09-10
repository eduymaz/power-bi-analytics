# Lecture 06 - Power BI Tarih Fonksiyonları ve DAX Ölçüleri

**Tarih:** 24.08.2025

## Ders Başlangıcı Sorular

### Soru 1: Verilerimizi Power Query'e yükledikten sonra ne yapmalıyız?
**Cevap:**
- **Tablo ismi** düzenlenmelidir
- **Kolon ismi** düzenlenmelidir  
- **Veri ismi** düzenlenmelidir

### Soru 2: Verilerin yüklendiği yere ne denir?
**Cevap:**
- **Veri Modeli** kısmı denir (Power Query'den Power BI'a geçerken veri modeli adını alır)
- Veri Modelini aktardıktan hemen sonra yapılan ilk işlem:
  - Doğru model kuruldu mu kontrol edilmelidir
  - **İlişkiler Tablosu** incelenir
  - Analiz yaptığımız tabloda tarihsel analiz var ise, **harici tarih tablosu** oluşturulur

### Soru 3: Ölçüler için ayrı klasör nasıl oluşturulur?
**Cevap:**
- Entry Data ile ölçüler için ayrı klasör oluşturulur
- Kontrolü yönetebilmek adına kullanılır

## GÜNÜN KONUSU: Tarih Fonksiyonları ve DAX Ölçüleri

### Tarih Tablosu Oluşturma

#### CALENDARAUTO() Fonksiyonu
```dax
CALENDARAUTO()
```
- **Ne zaman kullanılır:** Sadece tek bir tabloda tarih bilgisi olacaksa kullanılmalıdır
- **Dikkat:** Başka tarih kolonları varsa CALENDAR kullanılmalıdır

#### CALENDAR() Fonksiyonu
```dax
tatih_tablosu = CALENDAR(FIRSTDATE(islemler1[islem_trh]),MAX(islemler1[islem_trh]))
```

### DAX Ölçüleri

#### SUMX Fonksiyonu
**Derste Soru:** Birden fazla kolonu toplayan çarpan fonksiyon?
**Cevap:** SUMX

```dax
toplam_ciro = SUMX(islemler1,islemler1[birim_fiyat]*islemler1[satis_adet])
```

#### CALCULATE Fonksiyonu - Kriterli Hesaplamalar
```dax
tl_ciro = CALCULATE([toplam_ciro],islemler1[doviz_cinsi]="TL")
usd_ciro = CALCULATE([toplam_ciro],islemler1[doviz_cinsi]="USD")
euro_ciro = CALCULATE([toplam_ciro],islemler1[doviz_cinsi]="EURO")
```

#### Tarih Bazlı Hesaplamalar

**Dünkü Ciro:**
```dax
dun_ciro = CALCULATE([tl_ciro],DATEADD(tatih_tablosu[Date], -1, DAY))
```

**Günlük Değişim:**
```dax
gunluk_degisim = ([tl_ciro]-[dun_ciro]) / [dun_ciro]
```

**Hata Kontrolü ile Günlük Değişim:**
```dax
gunluk_degisim = IFERROR(([tl_ciro]-[dun_ciro]) / [dun_ciro],BLANK())
```
> **Not:** Infinity veya error çıkan sonuçları kaldırır

**Geçen Ay Ciro:**
```dax
gecen_ay_ciro = CALCULATE([tl_ciro],DATEADD(tatih_tablosu[Date], -1,MONTH))
```

**Aylık Değişim:**
```dax
aylik_degisim = IFERROR(([tl_ciro]-[gecen_ay_ciro])/[gecen_ay_ciro], BLANK())
```

## Önemli Kavramlar

### 1. Veri Modeli Yönetimi
- **Power Query**: Veri temizleme ve dönüştürme aşaması
- **Veri Modeli**: Power Query'den Power BI'a geçiş aşaması
- **İlişkiler Tablosu**: Tablolar arası ilişkilerin kontrolü
- **Entry Data**: Ölçüler için klasör organizasyonu

### 2. Tarih Fonksiyonları
- **CALENDARAUTO()**: Tek tarih kolonu için otomatik tarih tablosu
- **CALENDAR()**: Özel tarih aralığı için tarih tablosu
- **DATEADD()**: Tarih bazlı hesaplamalar için
- **FIRSTDATE()** ve **MAX()**: Tarih aralığı belirleme

### 3. DAX Ölçü Fonksiyonları
- **SUMX()**: Satır bazlı hesaplamalar için
- **CALCULATE()**: Kriterli hesaplamalar için  
- **IFERROR()**: Hata kontrolü için
- **BLANK()**: Boş değer döndürme

### 4. Hesaplama Türleri
- **Toplam Hesaplamalar**: SUMX ile çarpım-toplam işlemleri
- **Kriterli Hesaplamalar**: CALCULATE ile filtrelenmiş hesaplamalar
- **Tarih Bazlı Karşılaştırmalar**: DATEADD ile dönemsel analiz
- **Hata Yönetimi**: IFERROR ile güvenli hesaplamalar

## Dosya Yapısı ve İçerikleri

### Ana Dosyalar
- **Lecture notes.docx**: Ders notları ve teorik açıklamalar
- **HOMEWORK06.docx**: 6. hafta ödev talimatları
- **odev.xlsx**: Ödev için kullanılacak Excel veri seti

### Power BI Dosyaları
- **Lec06-section1.pbix**: İlk bölüm uygulamaları
- **Lec06-section2.pbix**: İkinci bölüm uygulamaları

### Kaynak Klasörü İçerikleri
- **24082025_kaynak.pbix**: Ana kaynak Power BI dosyası
- **dash_db.xlsx**: Dashboard için veri tabanı Excel dosyası
- **BYMMB_POWERBI_TR_MAP.json**: Türkiye haritası coğrafi veri dosyası
