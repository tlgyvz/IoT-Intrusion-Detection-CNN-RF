# IoT Saldırı Tespiti: Hibrit CNN ve Random Forest Yaklaşımı

Bu proje, **CICIoT2023** veri setini kullanarak IoT ağlarındaki siber saldırıları tespit etmek amacıyla geliştirilmiş hibrit bir makine öğrenmesi modelidir. Proje kapsamında ağ paketleri görselleştirilmiş ve derin öğrenme ile geleneksel makine öğrenmesi yöntemleri birleştirilmiştir.

#Proje Özeti
- **Veri Seti:** [CICIoT2023](https://www.unb.ca/cic/datasets/iotdataset-2023.html) (PCAP formatında ham ağ trafiği).
- **Yöntem:** Byte-to-Image dönüşümü + CNN (Öznitelik Çıkarıcı) + Random Forest (Sınıflandırıcı).
- **Sınıflar:** Benign (Normal), DDoS-SYN Flood, Mirai-udpplain, Recon-PortScan.
- **Başarı Oranı:** %93.09 Genel Doğruluk (Accuracy).

#Metodoloji
Bu çalışmada "Trafikten Görüntüye" (Traffic-to-Image) yaklaşımı benimsenmiştir:
1. **Önişleme:** PCAP dosyalarındaki ham paket baytları okunarak her bir paket 28x28 boyutunda gri tonlamalı (grayscale) görsellere dönüştürülmüştür.
2. **Öznitelik Çıkarımı (CNN):** Evrişimli Sinir Ağları (CNN), paket görsellerindeki mekansal desenleri otomatik olarak öğrenmek için kullanılmıştır.
3. **Sınıflandırma (Random Forest):** CNN'in son katmanından elde edilen öznitelik vektörleri, hiperparametre optimizasyonu yapılmış bir Random Forest modeline girdi olarak verilmiştir.

#Sonuçlar
Modelin test verisi üzerindeki performansı aşağıdadır:

| Saldırı Türü | Precision | Recall | F1-Score |
|--------------|-----------|--------|----------|
| **Benign**   | 0.95      | 0.94   | 0.94     |
| **SYN Flood**| 0.97      | 0.90   | 0.93     |
| **Mirai**    | 0.99      | 0.92   | 0.95     |
| **PortScan** | 0.84      | 0.96   | 0.90     |

**Genel Doğruluk:** %93.09

#Referanslar
Bu çalışma aşağıdaki temel literatür kaynaklarına dayanmaktadır:
- **Veri Seti:** Neto, E. C., et al. (2023). "CICIoT2023: A Real-time Dataset for Cybersecurity in the Era of Modern IoT".
- **Görselleştirme:** Wang, W., et al. (2017). "Malware Traffic Classification Using Convolutional Neural Networks for Representation Learning".
- **Hibrit Model:** "A CNN-RF Hybrid Model for Intrusion Detection System".

#Kurulum ve Çalıştırma
Projenin çalışması için gerekli kütüphaneler:
```bash
pip install scapy tensorflow scikit-learn matplotlib seaborn
