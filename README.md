#IoT Saldırı Tespiti: Hibrit CNN ve Random Forest Yaklaşımı

Bu proje, **CICIoT2023** veri setini kullanarak IoT ağlarındaki siber saldırıları
tespit etmeyi amaçlayan hibrit bir makine öğrenmesi modelini içermektedir.
Çalışmada ham ağ paketleri **görüntü temsiline dönüştürülmüş**, bu sayede
**derin öğrenme (CNN)** ve **geleneksel makine öğrenmesi (Random Forest)**
birlikte kullanılmıştır.

---

##Proje Özeti

- **Veri Seti:** [CICIoT2023](https://www.unb.ca/cic/datasets/iotdataset-2023.html)  
  (PCAP formatında gerçek IoT ağ trafiği)
- **Yaklaşım:**  
  *Byte-to-Image* (bayttan görüntüye) dönüşümü  
  + **CNN** ile otomatik özellik çıkarımı  
  + **Random Forest** ile sınıflandırma
- **Kullanılan Sınıflar:**
  - Benign (Normal trafik)
  - DDoS-SYN Flood
  - Mirai-udpplain
  - Recon-PortScan
- **En İyi Hibrit Model Başarımı (CNN + RF):**
  - **Accuracy:** ≈ **%95**
  - **Macro F1:** ≈ **%95**

---

##Metodoloji

Bu projede “**Trafikten Görüntüye**” (Traffic-to-Image) yaklaşımı benimsenmiştir:

1. **Ön İşleme (PCAP → Byte):**
   - PCAP dosyalarındaki her ağ paketi ham bayt dizisi olarak okunur.
   - Her paket, en fazla **784 bayt** olacak şekilde kırpılır veya sıfır dolgu (padding) ile tamamlanır.

2. **Görüntü Dönüşümü (Byte → Image):**
   - 784 elemanlı vektör, **28×28** boyutunda gri tonlamalı görüntüye dönüştürülür.
   - Piksel değerleri `[0, 255]` aralığından `[0, 1]` aralığına normalize edilir.

3. **Öznitelik Çıkarımı (CNN):**
   - 28×28×1 giriş görüntüleri, hafif bir **2B CNN mimarisine** verilir:
     - Conv2D (32 filtre, 3×3, ReLU) + MaxPooling (2×2)
     - Conv2D (64 filtre, 3×3, ReLU)
     - Flatten + Dense (128 nöron, ReLU)
   - Çıkış katmanından bir önceki **Flatten katmanı**, paket başına sabit boyutlu özellik vektörü (**Z**) olarak kullanılır.

4. **Sınıflandırma (Random Forest):**
   - CNN’in ürettiği özellik vektörleri **Random Forest** modeline girdi olarak verilir.
   - RF için hiperparametre araması **Grid Search** ile yapılır:
     - `n_estimators ∈ {50, 100}`
     - `max_depth ∈ {None, 20}`

5. **Değerlendirme:**
   - Eğitim/Test bölünmesi: **%80 eğitim / %20 test** (stratified split).
   - Metrikler: Accuracy, Precision, Recall, F1-Score, Confusion Matrix, sınıf bazlı ROC/AUC.

---

##Ana Sonuçlar (CNN + Random Forest)

CICIoT2023 veri seti üzerinde, **CNN özellik çıkarımı + Random Forest**
hibrit modelinin sınıf bazlı performansı:

| Sınıf      | Precision | Recall | F1-Score |
|-----------|-----------|--------|----------|
| **Benign**    | 0.95      | 0.94   | 0.95     |
| **SYN_Flood** | 0.99      | 0.94   | 0.96     |
| **Mirai**     | 0.99      | 0.96   | 0.97     |
| **PortScan**  | 0.88      | 0.96   | 0.92     |

**Genel Sonuçlar (CNN + RF, en iyi parametrelerle):**

- **Accuracy:** ≈ **0.95**  (~%95)
- **Macro Precision:** ≈ **0.95**
- **Macro Recall:** ≈ **0.95**
- **Macro F1-Score:** ≈ **0.95**

Bu değerler, hibrit modelin hem normal trafiği hem de farklı saldırı
türlerini yüksek doğrulukla ayırt ettiğini göstermektedir. Özellikle:

- **Mirai** ve **SYN Flood** için **F1-Score ≈ %96–97**  
- **PortScan** için yüksek **Recall (~%96)** → Port tarama saldırıları kaçırılmadan yakalanıyor.

---

##Model Karşılaştırmaları

Çalışma kapsamında, önerilen CNN + RF modeli farklı senaryolar ve
alternatif sınıflandırıcılarla da karşılaştırılmıştır:

###CNN Özellikleri Üzerinde Farklı Modeller

CNN’in ürettiği özellik vektörleri (**Z**) kullanılarak üç farklı sınıflandırıcı eğitildi:

| Model         | Accuracy (≈) | Notlar                                      |
|---------------|--------------|---------------------------------------------|
| **Random Forest** | **%94–95**    | Dengeli başarı, hızlı ve kararlı          |
| **SVM**           | ≈ %91        | Daha yavaş, yüksek boyutlu veride zorlanıyor |
| **XGBoost**       | ≈ %95        | En yüksek accuracy, ancak daha maliyetli  |

- **RF**, başarı / hız / karmaşıklık dengesinde en pratik çözüm olarak öne çıkmıştır.
- **XGBoost**, biraz daha yüksek doğruluk sağlasa da eğitim süresi ve model karmaşıklığı daha yüksektir.
- **SVM**, yüksek boyutlu CNN özelliklerinde hem daha yavaş çalışmış hem de doğruluk olarak geride kalmıştır.

###CNN’siz Random Forest (Baseline)

Ek olarak, CNN kullanılmadan **ham bayt temsili** doğrudan Random Forest’a
verilerek bir taban çizgi (baseline) modeli kurulmuştur.

- Bu model bazı senaryolarda **yüksek doğruluk** üretse de:
  - Özellik uzayı tamamen ham baytlara bağlıdır,
  - Yorumlanabilirlik düşüktür,
  - Farklı veri setlerine genellenebilirlik, CNN tabanlı temsile göre daha zayıf olabilir.

Bu nedenle proje kapsamında, hem literatür uyumluluğu (byte-to-image yaklaşımı)
hem de daha anlamlı bir temsil uzayı elde etmek için CNN + RF hibrit yapısı
esas model olarak tercih edilmiştir.

###1D CNN ile Karşılaştırma

- Paketler sadece **1 boyutlu bayt dizisi** olarak ele alınıp **1D CNN + RF**
kombinasyonu da test edilmiştir.
- 1D CNN, doğruluk olarak kabul edilebilir sonuçlar üretmiş olsa da:
  - 2D CNN’e göre biraz daha düşük accuracy/F1 değerleri vermiş,
  - Yerel uzamsal desenleri (ör. başlık–payload ilişkisi) yakalamada daha sınırlı kalmıştır.
- Gerçek ağ trafiğinde bayt düzeni kaydığında/bozulduğunda, 2B görüntü
temsili üzerinden çalışan CNN’lerin daha esnek ve gürültüye dayanıklı olacağı
değerlendirilmiştir.

---

##Referanslar

Bu çalışma aşağıdaki literatüre dayanmaktadır:

- **Veri Seti:**
  - Neto, E. C. P. et al.,  
    *“CICIoT2023: A Real-time Dataset and Benchmark for Large-Scale Attacks in IoT Environment”*, 2023.
- **Görselleştirme (Byte-to-Image):**
  - Wang, W. et al.,  
    *“Malware Traffic Classification Using Convolutional Neural Networks for Representation Learning”*, IEEE, 2017.
- **Hibrit CNN + RF Yaklaşımları:**
  - *“A CNN-RF Hybrid Model for Intrusion Detection System”*, çeşitli IDS çalışmalarında benzer hibrit yapılar.

---

##Gerekli Python kütüphaneleri:

```bash
pip install scapy tensorflow scikit-learn matplotlib seaborn
