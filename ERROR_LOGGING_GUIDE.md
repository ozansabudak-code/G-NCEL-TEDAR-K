# Hata Loglama ve Debug Rehberi
## Error Logging and Debugging Guide

Bu dokümanda, uygulamaya eklenen kapsamlı hata loglama sisteminin nasıl kullanılacağı açıklanmaktadır.

---

## 📋 Özellikler / Features

### 1. Otomatik Hata Loglama
- **Dosya:** `error_log_YYYY-MM-DD.log`
- **Lokasyon:** Uygulama ana dizini
- **Format:** Zaman damgası, thread bilgisi, kod konumu, hata detayı

### 2. Thread Güvenli İşlemler
- Tüm UI güncellemeleri `root.after()` ile yapılır
- Thread hatalarını yakalayan wrapper fonksiyonlar
- Uygulama kapanırken güvenli temizlik

### 3. Detaylı Hata Takibi
- Hangi thread'de hata oluştuğu
- Hangi satırda hata aldığınız
- Ne yapmaya çalışırken hata aldığınız
- Tam stack trace

---

## 🔧 Kullanım / Usage

### Hata Log Dosyasını Okuma

Her gün yeni bir log dosyası oluşturulur: `error_log_2026-02-09.log`

**Örnek Hata Kaydı:**
```
================================================================================
TIMESTAMP: 2026-02-09 12:45:23.456
THREAD: BackgroundThread (ID: 12345)
LOCATION: Line 2376
CONTEXT: update_market_ticker - label update
ERROR TYPE: RuntimeError
ERROR MESSAGE: main thread is not in main loop
TRACEBACK:
Traceback (most recent call last):
  File "/path/to/GÜNCEL", line 2376, in _fetch
    ticker_label.configure(text=ticker_text)
RuntimeError: main thread is not in main loop
================================================================================
```

### Hata Türlerini Anlama

#### 1. **RuntimeError: main thread is not in main loop**
**Ne Anlama Geliyor:** Bir thread, Tkinter widget'ını doğrudan güncellemeye çalışıyor.

**Nerede Çözüldü:** 
- `thread_safe_after()` fonksiyonu ile
- `safe_ui_call()` wrapper ile
- `run_in_thread()` içinde otomatik `root.after()` kullanımı ile

**Artık Olmamalı:** Bu hata artık loglanır ve düzeltilmiş olmalı.

#### 2. **Tcl_AsyncDelete**
**Ne Anlama Geliyor:** Uygulama kapanırken thread'ler hala çalışıyor.

**Nerede Çözüldü:**
- `_app_shutting_down` flag ile
- `on_closing()` fonksiyonunda 100ms gecikme ile
- Thread'lerde shutdown kontrolü ile

**Artık Olmamalı:** Temiz kapanış sağlandı.

#### 3. **AttributeError: 'NoneType' object has no attribute 'configure'**
**Ne Anlama Geliyor:** Widget destroy edilmiş ama thread hala erişmeye çalışıyor.

**Nerede Çözüldü:**
- Tüm widget erişimlerinde `try/except` blokları
- Widget varlık kontrolleri
- Güvenli çağrı wrapper'ları

---

## 💡 Geliştirici Notları / Developer Notes

### Yeni Thread Eklerken

```python
# ❌ YANLIŞ - Doğrudan UI güncelleme
def my_background_task():
    result = fetch_data()
    my_label.configure(text=result)  # HATA!

threading.Thread(target=my_background_task, daemon=True).start()

# ✅ DOĞRU - run_in_thread ile
def my_background_task():
    result = fetch_data()
    return result

def update_ui(result):
    my_label.configure(text=result)

run_in_thread(
    my_background_task,
    callback=update_ui,
    context="My task description",
    location="Line 1234"
)
```

### Manuel root.after() Kullanımı

```python
# ❌ YANLIŞ - Hata kontrolü yok
root.after(0, lambda: widget.configure(text="Hi"))

# ✅ DOĞRU - thread_safe_after kullan
thread_safe_after(
    root, 0,
    lambda: widget.configure(text="Hi"),
    context="Widget update",
    location="Line 1234"
)
```

