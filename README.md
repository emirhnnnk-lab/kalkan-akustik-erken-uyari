# Kalkan: FPV dronları sesinden ve böcek gözüyle erken fark etmek

> **EN summary:** R&D on a low-cost early-warning sensor network for small FPV drones (defensive, warning-only), with two senses: acoustic detection and insect-inspired vision circuits (STMD, T4/T5, LGMD, modeled and written in Verilog for FPGA). An honest results table: a naive harmonic detector was near-random on real recordings (AUC 0.54); spectral whitening and frame-wise comb scoring reached AUC 0.89 on held-out data, but only 0.77 on a different dataset, so field calibration is required. Full English version: [README.en.md](README.en.md).

Küçük FPV dronlar ucuz, hızlı ve radarda zor görünüyor. Ben şunu merak ettim: bunları **sesinden** ve **böcek gözüne benzeyen basit devrelerle**, düşük maliyetli sensörlerle önceden fark edebilir miyiz? Bu repo o Ar-Ge'nin kaydı: ham sesten başlayıp sinyal işleme, küçük bir öğrenen model, sensörler arası oylama ve mikrodenetleyiciye gömülecek C koduna kadar uzanan bir zincir.

> **Sadece savunma ve uyarı amaçlı.** Sistemin tek işi erken uyarmak; hiçbir sürümünde bir canlıya zarar verecek bir işlev yok. Yakında bir canlı varsa, kimin olursa olsun, sistem durur. Bkz. [ETIK_ILKE.md](ETIK_ILKE.md).

