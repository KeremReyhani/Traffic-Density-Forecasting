# Akıllı Trafik — İstanbul Trafik Yoğunluğu Tahmini

İstanbul'a ait trafik hız/araç sayısı verilerini hava durumu bilgisiyle birleştirerek; bölgesel (geohash bazlı) trafik yoğunluğunu klasik makine öğrenmesi, derin öğrenme, grafik sinir ağları (GNN) ve zaman serisi yöntemleriyle tahmin eden bir proje.

## Proje Hakkında

Proje, İstanbul'un farklı bölgelerinden (geohash ile kodlanmış konumlar) toplanan anlık trafik hızı ve araç sayısı verilerinden bir **yoğunluk endeksi (DENSITY)** türetip, bu endeksin gelecekteki değerini tahmin etmeyi amaçlıyor. Zaman periyotları (gece, sabah zirvesi, gündüz, akşam zirvesi, geç akşam) ve yağış durumu gibi değişkenler de modele dahil edilmiştir.

## Proje Yapısı

```
Akıllı Trafik/
├── Veri hazırlık/
│   ├── baslangic.ipynb                          # Ham trafik verisinin ilk işlenmesi
│   ├── havaverisi.ipynb                         # Hava durumu verisinin hazırlanması
│   ├── istanbul_traffic_2024_grouped.csv        # Gruplanmış ham trafik verisi
│   ├── istanbul_hava_durumu_saatlik_2024.csv    # Saatlik hava durumu verisi
│   └── trafik_hava_birlesik.csv / trafik_hava.csv  # Trafik + hava nihai veri seti
├── chat.py                       # Ham CSV'lerin okunup gruplanması, zaman periyodu özellik çıkarımı
├── Proje.ipynb                    # Ana veri işleme: yoğunluk endeksi, zaman/lag/rolling özellikleri
├── proje.csv                      # Modelleme için hazırlanmış nihai veri seti
├── Modeller/
│   ├── ml.ipynb                   # XGBoost, Random Forest, LightGBM ile tahmin
│   ├── deep.ipynb                 # PyTorch tabanlı derin öğrenme (zaman-mekan tensörü)
│   ├── graph.ipynb                # Geohash komşuluğuna dayalı Graph Neural Network (STGCN)
│   ├── time.ipynb                 # SARIMAX ve Prophet ile zaman serisi tahmini
│   ├── time2.py                   # Zaman serisi modelleri için yardımcı/alternatif betik
│   ├── hybrid.ipynb               # OOF stacking + meta-model ile hibrit tahmin
│   ├── hazırlık.ipynb              # Model öncesi ek veri hazırlığı
│   ├── stgcn_clean_outputs/        # Eğitilmiş STGCN model ağırlıkları
│   ├── hybrid_oof_outputs/         # Hibrit model final meta-modeli
│   └── DL_MULTI_FEATURE_RESULTS/   # Derin öğrenme modellerinin (LSTM, CNN-LSTM, Transformer) ağırlıkları
├── TRAFİK YOĞUNLUK TAHMİNLEMESİ.pptx   # Proje sunumu
├── 21052045_KeremReyhani.pdf            # Proje raporu
└── LICENSE                                # Proje lisansı
```

## Yöntem

### 1. Veri Hazırlama (`chat.py`, `Veri hazırlık/`, `Proje.ipynb`)
- Ham trafik CSV'lerinin (geohash, min/maks/ortalama hız, araç sayısı) okunup birleştirilmesi
- Geohash'lerin kısaltılması, saatlik verilerin zaman periyoduna göre gruplanması (Gece, Sabah Zirvesi, Gündüz, Akşam Zirvesi, Geç Akşam)
- Hız değerlerindeki Türkçe sayı formatının düzeltilmesi
- Hava durumu verisiyle (yağış durumu vb.) birleştirme
- **Yoğunluk Endeksi** türetilmesi: `DENSITY = NUMBER_OF_VEHICLES / AVERAGE_SPEED`
- Zaman özellikleri (haftanın günü, ay, hafta sonu mu), lag (1, 24, 168 saat) ve hareketli ortalama (rolling) özellikleri

