# Çöküş Tespit Sistemi Kullanım Kılavuzu
# Crash Detection System Usage Guide

## 🎯 Amaç / Purpose

Bu sistem, programda meydana gelen her türlü çöküşü tespit eder, detaylı bilgileri toplar ve kullanıcıya net bir şekilde gösterir.

This system detects all types of crashes in the program, collects detailed information, and clearly shows it to the user.

---

## 🔍 Özellikler / Features

### 1. Otomatik Çöküş Yakalama / Automatic Crash Catching

**TÜM yakalanmamış hatalar otomatik yakalanır:**
- Python istisnaları (exceptions)
- Thread hataları
- Tkinter callback hataları
- Dosya işlemi hataları
- API hataları
- Her türlü beklenmeyen hata

**ALL unhandled errors are automatically caught:**
- Python exceptions
- Thread errors
- Tkinter callback errors
- File operation errors
- API errors
- Any unexpected error

### 2. Detaylı Çöküş Raporları / Detailed Crash Reports

Her çöküş için otomatik olarak oluşturulur:
`crash_report_2026-02-10_05-17-47.txt`

**Raporda yer alan bilgiler:**

```
================================================================================
ÇÖKÜŞ RAPORU / CRASH REPORT #1
================================================================================
Tarih/Date: 2026-02-10 05:17:47.123456
Hata Tipi/Error Type: AttributeError
Hata Mesajı/Error Message: 'NoneType' object has no attribute 'configure'

TAM TRACEBACK / FULL TRACEBACK:
--------------------------------------------------------------------------------
[Full Python traceback here]

THREAD BİLGİSİ / THREAD INFORMATION:
--------------------------------------------------------------------------------
Ana Thread / Main Thread: MainThread
Mevcut Thread / Current Thread: AnalysisThread
Thread ID: 12345
Aktif Thread Sayısı: 3
Tüm Aktif Threadler:
  - MainThread (ID: 1, Alive: True)
  - AnalysisThread (ID: 12345, Alive: True)
  - Thread-3 (ID: 67890, Alive: True)

ÇÖKÜŞ NOKTASI DEĞİŞKENLER / CRASH POINT VARIABLES:
--------------------------------------------------------------------------------
Fonksiyon / Function: update_ui
Dosya / File: GÜNCEL
Satır / Line: 2345
Yerel Değişkenler:
  widget = None
  text = "Analyzing..."
  df = <DataFrame with 1000 rows>

GLOBAL DURUM / GLOBAL STATE:
--------------------------------------------------------------------------------
analysis_thread_running: True
_app_shutting_down: False

BELLEK BİLGİSİ / MEMORY INFO:
--------------------------------------------------------------------------------
Garbage Collector: True
GC Counts: (234, 10, 5)
================================================================================
```

### 3. Görsel Çöküş Diyalogu / Visual Crash Dialog

Program çöktüğünde otomatik popup açılır:

**🔴 PROGRAM ÇÖKTÜ / APPLICATION CRASHED**

**Gösterir:**
- Hata tipi ve mesaj
- Dosya adı ve satır numarası
- Fonksiyon adı
- Tam traceback
- Crash rapor dosyası

**Butonlar:**
- 📄 **Raporu Aç** - Crash rapor dosyasını açar
- 📋 **Kopyala** - Hata detaylarını panoya kopyalar
- ❌ **Kapat** - Dialogu kapatır

### 4. Debug Modu / Debug Mode

**Açmak için / To enable:**

GÜNCEL dosyasında ~316. satırda:
```python
DEBUG_MODE = True   # Detaylı loglama
```

**Kapatmak için / To disable:**
```python
DEBUG_MODE = False  # Normal mod
```

**Debug çıktısı örneği:**
```
[DEBUG 05:17:47.123] [MainThread] [start_analysis_threaded] >>> Fonksiyon başladı
[DEBUG 05:17:47.234] [AnalysisThread] [analiz_et_with_options] Dosyalar okunuyor...
[DEBUG 05:17:48.345] [AnalysisThread] [analiz_et_with_options] 5000 satır yüklendi
[DEBUG 05:17:48.456] [AnalysisThread] [analiz_et_with_options] Analiz tamamlandı
```

**Format:**
```
[DEBUG timestamp] [ThreadName] [Location] Message
```

---

## 📖 Kullanım Senaryoları / Usage Scenarios

### Senaryo 1: Program Çöküyor

**Problem:** Program analiz sırasında çöküyor, neden bilmiyorum.

**Çözüm:**
1. ✅ Programı çalıştır
2. ✅ Çöküşü tetikle (örn: analiz başlat)
3. ✅ Popup açılır → Hata detaylarını göster
4. ✅ "Raporu Aç" butonuna tıkla
5. ✅ Crash report dosyasını oku
6. ✅ Satır numarasını ve hata mesajını gör
7. ✅ Sorunu anla ve çöz

