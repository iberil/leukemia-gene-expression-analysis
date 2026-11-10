# Akut Lösemide ALL/AML Sınıflandırması: Platformlar Arası Genelleştirme ve Sızıntısız Makine Öğrenmesi

Mikrodizi gen ifade verisiyle akut lenfoblastik lösemi (ALL) ve akut miyeloid lösemi (AML) ayrımı üzerine dönem projesi.
Bu depo **ara rapor** aşamasını içerir: üç veri setinin hazırlanması ve keşifsel veri analizi (EDA).
Makine öğrenmesi modelleri final aşamasında eklenecektir.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/iberil/leukemia-gene-expression-analysis/blob/main/Golub_ALL_AML_EDA_and_Preprocessing.ipynb)

## Proje kapsamı

Veriler p ≫ n yapısındadır (ör. Golub: 72 örnek, 7.129 prob). Çalışma üç soruyu hedefler:

1. Üç farklı Affymetrix platformundaki veri seti ortak bir gen uzayında nasıl karşılaştırılır?
2. Platform (batch) kayması sınıf sinyaline göre ne kadar baskındır?
3. Ön işleme, öznitelik seçimi ve doğrulama adımları sızıntısız nasıl kurulur? (Final aşaması)

## Veri setleri

| Veri seti | Platform | Örnek | Prob | Kaynak | Lisans |
|---|---|---|---|---|---|
| **Golub** (ana) | Affymetrix Hu6800 | 72 (47 ALL, 25 AML); eğitim 38, test 34 | 7.129 | `data/` klasörü (DLI Accelerated Data Science Teaching Kit, Modül 19 / Kaggle) | CC0 |
| **GSE131184** (benzer) | Affymetrix HG-U133 Plus 2.0 (GPL570) | 125 (49 T-ALL, 76 AML) | 54.675 | NCBI GEO, çalışma sırasında indirilir | GEO kullanım koşulları |
| **Armstrong / MLL** (bonus B4) | Affymetrix HG-U95Av2 | 72, MLL çıkarıldıktan sonra 52 (24 ALL, 28 AML) | 12.533 | `data/MLL.tab` (Biolab) | Biolab sayfası lisans belirtmemektedir |

Notlar:
- Üç veri setinin ortak gen sayısı **4.128**'dir (prob → HUGO sembolü eşlemesinde ilk sembol alınmıştır). MyGene.info canlı bir servis olduğundan eşleme sorgu tarihi (10 Kasım 2026) ile raporlanmıştır; farklı bir günde birkaç genlik fark çıkabilir (not defteri bu durumda uyarı basar).
- Golub'daki ALL sınıfı B- ve T-hücreli, GSE131184'teki ALL sınıfı yalnızca T-ALL'dir; üç veri setindeki ALL sınıfı klinik olarak aynı değildir.
- Armstrong'un AML örnekleri ile Golub arasında hasta düzeyinde örtüşme olasılığı vardır.

## Depo yapısı

```
.
├── Golub_ALL_AML_EDA_and_Preprocessing.ipynb   # tüm analiz (Hücre 1-8)
├── data/
│   ├── data_set_ALL_AML_train.csv              # Golub eğitim (38 örnek)
│   ├── data_set_ALL_AML_independent.csv        # Golub bağımsız test (34 örnek)
│   ├── actual.csv                              # Golub etiketleri (patient, cancer)
│   └── MLL.tab                                 # Armstrong/MLL (Biolab)
├── requirements.txt
└── README.md
```

## Not defterinin akışı

| Hücre | İçerik |
|---|---|
| 1 | Kurulum, import'lar, sabit tohum (42), sürüm bilgisi |
| 2 | Golub yükleme ve temizleme (call sütunları, 38/34 bölme, etiket eşleşmesi doğrulaması) |
| 3 | GSE131184 indirme, etiketleme, batch ve alt tip bilgisi |
| 4 | Armstrong/MLL yükleme, MLL örneklerinin çıkarılması |
| 5 | Prob → gen sembolü eşlemesi (MyGene.info, GPL570 tablosu), ortak gen kesişimi |
| 6 | Ön işleme (kırpma + log2), dağılım ve PCA şekilleri, ayırt edici genler, ısı haritası |
| 7 | Veri kalitesi tablosu, sınıf dağılımı, t-SNE, Golub eğitim/test karşılaştırması |
| 8 | GSE131184 için ayrı EDA (PCA, batch, alt tip, ısı haritası) ve Golub ile karşılaştırma |

