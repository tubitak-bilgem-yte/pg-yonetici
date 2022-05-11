---
title: "Veritabanı Servisinde İyileştirme"
parent: Performans
layout: default
nav_order: 1
---

## Veritabanı Servisinde İyileştirme

### Vacuum

- Veritabanında kullanılmayan kayıtları temizleyerek veritabanı boyutunu küçültür.
- Tüm PostgeSQL sistemleri için mutlaka gerekli.
- PostgreSQL öntanımlı otomatik vakumlama yapıyor (autovacuum).
- Vacuum Full: Tüm tabloyu sıfırdan baştan oluşturur.
- Vacuum Freeze: Tablonun otomatik vakumlanmamasını sağlar.

#### Elle Vakumlamak

- Otomatik vakumlama istenmeyen zamanlarda I/O yaratıyor
- Sistemin kullanılmadığı saatler var
- Otomatik vakumlama kapatalıp, o saatler için cron’dan vakum tetiklenebilir.

#### Otomatik Vakumlamayı Ehlileştirmek

- Hemen her vakumlama sorunu, daha sık vakumlayarak çözümlenebilir:
  - *autovacuum_vacuum_cost_limit = -1*
  - *autovacuum_vacuum_cost_delay = 20*
  - *autovacuum_vacuum_threshold = 50*
  - *autovacuum_vacuum_scale_factor = 0.2*
  - *autovacuum_analyze_threshold + autovacuum_analyze_scale_factor*
  - *Tüm `autovacuum_*` parametreleri her tablo için ayrı ayrı düzenlenebiliyor.*

#### Tam Vakumlamak

- Düzenli Vacuum Full çalıştırmak gereksiz
- Düzenli vakumlanan tablolar için hiç gerekmemeli
- Tablo kitlediği için servis kesintisine yol açar
- Sadece düzenli vakumlanmadığı için tablo satırlarının çoğu öldüyse anlamlı.

### Veritabanı Parametrelerini Düzenleme

#### Bellek Öncelikli Parametreler

- PostgreSQL’ın öntanımlı değerleri genel olarak yeterli
- max_connections
- shared_buffers: RAM * 1/4
- work_mem
- maintenance_work_mem
- effective_cache_size: RAM * 3/4

#### I/O Öncelikli Parametreler

- wal_buffers
- checkpoint_segments: 10 (en az)
- checkpoint_completion_target: 0.9

{% include links.html %}
