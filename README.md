# 🚀 Python ile 1 Saniyede Binlerce Port Taramak: Threading ve I/O İşlemlerinin Sırrı

## Giriş

Bu makale, basit bir **Port Tarayıcı** uygulamasının hızını neden %99 oranında artırmak zorunda kaldığımızı ve bu performans engellerini aşmak için **Eş Zamanlılık (Concurrency)** prensibini nasıl uyguladığımızı teknik olarak açıklamaktadır.

Port tarama, bir ağ cihazından cevap beklediğimiz için doğası gereği bir **I/O (Girdi/Çıktı) Yoğun** işlemdir. Kodun çoğu zamanı işlem yapmak yerine **ağdan cevap bekleyerek** geçer. Amacımız, bu bekleme sürelerini yok sayarak tarama süresini dramatik bir şekilde kısaltmaktı.

---

## 1. Problem Tanımı: Senkron (Sıralı) Yaklaşım

İlk taslakta uygulanan **Senkron** yürütme modeli, tarama süresini kabul edilemez seviyelere çıkarıyordu. Her port için bağlantı denemesi sırayla başlar ve sırayla sona ererdi.

### ⏱️ Senkron Tarama Süresi Hesaplaması

Projede kullandığımız zaman aşımı (timeout) değeri 0.1 saniye olduğunda, senkron yapının performansı aşağıdaki gibiydi:

| Parametre | Değer | Hesaplama |
| :--- | :--- | :--- |
| Taranan Port Sayısı | 80 | |
| Port Başına Bekleme Süresi | 0.1 saniye | |
| **Tahmini Toplam Süre** | **8 saniye** | 80 port \* 0.1 saniye/port |

> **Ana Engelleme:** Bir portun denemesi tamamlanmadan, program diğer portun denemesine başlayamaz. Bu, 80 port için en az 8 saniyelik bir gecikme anlamına geliyordu.

---

## 2. Çözüm: Threading (Çoklu İş Parçacığı) ile Eş Zamanlılık

I/O yoğunluğu yüksek olan bu sorunun çözümü, **eş zamanlılık** sağlamaktı. Python'da, bu tür ağ beklemelerini yönetmek için **`threading`** kütüphanesini kullandık.

### 💡 Threading Mimarisi Nasıl Çalışır?

1.  **Paralel Görev Atama:** Her bir port bağlantı denemesi ayrı bir **İş Parçacığına (Thread)** atanır.
2.  **I/O Beklemesini Maskeleme:** Bir thread ağdan cevap beklerken (I/O beklemesi), CPU hemen diğer thread'i çalıştırır. CPU, boş kalmak yerine thread'ler arasında hızla geçiş yapar.
3.  **Süre Kazanımı:** Threading sayesinde, 80 farklı bekleme süresi üst üste toplanmak yerine, hepsi **aynı anda** çalışıyormuş gibi görünür.
### 📉 Sonuç: Performans Artışı Tablosu

Threading ile tarama süresi, toplam bekleme süresi olmaktan çıkıp, sadece en uzun tek bir bekleme süresine düştü.

| Özellik | Senkron (Sıralı) | Threading (Eş Zamanlı) |
| :--- | :--- | :--- |
| **Hedeflenen Engel** | I/O Beklemesi | I/O Beklemesi |
| **80 Port Tahmini Süre** | ~ 8 saniye | **~ 0.1 saniye** |
| **Kazanılan Verim** | Düşük | **%99 zaman tasarrufu** |

---

### Sonuç: Doğru Aracı Seçmek

Bu proje, bir geliştiricinin doğru teknik kararı vermesinin önemini göstermektedir: Bir I/O problemine (ağ beklemeleri) karşı en etkili ve pratik çözüm **threading** olmuştur.

Bu analiz, sadece çalışan bir kod yazmak yerine, **performans engellerini aşabilen ve doğru teknik kararları verebilen** bir geliştirici olduğumu kanıtlamaktadır.

---
### 📚 Derinlemesine Analiz
Bu projenin arkasındaki performans kararlarını merak ediyor musunuz?

➡️ **Makale:** [Python ile 1 Saniyede Binlerce Port Taramak: Threading ve I/O İşlemlerinin Sırrı](Makalenizin_linki)


### 🐍 Uygulanan Kritik Kod Bloğu

Bu performans artışını sağlayan temel kod yapısı şudur:

```python
# Threading kullanılan eş zamanlı tarama
for port in range(baslangic, bitis + 1):  
    # Her port için yeni bir thread oluşturulur
    thread = threading.Thread(target=tarama_portu, args=(hedef, port))
    
    # Thread başlatılır; program bu noktada beklemez.
    thread.start()