### 2. Modelleme (`Modeller/` klasörü)
- **Klasik ML (`ml.ipynb`):** XGBoost, Random Forest, LightGBM ile geohash bazlı zamana dayalı train/test ayrımı
- **Derin Öğrenme (`deep.ipynb`):** Trafik verisinin (zaman, bölge, özellik) 3 boyutlu tensöre dönüştürülüp LSTM benzeri mimarilerle eğitilmesi
- **Grafik Sinir Ağı (`graph.ipynb`):** Geohash'ler arası coğrafi komşuluk ilişkisi (haversine mesafesi) kullanılarak STGCN (Spatio-Temporal Graph Convolutional Network) ile mekansal-zamansal tahmin
- **Zaman Serisi (`time.ipynb`, `time2.py`):** Her geohash için ayrı ayrı SARIMAX ve Prophet modelleri
- **Hibrit (`hybrid.ipynb`):** Farklı modellerin OOF (out-of-fold) tahminlerinin bir meta-model ile birleştirilmesi (stacking)

### 3. Değerlendirme
- MAE, RMSE, R² metrikleri
- Bölgeler (geohash) arası ve zaman periyotları arası karşılaştırmalı performans analizi

## Kullanılan Teknolojiler

- Python 3, pandas, numpy
- scikit-learn, XGBoost, LightGBM
- PyTorch (derin öğrenme ve GNN modelleri)
- statsmodels, Prophet (zaman serisi)
- geohash2, haversine (coğrafi hesaplamalar)
- joblib (model kaydetme)

## Kurulum ve Çalıştırma

Bu projeyi çalıştırmak için yukarıdaki kütüphanelerin (pandas, numpy, scikit-learn, xgboost, lightgbm, torch, statsmodels, prophet, geohash2, haversine, joblib) kendi Python ortamınıza kurulu olması gerekir.

Notebook'ları sırasıyla çalıştırın:

```bash
python chat.py                          # 1. Ham veriyi grupla
jupyter notebook Proje.ipynb            # 2. Özellik mühendisliği, proje.csv oluştur
jupyter notebook Modeller/ml.ipynb      # 3a. Klasik ML modelleri
jupyter notebook Modeller/deep.ipynb    # 3b. Derin öğrenme modeli
jupyter notebook Modeller/graph.ipynb   # 3c. Graph Neural Network (STGCN)
jupyter notebook Modeller/time.ipynb    # 3d. SARIMAX / Prophet
jupyter notebook Modeller/hybrid.ipynb  # 3e. Hibrit stacking modeli
```

## Çıktılar

- Bölgesel trafik yoğunluğu tahmin sonuçları
- Eğitilmiş model ağırlıkları (`stgcn_clean_outputs/`, `DL_MULTI_FEATURE_RESULTS/`)
- Hibrit modelin final meta-modeli (`hybrid_oof_outputs/meta_model.joblib`)
- Proje sunumu (`.pptx`) ve detaylı rapor (`.pdf`)

## Not

- Kod içerisindeki dosya yolları (`C:/Users/kerem/Desktop/...`) yerel makineye özeldir; projeyi çalıştırmadan önce kendi veri yolunuzla güncellemeniz gerekir.
- `proje.csv` yalnızca proje kök dizininde tutulmalıdır; `Modeller/` altında oluşan ikinci kopyayı repoya yüklemeden önce silmeniz önerilir.
- `hybrid_oof_outputs/` içindeki ara `.pth` checkpoint'lerinin (fold başına oluşan onlarca dosya) **tamamının repoya dahil edilmesi önerilmez**; sadece final `meta_model.joblib` dosyasının saklanması, ara checkpoint'lerin ise `.gitignore` ile hariç tutulması daha uygundur.
- `DL_MULTI_FEATURE_RESULTS/` altındaki büyük `.pth` model ağırlıkları için Git LFS önerilir.
- `proje.csv` büyük hacimli olduğundan (~90 MB) Git ile doğrudan takip edilmesi risklidir (GitHub'ın standart dosya boyutu sınırına yakın); veri harici olarak (Drive/Kaggle bağlantısı) paylaşılabilir veya Git LFS kullanılabilir.
- `.pth`/`.ckpt` gibi büyük model dosyalarını repoya yüklemeden önce `.gitignore` ile hariç tutmanız önerilir.