Ön işleme kararları: Golub ve Armstrong'da değerler [20, 16000] aralığına kırpılıp log2 dönüşümü uygulanmıştır; GSE131184 yazarlarca RMA ile işlenmiş (log2 ölçeğinde) olduğundan dönüştürülmemiştir.

## Yeniden üretme

**Seçenek 1: Google Colab.** Yukarıdaki "Open In Colab" düğmesine basın ve hücreleri sırayla çalıştırın. Hücre 1 gerekli paketleri kurar.

**Seçenek 2: yerel kurulum.**

```bash
git clone https://github.com/iberil/leukemia-gene-expression-analysis.git
cd leukemia-gene-expression-analysis
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute Golub_ALL_AML_EDA_and_Preprocessing.ipynb
```

Gereksinimler:
- **İnternet bağlantısı.** Not defteri GSE131184'ü NCBI GEO'dan indirir (`GSE131184_family.soft.gz`), gen sembolü eşlemesi için MyGene.info'ya sorgu atar ve Golub/Armstrong dosyalarını bu deponun `data/` klasöründen ham GitHub bağlantısıyla okur.
- Hücreler **sırayla** çalıştırılmalıdır (sonraki hücreler önceki hücrelerin değişkenlerini kullanır).
- Çıktı olarak çalışma dizininde şu dosyalar üretilir: `fig_*.png` (300 dpi şekiller), `tablo_veri_kalitesi.csv`, `tablo_veri_setleri.csv`, `ortak_genler.csv` ve `map_*_prob_sembol.csv`.

**Tekrarlanabilirlik:** tohum 42 (`random`, `numpy`, `PCA`, `TSNE`). Kullanılan sürümler Python 3.13, numpy 2.4.6, pandas 2.2.3, SciPy 1.16.3, scikit-learn 1.6.1, matplotlib 3.10.0, seaborn 0.13.2, statsmodels 0.15.0 ve GEOparse 2.0.4'tür (`requirements.txt`).

## Ara rapordaki başlıca bulgular

- Düzeltme uygulanmamış birleşik veride platform kayması sınıf sinyalinden baskındır (PCA silhouette: platform 0,671, sınıf 0,076).
- Sınıflar arası anlamlı fark gösteren gen sayısı (Welch t-testi, BH-FDR < 0,05): Golub'da 662, GSE131184'te 1.776 (4.128 gen içinde).
- Golub eğitim ve test kümeleri arasındaki gen düzeyi farkın büyük bölümü sınıf bileşiminden kaynaklanıyor görünmektedir.
- İki veri setinde ortak anlamlı gen sayısı 369'dur; ilk 100 gende kesişim yalnızca 13'tür (ALL sınıfı klinik olarak aynı değildir).

Ayrıntılar ve şekiller için ara rapora bakınız.

## Final aşaması planı

Sızıntısız boru hattı (imputasyon, düşük varyans filtresi, z-skor, öznitelik seçimi ve sınıflandırıcı yalnızca eğitim katı içinde), üç yöntem (L1/Elastic Net lojistik regresyon, RBF çekirdekli SVM, rastgele orman; karar ağacı taban çizgisi) ve rehberin iki doğrulama protokolü (38/34 sabit bölme ve Golub'un 72 örneğinde tabakalı 5×5 tekrarlı iç içe çapraz doğrulama). Platform düzeltmesi (ComBat) gerekirse yalnızca eğitim katında kestirilecektir.

## Kaynaklar

- T. R. Golub vd., "Molecular classification of cancer," *Science*, 286(5439):531–537, 1999.
- S. A. Armstrong vd., "MLL translocations specify a distinct gene expression profile that distinguishes a unique leukemia," *Nat. Genet.*, 30(1):41–47, 2002.
- J. Bond vd., "A transcriptomic continuum of differentiation arrest identifies myeloid interface acute leukemias with poor prognosis," bioRxiv, 2019, doi:10.1101/2019.12.10.870121 (GSE131184).
- W. E. Johnson, C. Li ve A. Rabinovic, "Adjusting batch effects in microarray expression data using empirical Bayes methods," *Biostatistics*, 8(1):118–127, 2007 (ComBat).

Tam kaynakça ara raporda yer almaktadır.

## GenAI kullanım beyanı

Bu projede kod geliştirme, metin ve kaynakça düzenleme ile dil denetimi için Anthropic'in Claude modelinden yardımcı araç olarak yararlanılmıştır. Üretilen içerik yazar tarafından gözden geçirilmiş, kodlar çalıştırılmış ve sayılar çıktılarla karşılaştırılmıştır. Ayrıntılı beyan belgesi ara rapor teslim paketindedir.