**Canlı demo:** [emirhnnnk-lab.github.io/kalkan-akustik-erken-uyari](https://emirhnnnk-lab.github.io/kalkan-akustik-erken-uyari/). Senaryo simülasyonu, vizör, sensör ağı, spektrogram "kulak", böcek beyni ve Monte Carlo sekmeleri tarayıcıda çalışıyor.

## Ne yaptım
1. **Önce gerçek veriyle sınadım.** Yapay sese göre tasarladığım basit "harmonik tarak" dedektörü, gerçek kayıtlarda neredeyse yazı tura kadar başarılıydı (AUC 0,54). Bu benim için projenin en önemli anıydı: yapay varsayımım gerçekte çalışmıyordu.
2. **Sinyal işlemeyi düzelttim.** Sesin arka plan gürültüsünü dengeleyip (spektral beyazlatma) skoru tek parça yerine kare kare hesaplayınca, hiç görmediği kayıtlarda **AUC 0,89**'a çıktı.
3. **Küçük bir öğrenen model** kurdum (log-mel özellikleri + lojistik regresyon) ve onu **mikrodenetleyicide çalışacak şekilde C'ye** aktardım.
4. **Sensör ağı:** 4 sensörün oylaması (k/n), bir sensörün ya da paketin kaybolması, yerleşim geometrisi ve ani patlama seslerinin ayıklanması.
5. **Akustik "yaklaşma" tahmini:** sesin büyüme hızından çarpmaya kalan süreyi tahmin etmek.

## Sonuçlar (dürüst tablo)
| Deney | Sonuç |
|---|---|
| Basit harmonik tarak, 160 gerçek kayıt | AUC **0,54** |
| Beyazlatma + kare kare tarak, ayrı test seti | AUC **0,89**, tespit %88, yanlış alarm %22 |
| Başka bir veri seti (açık alan), eşik değişmeden | AUC 0,77; eşik bir yerden diğerine taşınmıyor, sahada ayar gerekiyor |
| 30 saniyelik ortam kaydıyla sahada ayar | Yanlış alarm yarıya iniyor, tespit aynı kalıyor |
| Ani patlama seslerini ayıklamak | Yanlış alarm %22 → %15 |
| 4 sensörden 2'sinin oyu (bağımsız gürültü) | Tespit %98,7, yanlış alarm %4,3 |
| Veriyi çoğaltmak, farklı dron sesleri eklemek | Genellemeye yardım etmedi (bu olumsuz sonucu da kayıtta tuttum) |

Olumsuz sonuçları bilerek tuttum. Bence projenin en değerli dersi şu: yapay veriye göre kurduğun varsayımı **mümkün olduğunca erken** gerçek veriyle sınamak.

## Böcek beyni: görmek için doğadan ödünç devreler
Küçük, hızlı ve kalabalık bir arka planda uçan bir şeyi çok az enerjiyle fark etmeyi en iyi yapan sistemler böcek beyinleri. Dört böcek devresini modelledim:

| Böcek | Devre | Kalkan'daki işi |
|---|---|---|
| Yusufçuk | STMD (küçük hedef hareket dedektörü) | Ağaçlık bir arka planda küçük, hareketli noktayı seçmek |
| Meyve sineği | T4/T5 (Hassenstein-Reichardt hareket devresi) | Kameranın kendi sarsıntısını dünyanın hareketinden ayırmak |
| Çekirge | LGMD (yaklaşan cisim nöronu) | Büyüyen görüntüden çarpmaya kalan süreyi hesaplamak: τ = 1 / d(ln θ)/dt |
| Ormia sineği | Mekanik olarak bağlı minik kulaklar | Birbirine çok yakın iki mikrofonla hassas ses yönü (henüz simüle edilmedi) |

- **Demoda:** "Böcek beyni" sekmesinde 80×60 piksellik yapay bir orman sahnesi, olay kamerası modeli, STMD, T4/T5 hareket akışı, yaklaşma hesabı ve 8 yön nöronunun ateşleme deseni tarayıcıda çalışıyor.
- **Donanıma doğru:** LGMD, STMD ve T4/T5 devrelerini FPGA için Verilog ile yazdım ve test tezgâhında denedim. Her piksel paralel işlenebiliyor.
  - T4/T5 temiz çalışıyor: sağa giden hareket +, sola giden −, duran 0.
  - **Dürüst sınır:** basit yusufçuk devresi, kameranın kendi hareketini küçük hedeften tam ayıramıyor (küçük hedef 202, kendi hareketi 167 puan). Temiz bir ayrım için daha gelişmiş sürümü (ESTMD) gerekiyor.
- Gazebo'da bir fizik ikizi kurdum: kameraya doğru uçan bir dron sahnesi. Kamera görüntüsünü böcek devrelerine bağlayan köprüyü henüz yazmadım.

## Yayında bulduğum bir hata (08.11)
Demodaki "Gerçek kayıtlarda canlı ölç" düğmesi benim bilgisayarımda çalışıyordu ama yayında hata veriyordu. Sebebi: düğme 360 gerçek ses kaydını okuyordu, ve o kayıtlar repoda yoktu. Veri setinin açık bir lisansı olmadığı için kayıtları dağıtamıyorum. Çözüm: her kaydın skorunu aynı kodla kendi bilgisayarımda hesapladım ve sadece sayıları yayınladım (`sonuclar/ikiz_skorlar.json`). Eşik ve AUC artık tarayıcıda bu sayılardan canlı hesaplanıyor ve sonuç yayınladığım rakamlarla birebir aynı: AUC 0,888, tespit %88, yanlış alarm %22.

## Yapı
```
index.html   Tarayıcıda çalışan etkileşimli demo
sonuclar/    Deneylerin JSON çıktıları
ETIK_ILKE.md Projenin değişmez etik ilkesi
```
> Deney kodları, model ağırlıkları ve firmware **özel** tutuluyor; teknik görüşmelerde gösterilebilir.
Ses kayıtları açık veri setlerinden geliyor: DroneAudioDataset (Al-Emadi vd. 2019, atıf istiyor, açık lisansı yok) ve DroneDetectionThesis (CC0). Lisans ve boyut nedeniyle repoda yoklar.

## Teknoloji
Python · numpy · scipy · scikit-learn · STFT / log-mel · C · Verilog (FPGA) · Canvas JS · Gazebo / PX4 (simülasyon)

---
Geliştiren: **Emirhan Karaca**
