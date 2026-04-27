# NX Nastran Kalınlık Optimizasyon Aracı
### Sunum Taslağı — Giriş Seviye Mühendisler İçin

---

## Slayt 1 — Kapak

**Başlık:** NX Nastran Kalınlık Optimizasyon Aracı
**Alt başlık:** PSHELL Kalınlıklarını Otomatik Optimize Et — Kütle Minimizasyonu

> **Konuşma notu:** Bu araç, bir yapının PSHELL eleman kalınlıklarını otomatik olarak
> optimize ederek deplasman ve gerilme kısıtlarını karşılayan en hafif tasarımı buluyor.

---

## Slayt 2 — Problem: Neden Optimizasyon?

**Başlık:** Neden El ile Boyutlandırma Yetmez?

- Her PSHELL kalınlığını manuel denemek → onlarca/yüzlerce Nastran çalışması
- Mühendis zamanı pahalı, iterasyon süreci yorucu
- "Yeterince iyi" ≠ "En hafif geçerli tasarım"
- 341 PSHELL için tüm kombinasyonları elle denemek imkânsız

> **Görsel öneri:** Kalın yapı → ince yapı → kısıt sınırında optimum tasarım şeması
>
> **Konuşma notu:** Amacımız kısıtları tam olarak kullanan, gereksiz malzeme içermeyen
> tasarıma ulaşmak. Araç bunu otomatik yapıyor.

---

## Slayt 3 — Araç Ne Yapıyor?

**Başlık:** Aracın Görevi

- **Girdi:** BDF dosyası + izin verilen gerilme/deplasman değerleri
- **İşlem:** Nastran çalışmalarını otomatik, paralel koşturarak kalınlıkları dener
- **Çıktı:** En hafif geçerli kalınlık dağılımı + optimize edilmiş BDF

> **Görsel öneri:** BDF → [Optimizasyon Aracı] → Optimize BDF + Kütle/Deplasman Grafikleri
>
> **Konuşma notu:** Nastran'ı kendin çalıştırmana gerek yok.
> Araç arka planda tüm iterasyonları halleder.

---

## Slayt 4 — Nasıl Çalışır? (Adım Adım)

**Başlık:** Çalışma Akışı

1. BDF dosyasını seç → araç PSHELL property'lerini otomatik okur
2. Allowable Excel'ini gir (eleman bazlı gerilme limitleri, MPa)
3. Deplasman limitini ve yönünü belirle (örn. 11.7 mm, Mag)
4. Min/Max kalınlık ve adım boyutunu gir (örn. 0.5 – 5.0 mm, adım 0.1)
5. Algoritma ve iterasyon sayısını seç
6. **"Optimizasyonu Başlat"** → araç Nastran'ı otomatik koşturur
7. Sonuç: optimize BDF + kütle/deplasman grafikleri + log dosyaları

> **Konuşma notu:** Tüm Nastran çalışmaları araç tarafından yönetilir.
> Paralel mod aktifse aynı anda birden fazla çalışma koşturulur.

---

## Slayt 5 — Girdiler

**Başlık:** Neye İhtiyacın Var?

| Girdi | Açıklama |
|-------|----------|
| BDF Dosyası | PSHELL elemanları içeren Nastran deck |
| NX Nastran | `nastran.exe` dosya yolu |
| Çıktı Klasörü | Sonuçların kaydedileceği klasör |
| Allowable Excel | Her PSHELL için izin verilen Von Mises gerilmesi (MPa) |
| Max Deplasman | Yapısal kısıt (mm) |
| Min T / Max T / Step | Kalınlık aralığı ve adım boyutu (mm) |
| Paralel Run | Aynı anda koşturulan Nastran sayısı |

> **Önemli:** BDF dosyasında `PARAM,POST,-1` satırı bulunmalı —
> aksi hâlde Nastran OP2 çıktısı üretmez ve araç sonuç okuyamaz.

---

## Slayt 6 — Algoritmalar

**Başlık:** Hangi Algoritmayı Seçmeli?

