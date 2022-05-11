---
title: "Donanım Performansı"
parent: Performans
layout: default
nav_order: 5
---

## Donanım Performansı

### Donanım ve Yapılandırma Seçimleri

{% include image.html file="donanim_parcalari.png" url="/pg-yonetici/images/donanim_parcalari.png" alt="Donanım Parçaları" caption="Donanım Parçaları"%}

- **CPU**
  - En az iki core
  - Core sayısı != CPU sayısı
  - PostgreSQL sunucu fonksiyonları yoğun kullanılırsa önem kazanır.
- **RAM**
  - Ne kadar çok RAM o kadar iyi
  - Fazla RAM önbellek olarak kullanılır
- **Disk**
  - Performansı en önemli parça
  -Tür: SATA, SAS, SSD
  -RPM: 7200 / 10000 / 15000
  -SAN varsa Fiber olmalı

### RAID Yapılandırması

- Donanım RAID
  - Pil destekli bir bellek cache’i kullanır
  - Herhangi bir çökmede daha rahat kurtarılabilir
  - İşletim sistemi düzeyinde görünmez
- Yazılım RAID
  - Yazma performansı daha düşük
  - Raporlama sistemleri için kullanılabilir

| RAID | Okuma Performansı | Yazma Performansı | Disk Sayısı (En Az) | Hata Toleransı | Kapasite Verimi |
|-------|--------|-------|--------|-------|--------|
| 0 | + | + | 2 | 0 | %100 |
| 1 | + | / | 2 | 1 | %50 |
| 5 | + | - | 3 | 1 | %66 |
| 6 | + | — | 4 | 2 | %50 |
| 1+0 | + | + | 4 | 1 (*) | %50 |

{% include image.html file="standart_raid10_yapisi.png" url="/pg-yonetici/images/standart_raid_yapisi.png" alt="Standart Raid 10 Yapısı.png" caption="Standart Raid 10 Yapısı"%}

### Fiziksel Makine mi? Sanal Makine mi?

- Fiziksel Makine
  - Doğrudan donanıma erişim
  - Donanımın baştan iyi planlanma zorunluluğu
  - Başka bir donanıma taşıma problemi
  - Network KVM, DRAC, ILO gibi bir bağlantı gerekli

- Sanal Makine
  - Sanallaştırma altyapısı nedeniyle performans kaybı
  - Donanımı ihtiyaç oldukça arttırabilmek
  - Makinenin kolayca başka bir fiziksel makineye taşınabilmesi

### Standart Disk Yapısı mı? LVM mi?

- Standart
  - Doğrudan diske erişim
  - Donanımın baştan iyi planlanma zorunluluğu
  - Disk büyütmek ancak son disk bölümü için ve RAID ile

- LVM
  - Canlı disk büyütebilmek
  - Canlı bir disk bölümünü bir fiziksel diskten diğerine taşıyabilmek
  - Yedek alırken snapshot özelliği
  - Dosya sistemi kurtarmakta zorluk

### Donanım Performansı Ölçümü ve Darboğazının İncelenmesi

**Donanım Performansının Ölçümü**:

- Gerçek anlamda yapması zordur.
- Karşılaştırarak, iyileşme görme amaçlı mantıklı
- Çok çeşitli araçlar var:
  - unixbench
  - fio
  - sysbench
  - hdparm (I/O)

**Darboğazının İncelenmesi**:

- Genel Araçlar: uptime, (h)top, vmstat, dstat
- I/O: iotop, iostat
- RAM: free
- CPU: mpstat
- Ağ: nicstat, iperf

{% include links.html %}
