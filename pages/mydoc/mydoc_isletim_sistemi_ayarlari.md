---
title: "Sorguların İyileştirilmesi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 16, 2020
summary: "Sorguların İyileştirilmesi"
sidebar: mydoc_sidebar
permalink: mydoc_isletim_sistemi_ayarlari.html
folder: mydoc
---

Donanım Performansı
mydoc_donanim_performansi.html

## İşletim Sistemi Ayarları

### PostgreSQL Kaynak Kullanımı

**NUMA**:

- NUMA (Non Uniform Memory Access)
  - CPU’ların kendi bellekleri var
  - CPU bellekleri NUMA iç ağı ile bağlı
  - Memory interleaving: CPU kendi belleğini bitirince diğer CPU belleklerine kullanmaya çalışıyor.

Sorun olduğunda?

- CPU core’ları anlamsız biçimde çok yüklenir
- İhtiyaç gözükmemesine rağmen swap kullanımı başlar

- PostgreSQL için önerilenler:
  - BIOS’ta Memory Interleaving açık olması
  - Postgresql sürecinin NUMA interleave ayarı ile başlatılması

```sql
# systemctl edit postgresql-11
[Service]
ExecStart=
ExecStart=numactl --interleave=all /usr/pgsql-11/bin/postmaster \
                     -D ${PGDATA}

# systemctl daemon-reload
```

PostgreSQL için önerilenler:

- `vm.zone_reclaim_mode = 0`
- `kernel.numa_balancing = 0`

**Huge Pages**:

- Linux öntanımlı 4K’lık parçalar halinde bellek yerleştirir
- Fiziksel bellek adreslerinin listesi → TLB
- Kullanılan RAM çok artınca TLB’de tutulan adres sayısı çok artar ve performans düşer.
- Huge pages = 2048K (öntanımlı)

Önerilen (bol RAM ile):

- PostgreSQL `shared_buffer` ve Huge page kullanan diğer uygulamaların kullanacağı memory miktarının huge page için ayrılan alana sığması
- Huge page kullanmayacak uygulamalar için huge page olarak ayrılmayan yer kalması

Değerin hesaplanması:

- huge_pages ayarının kapatılarak PostgreSQL’i başlat (postgresql.conf `huge_pages = off`)
- Kullanılan bellek miktarını huge page boyutuna böl
- huge_pages ayarını açarak PostgreSQL’i başlat (postgresql.conf `huge_pages = try`)

```sql
# pmap `head -1 /var/lib/pgsql/11/data/postmaster.pid` | awk '/rw-s/ && /zero/ {print $2}'
6490428K
# grep ^Hugepagesize /proc/meminfo
Hugepagesize:       2048 kB
# python -c "print 6490428/2048.0"
3169.15429688
# sysctl -w vm.nr_hugepages=3170
# grep Huge /proc/meminfo
```

### I/O Scheduler

- CFQ: Her derde deva ve karmaşık
- Noop: Ramdisk, flash ve benzeri hızlı aygıtlarda
- Deadline
  - Veritabanı sistemleri için önerilen
  - n okumaya karşılık 1 yazma işlemi yapıyor
  - Daha iyi okuma performansı sağlıyor
  - Azar azar okuma, ardışık yazma işlemlerinde başarılı.
- Farklı roldeki veritabanı sunucuları için farklı scheduler kullanılabilir.

### Swap (Takas)

- Sistem hiçbir zaman swap’e düşmemeli
- RAM yetmiyorsa RAM eklenmeli
- Swap tamamen kapatılmamalı → `OOM_KILLER`
- Sorun: Çok RAM var ama sistem swap’e düşüyor
  - Çözüm: `vm.swappiness = 1`

### Diske Veri Yazma

- Linux’ta öntanımlı vm.dirty_ratio ve vm.dirty_background_ratio
- Yüksek RAM miktarında bu oranlar çok büyük sonuç veriyor
- Sorun: Checkpoint sırasında yoğun I/O sonucu sistemin teklemesi
  - Çözüm: `vm.dirty_bytes` ve `vm.dirty_background_bytes` ile doğrudan değer belirtme
  - Doğru değer RAID belleğine göre de değişken
  - Doğru değeri bulmak için örnekler denenerek sonuçları karşılaştırılmalı

### İşlemci Güç Koruması

- acpi_cpufreq + intel_pstate (yeni çekirdek)
- scaling_governor: performance, powersave, …​
- performance: sürekli tam güç
- powersave: en düşük hızda çalıştır, gerekince hızlan
- performance, powersave’den daha hızlı olabiliyor
- /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

### Yazma Engeli

- Veri kaybı riskini azaltmak için dosya sistemine write barrier
- RAID aygıtınızın pil destekli yazma önbelleği varsa gerekli değil.
- mount seçeneklerine nobarrier eklenebiliyor
- %3 yazma performansında artış

### Dosya Erişilme Tarihi (atime)

- Dosya sistemleri bir dosyanın son erişilme zamanını tutuyor.
- Yoğun I/O olan sistemlerde performans kaybı.
- Hiç tutmamak belli uygulamaları kırabiliyor
- Mount seçenekleri:
  - noatime: Hiç tutmamak
  - relatime: Tutarsızlık olursa güncelle.

### Dosya Sistemi Seçimi

- Ext4, XFS
  - 90’ların tasarımı
  - Zaman içinde iyileşiyor
  - Güvenilir journal dosya sistemleri
  - Kanıtlanmış
  - 16 TB+ dosya sistemleri → XFS
- Btrfs, ZFS
  - Yeni nesil (2000’lerin ikinci yarısı)
  - Kendinden LVM, snapshot, sıkıştırma ve benzeri yetenekler
  - Büyük hacimde veri için tasarlanmış

{% include links.html %}
