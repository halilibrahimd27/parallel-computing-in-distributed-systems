# MPI + OpenMP Dağıtık Hesaplama Sistemi

Bu proje, **Docker Compose kullanarak dağıtık bir hesaplama ortamı oluşturur**. MPI (**Message Passing Interface**) kullanarak düğümler arasında veri paylaşımı yapılır ve OpenMP (**Open Multi-Processing**) ile her düğüm kendi içinde **çok çekirdekli paralel hesaplama yapar**.

---

## 🚀 **Proje Yapısı**

```
/MPI-OpenMP-Cluster
│── Dockerfile
│── docker-compose.yml
│── mpi_openmp.cpp
└── README.md
```

---

## 🎯 **Proje Hedefleri**
- Docker ile **dağıtık bir sistem** oluşturmak.
- **Docker Compose** kullanarak **birden fazla düğüm** çalıştırmak.
- **MPI** ile düğümler arasında veri iletişimi sağlamak.
- **OpenMP** ile her düğümün içinde **çok çekirdekli paralel hesaplama yapmasını sağlamak.**
- **Dağıtık sistemde hesaplama sonuçlarını toplamaya yönelik bir pipeline kurmak.**

---

## 📦 **Kurulum ve Çalıştırma**

### **1️⃣ Docker ve Docker Compose'u Kurun**
Eğer sisteminizde Docker ve Docker Compose yoksa, aşağıdaki komutlarla kurabilirsiniz:

#### **Ubuntu / Debian**
```sh
tsudp apt update
sudo apt install -y docker.io docker-compose
```

#### **MacOS**
```sh
brew install docker docker-compose
```

#### **Windows** (WSL2 Kullanman Önerilir)
- [Docker Desktop](https://www.docker.com/products/docker-desktop) 'u indir ve kur.

---

### **2️⃣ Projeyi Çalıştırma**
Aşağıdaki komutlarla proje başlatılabilir:

```sh
docker-compose build
docker-compose up
```

> **Not:** Eğer eski konteynerlerden kaynaklı sorun yaşarsan, aşağıdaki komutu kullanarak eski konteynerleri temizle:

```sh
docker-compose down --remove-orphans
docker system prune -a
```

---

## ⚙️ **Sistem Mimarisi**
Bu proje, **Master-Worker** mimarisini kullanarak veri işleme yapar:

1. **Master düğüm** (**rank 0**) ilk veriyi başlatır.
2. **Her düğüm MPI ile veriyi alır, OpenMP ile paralel işleyerek günceller ve sonraki düğüme yollar.**
3. **Son düğüm** (**rank N-1**), nihai sonucu **Master'a geri yollar.**

---

## 📜 **Beklenen Çıktı**
```plaintext
🎯 [Master] Başlangıç değeri: 0
📤 [Master] Düğüm 1'e veri gönderildi: 10

📥 [Düğüm 1] Veri alındı: 10
🔄 [Düğüm 1] Thread 0/4 işleniyor...
📤 [Düğüm 1] Güncellenmiş veri: 15
➡️ [Düğüm 1] Düğüm 2'ye veri gönderildi: 15

📥 [Düğüm 2] Veri alındı: 15
🔄 [Düğüm 2] Thread 0/4 işleniyor...
📤 [Düğüm 2] Güncellenmiş veri: 25
➡️ [Düğüm 2] Düğüm 3'e veri gönderildi: 25

📥 [Düğüm 3] Veri alındı: 25
🔄 [Düğüm 3] Thread 0/4 işleniyor...
📤 [Düğüm 3] Güncellenmiş veri: 40
✅ [Düğüm 3] Son düğüm. Master’a sonuç gönderildi: 40

🏁 [Master] Nihai sonuç alındı: 40
```

---

## 📊 **MPI ve OpenMP Performans Karşılaştırması**

Bu rapor, **MPI (Message Passing Interface)** ve **OpenMP (Open Multi-Processing)** teknolojilerinin **performans farklarını ve kullanım senaryolarını** karşılaştırarak hangi durumlarda daha verimli olduklarını açıklamaktadır.

### 📌 **Genel Karşılaştırma**

| **Özellik**          | **MPI (Message Passing Interface)** | **OpenMP (Open Multi-Processing)** |
|----------------------|--------------------------------|--------------------------------|
| **Kullanım Alanı**   | Dağıtık sistemlerde düğümler arası iletişim | Tek sistemde çok çekirdekli işlem |
| **İletişim Modeli**  | Düğümler arası mesaj iletimi (process bazlı) | Paylaşılan bellek üzerinden doğrudan erişim (thread bazlı) |
| **Performans**       | Büyük veri kümeleri ve küme bilgisayarlarda avantajlı | Tek makine üzerindeki hesaplamalarda hızlı |
| **Ölçeklenebilirlik**| Büyük ölçekli küme sistemlerinde kolay ölçeklenir | Makinedeki çekirdek sayısıyla sınırlıdır |
| **Gecikme**          | Ağ gecikmeleri nedeniyle işlem süresi artabilir | Bellek paylaşımı nedeniyle düşük gecikme |
| **Karmaşıklık**      | Paralel programlama için daha karmaşıktır | Daha basit ve yönetilebilir |

---

### 📌 **Performans Karşılaştırması**

#### 🖥️ **1️⃣ Küçük Ölçekli Paralel İşlem (Tek Makine)**
- **OpenMP, aynı makinede çok çekirdekli işlem yapmada daha hızlıdır** çünkü bellek paylaşımını doğrudan kullanır.
- **MPI burada daha yavaş olabilir** çünkü her işlem birbirinden bağımsızdır ve haberleşme gereklidir.

**🔹 Örnek Senaryo:** Küçük boyutlu matris çarpımı  
✅ **Kazanan: OpenMP**

---

#### 🌍 **2️⃣ Büyük Veri Setleri ile Dağıtık Hesaplama**
- **MPI, büyük veri kümeleriyle daha iyi ölçeklenir** çünkü birden fazla makineye yayılabilir.
- **OpenMP, tek makinede sınırlıdır** ve veri seti büyüdükçe verimsiz hale gelir.

**🔹 Örnek Senaryo:** Küme bilgisayarlarında büyük veri analizi  
✅ **Kazanan: MPI**

---

#### ⚡ **3️⃣ Hibrit Kullanım Senaryosu (MPI + OpenMP)**
- **MPI, düğümler arası veri paylaşımını yönetirken**, **OpenMP her düğüm içinde paralel işlem yapabilir**.
- Bu yapı **büyük ölçekli hesaplamalarda en yüksek verimi sağlar**.

**🔹 Örnek Senaryo:** Büyük ölçekli bilimsel simülasyonlar (Hava durumu modelleme, fizik hesaplamaları)  
✅ **Kazanan: Hibrit Model (MPI + OpenMP)**

---

## 📌 **Sonuç**
- **Küçük ölçekli, tek makine üzerindeki paralel hesaplamalar için OpenMP daha hızlıdır.**
- **Büyük ölçekli, çok düğümlü sistemlerde MPI daha verimlidir.**
- **En iyi performans, MPI ile dağıtık hesaplama + OpenMP ile yerel paralel işlem kombinasyonuyla elde edilir.**

🚀 **Bu projede hem MPI hem de OpenMP kullanarak en iyi performans sağlandı.**

