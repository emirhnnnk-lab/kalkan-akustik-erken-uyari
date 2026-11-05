# Kalkan — FPV Drone Akustik Erken Uyarı Sistemi

Küçük FPV dronları **sesinden** erken tespit eden, düşük maliyetli sensör ağı Ar-Ge'si. Ham sesten başlayıp sinyal işleme, öğrenen model, ağ oylaması ve mikrodenetleyiciye gömülecek C koduna kadar uçtan uca bir zincir.

> **Savunma amaçlıdır.** Sistemin tek işlevi erken uyarıdır; hiçbir sürümde bir canlıya zarar verecek bir işlev yoktur. Bkz. [ETIK_ILKE.md](ETIK_ILKE.md).

▶️ **Canlı demo:** `index.html` dosyasını tarayıcıda açın (senaryo simülasyonu, vizör, ağ görünümü, spektrogram "kulak", Monte Carlo).

## Ne yaptım
1. **Gerçek veriyle sınadım.** Sentetik sese göre tasarlanan basit harmonik dedektör, gerçek kayıtlarda neredeyse rastgeleydi (AUC 0,54).
2. **Sinyal işlemeyi düzelttim.** Spektral beyazlatma ve kare kare tarak skoru ile görülmemiş kayıtlarda **AUC 0,89**.
3. **Küçük öğrenen model** (log-mel özellikleri + lojistik regresyon) ve **mikrodenetleyici için C'ye aktarım**.
4. **Ağ katmanı:** 4 düğümlü k/n oylama, düğüm/paket kaybı ve geometri analizi, darbe gürültüsü ayıklama.
5. **Akustik "looming":** sesin büyüme hızından çarpma süresi tahmini.

## Sonuçlar (dürüst tablo)
| Deney | Sonuç |
|---|---|
| Basit harmonik tarak, 160 gerçek kayıt | AUC **0,54** |
| Beyazlatma + kare kare tarak, ayrık test seti | AUC **0,89**, tespit %88, yanlış alarm %22 |
| Başka veri seti (açık alan), eşik değişmeden | AUC 0,77 — eşik taşınmıyor, saha kalibrasyonu gerekiyor |
| 30 sn ortam kaydıyla saha kalibrasyonu | Yanlış alarm yarıya iniyor, tespit aynı |
| Darbe gürültüsü ayıklama | Yanlış alarm %22 → %15 |
| 2/4 ağ oylaması (bağımsız gürültü) | Tespit %98,7, yanlış alarm %4,3 |
| Veri artırma, çeşitli dron verisi eklemek | Genellemeye yardım etmedi (olumsuz sonuç da kayıtta) |

Olumsuz sonuçları da bilerek tuttum: projenin en değerli dersi, sentetik varsayımı **erken** gerçek veriyle sınamaktı.

## Yapı
```
index.html   Tarayıcıda çalışan interaktif demo
sonuclar/    Deneylerin JSON çıktıları (28 deney)
ETIK_ILKE.md Projenin değişmez etik ilkesi
```
> Deney kodları, model ağırlıkları ve firmware **özel** tutulmaktadır; teknik görüşmelerde gösterilebilir.
Ham ses kayıtları açık veri setlerinden (DroneDetectionThesis, DADS) alınmıştır ve boyut nedeniyle repoda yoktur.

## Teknoloji
Python · numpy · scipy · scikit-learn · STFT / log-mel · C · Canvas JS · Gazebo / PX4 (simülasyon)

---
Geliştiren: **Emirhan Karaca**