| Algoritma | En İyi Kullanım | Özellik |
|-----------|----------------|---------|
| **OC + sep-CMA-ES** ⭐ | Büyük modeller (Önerilen) | Hibrit, paralel, güçlü |
| OC Standalone | Hızlı ilk sonuç, tek lisans | Seri, hızlı yakınsama |
| L-SHADE | Büyük modeller | Adaptif evrimsel, paralel |
| PSO | Orta büyüklükte modeller | Sürü tabanlı, paralel |
| Genetic Algorithm | Orta büyüklükte modeller | Popülasyon tabanlı |
| FD + SQP | Küçük modeller (az PSHELL) | Gradyan tabanlı |

> **Konuşma notu:** Hangi algoritmayı seçeceğinden emin değilsen
> **OC + sep-CMA-ES** ile başla. Araç, seçilen algoritmaya göre
> önerilen iterasyon sayısını otomatik gösterir.

---

## Slayt 7 — Temel Parametreler

**Başlık:** Parametreleri Nasıl Ayarlarsın?

- **Paralel Run:** Aynı anda çalışan Nastran sayısı
  → Sahip olduğun lisans sayısını geçme
- **Max İter:** Toplam deneme sayısı
  → Araç PSHELL sayısına göre önerilen değeri gösterir
- **Yakınsama Sabrı:** Kaç iterasyonda iyileşme olmazsa dursun
  → Önerilen: Max İter / 10 (örn. 300 iter → 30 sabır)
- **Surrogate Kullan:** L-SHADE/PSO'da bazı adımlar Nastran yerine
  matematiksel tahminle geçilir → süre kısalır, kalite korunur

> **Görsel öneri:** GUI ekran görüntüsü, parametreler işaretli ok ile

---

## Slayt 8 — Çıktılar

**Başlık:** Ne Elde Edersin?

- **Optimize BDF:** Doğrudan Nastran'da doğrulama için hazır model
- **optimized_thicknesses.csv:** Her PSHELL'in nihai kalınlığı
- **Kütle vs İterasyon grafiği:** Optimizasyon ilerlemesini gösterir (azalmalı)
- **Deplasman vs İterasyon grafiği:** Kısıtın nasıl kullanıldığını gösterir
- **optimization_log.txt:** Her adımın detaylı kaydı
- **iteration_results.txt:** Kütle / deplasman / stres özet tablosu
- **pareto_front.csv:** Kütle–deplasman açısından en iyi çözümler seti

> **Konuşma notu:** Pareto cephesi, farklı deplasman limitlerinde
> ulaşılabilecek minimum kütleleri gösterir — tasarım uzayını anlamak için kullanışlı.

---

## Slayt 9 — Örnek Sonuç

**Başlık:** Gerçek Çalışma Örneği

**Model:** 341 PSHELL, 2 yük durumu (Statik)
**Algoritma:** L-SHADE, 50 Paralel Run, 300 Nesil

| | Başlangıç | Optimum |
|--|-----------|---------|
| **Kütle** | ~0.0191 kg | ~0.0167 kg |
| **Max Deplasman** | 9.3 mm | 11.7 mm |
| **Gerilme** | OK | OK |

**Sonuç: %12.5 kütle azalması, kısıtlar tam kullanıldı**

> **Görsel öneri:** Kütle vs İterasyon grafiği (aşağı inen mavi eğri)
> ve Deplasman vs İterasyon grafiği (limite dayanan kırmızı eğri)

---

## Slayt 10 — Sık Karşılaşılan Sorunlar

**Başlık:** Dikkat Edilecekler

| Sorun | Sebep | Çözüm |
|-------|-------|-------|
| "OP2 bulunamadı" | BDF'de `PARAM,POST,-1` yok | BDF'e ekle |
| Nastran yavaş | Scratch klasörü ağ diskinde | Yerel diske (C: / D:) taşı |
| `[Lisans Doldu]` | Paralel Run > lisans sayısı | Paralel Run'u azalt |
| Erken durdu | Yakınsama Sabrı çok kısa | Max İter / 10 kullan |
| `[Lisans Bulunamadı]` | Lisans sunucusuna bağlanamıyor | Ağ bağlantısını kontrol et |

> **Konuşma notu:** Log dosyasındaki köşeli parantez mesajları
> (`[Lisans Doldu]`, `[F06 Hatası]` vb.) sorunu doğrudan işaret eder.

---

*Sunum taslağı — her slaydı PowerPoint'e kopyala, görselleri ve ekran görüntülerini ekle.*