### Senaryo 2: Hangi Fonksiyon Çöküyor?

**Problem:** Program bazen çöküyor ama hangi fonksiyonda bilmiyorum.

**Çözüm:**
1. ✅ DEBUG_MODE = True yap
2. ✅ Programı çalıştır
3. ✅ Konsolu izle
4. ✅ Her fonksiyon çağrısını göreceksin:
   ```
   [DEBUG] >>> start_analysis_threaded ÇAĞRILDI
   [DEBUG] >>> analiz_et_with_options BAŞLADI
   [DEBUG] Dosyalar okunuyor...
   [CRASH] !!! HATA: AttributeError...
   ```
5. ✅ Son başarılı mesaja bak
6. ✅ Sonraki adımda çöküş oldu demektir

### Senaryo 3: Thread Sorunu

**Problem:** Threading ile ilgili bir sorun var ama hangisi?

**Çözüm:**
1. ✅ Program çöktüğünde crash report aç
2. ✅ "THREAD BİLGİSİ" bölümünü oku
3. ✅ Hangi thread'in çöktüğünü gör
4. ✅ O thread'in işlemlerini incele

### Senaryo 4: Değişken Değerleri

**Problem:** Çöküş anında değişkenlerin değerlerini görmek istiyorum.

**Çözüm:**
1. ✅ Crash report aç
2. ✅ "ÇÖKÜŞ NOKTASI DEĞİŞKENLER" bölümünü oku
3. ✅ Tüm local değişkenlerin değerlerini gör
4. ✅ None, boş veya yanlış değerleri tespit et

---

## 🛠️ Troubleshooting

### Crash Raporu Oluşmuyor

**Sebep:** Dosya yazma izni yok
**Çözüm:** Program klasörüne yazma izni ver

### Popup Açılmıyor

**Sebep:** Tkinter initialized değil veya başka bir hata
**Çözüm:** Konsolu kontrol et, crash report dosyasında detay var

### Debug Mesajları Çok Fazla

**Sebep:** DEBUG_MODE = True
**Çözüm:** DEBUG_MODE = False yap (sadece crash raporları kalır)

### Eski Crash Raporlarını Temizle

```bash
# Windows PowerShell:
Remove-Item crash_report_*.txt

# Linux/Mac:
rm crash_report_*.txt
```

---

## 📊 Crash Report Analiz İpuçları

### 1. Önce Hata Mesajını Oku
```
Hata Mesajı/Error Message: 'NoneType' object has no attribute 'configure'
```
→ Bir widget None (yok)

### 2. Satır Numarasına Git
```
Satır / Line: 2345
```
→ GÜNCEL dosyasının 2345. satırına git

### 3. Fonksiyonu İncele
```
Fonksiyon / Function: update_ui
```
→ update_ui fonksiyonu sorunlu

### 4. Thread'i Kontrol Et
```
Mevcut Thread / Current Thread: AnalysisThread
```
→ AnalysisThread çöktü (background thread)

### 5. Değişkenleri İncele
```
widget = None
```
→ widget None, o yüzden configure çağrısı başarısız

### 6. Global Durumu Kontrol Et
```
analysis_thread_running: True
_app_shutting_down: False
```
→ Analiz çalışıyor, shutdown değil

---

## ✅ En İyi Pratikler / Best Practices

### 1. DEBUG_MODE Kullanımı

**Development (Geliştirme):**
```python
DEBUG_MODE = True  # Detaylı loglama
```

**Production (Canlı):**
```python
DEBUG_MODE = False  # Sadece crash raporları
```

### 2. Crash Raporlarını Sakla

- ✅ Crash raporlarını sil değil, sakla
- ✅ Tekrar eden sorunları tespit et
- ✅ Pattern'leri gör

### 3. Hızlı Debug

Crash olduğunda:
1. Popup'ta "Kopyala" butonuna tıkla
2. Hata detaylarını yapıştır
3. Satır numarasına git
4. Sorunu çöz

### 4. Thread Safety

Crash raporunda thread bilgisini kontrol et:
- MainThread → UI işlemleri
- AnalysisThread → Analiz işlemleri
- Diğer → Background işlemler

---

## 📞 Destek / Support

Crash raporunu analiz edemiyorsanız:

1. ✅ Crash report dosyasını sakla
2. ✅ Konsol çıktısını kopyala
3. ✅ Hangi işlemi yaptığınızı not et
4. ✅ Tüm bilgileri developer'a gönder

---

## 🎯 Özet / Summary

**Artık her crash için:**
- ✅ Tam traceback
- ✅ Satır numarası
- ✅ Fonksiyon adı
- ✅ Hata mesajı
- ✅ Thread bilgisi
- ✅ Değişken değerleri
- ✅ Otomatik rapor

**GİZEM YOK! / NO MYSTERY!**

Her crash net ve anlaşılır şekilde raporlanır.
Every crash is clearly and comprehensibly reported.