### UI İşlemlerini Koruma

```python
# ✅ DOĞRU - safe_ui_call ile koru
safe_ui_call(
    lambda: complicated_ui_operation(),
    context="Complicated operation",
    location="Line 1234"
)
```

---

## 🐛 Hata Ayıklama Süreci / Debugging Process

### 1. Hatayı Gözlemle
- Console çıktısını oku
- Hangi işlem sırasında oluştu?

### 2. Log Dosyasını Kontrol Et
```bash
# En son hatayı gör
tail -50 error_log_2026-02-09.log

# Belirli bir hatayı ara
grep "RuntimeError" error_log_2026-02-09.log
```

### 3. Hata Konumunu Bul
- **LOCATION:** satırı hangi kod satırını gösterir
- **CONTEXT:** ne yapmaya çalışıyordu
- **THREAD:** hangi thread'de oluştu

### 4. Traceback'i İncele
- Stack trace tam yolu gösterir
- Her fonksiyon çağrısını takip et
- Hangi değişken None veya hatalı?

### 5. Düzelt ve Test Et
- Sorunu düzelt
- Aynı işlemi tekrarla
- Log dosyasını kontrol et

---

## 📊 Log Analizi / Log Analysis

### Sık Karşılaşılan Hatalar

```bash
# Thread hatalarını say
grep -c "RuntimeError" error_log_*.log

# Hangi fonksiyonlar en çok hata veriyor?
grep "CONTEXT:" error_log_*.log | sort | uniq -c | sort -rn | head -10

# Hangi thread'ler sorunlu?
grep "THREAD:" error_log_*.log | sort | uniq -c | sort -rn
```

### Performans İzleme

- Hata sayısı azalıyor mu?
- Aynı hatalar tekrar ediyor mu?
- Hangi saatlerde daha çok hata var?

---

## ✅ Test Senaryoları / Test Scenarios

### 1. Normal İşlem Testi
- Tüm sekmeleri aç
- Tüm butonları tıkla
- Hata olmamalı

### 2. Hızlı Kapanış Testi
- Uygulamayı aç
- Arka plan işlemi başlat (hammadde fiyat güncelleme)
- Hemen kapat
- Hata loglanmalı ama crash olmamalı

### 3. Çoklu Thread Testi
- Birden fazla analiz başlat
- Sayfa değiştir
- Tüm işlemler tamamlanmalı

### 4. Widget Destroy Testi
- Uzun işlem başlat
- Sekmeyi değiştir
- Eski widget'lar destroy olmalı
- Hata olmamalı veya loglanmalı

---

## 🎯 Başarı Kriterleri / Success Criteria

✅ Uygulama crash olmadan çalışıyor
✅ Tüm hatalar log dosyasına kaydediliyor
✅ Her hata için konum bilgisi var
✅ Thread'ler güvenli şekilde UI güncelliyor
✅ Uygulama temiz kapanıyor

---

## 📞 Destek / Support

Eğer hala hatalar yaşıyorsanız:

1. `error_log_YYYY-MM-DD.log` dosyasını kontrol edin
2. Hatanın tam çıktısını kopyalayın
3. Ne yaptığınızı (hangi butona bastınız, hangi işlem) açıklayın
4. Geliştiriciye iletin

**Dosya Konumu:** Uygulamanın çalıştığı klasör
**Format:** Tarihli log dosyaları (her gün yeni)
**İçerik:** Tam hata detayları ve stack trace

---

## 🔄 Güncelleme Geçmişi / Update History

### v1.0 (2026-02-09)
- İlk hata loglama sistemi eklendi
- `log_thread_error()` fonksiyonu
- `safe_ui_call()` wrapper
- `thread_safe_after()` fonksiyonu
- `run_in_thread()` güncellendi
- `update_market_ticker()` güncellendi
- `fetch_real_commodity_data()` güncellendi

### Gelecek Güncellemeler
- Daha fazla fonksiyona error logging eklenecek
- Otomatik hata raporlama
- Grafik hata dashboard'u
