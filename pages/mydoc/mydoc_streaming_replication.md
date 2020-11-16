---
title: "Streaming Replication"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 16, 2020
summary: "Streaming Replication"
sidebar: mydoc_sidebar
permalink: mydoc_streaming_replication.html
folder: mydoc
---

## Streaming Replication

- Master-Standby yapısında replikasyon sağlıyor
- Master sunucuda yapılan değişiklikler → WAL ( Write-Ahead Log ) arşivleri
- Öntanımlı asenkron, senkron olmaya zorlanabiliyor
- Standby WAL arşivlerini master’dan alıp kendine uyguluyor
- Dosya sistemindeki dosyaların farkları (blok bazlı) aktarılıyor

**Planlama**:

- PostgreSQL sunucuları aynı sürümde, işletim sisteminde ve mimaride olmalı
- PostgreSQL sunucuların dizin yerleri aynı olmalı
- CREATE TABLESPACE ile master’da yaratılan yeni tablespace’ler olursa, standby’larda da tek tek elle yaratılmalı
- Herhangi bir sorun olmaması için, tablespace’leri elle oluşturmak / düzenlemek yerine, standby’ın sıfırdan oluşturulması da önerilenler arasında.

### Kurulum - Master

Replikasyon için bir kullanıcı oluşturuyoruz:

```sql
psql -c "CREATE ROLE replikasyon WITH REPLICATION \
            PASSWORD 'parola' LOGIN;"
```

postgresql.conf içine:

```yaml
listen_addresses = '*'
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 32
wal_log_hints = on
```

pg_hba.conf içine:

```sql
host    replication     replikasyon     192.168.56.0/24       md5
```

Servisi yeniden başlatıyoruz:

```sql
systemctl restart postgresql-11
```

- Replikasyon için, replikasyon yetkileri olan özel bir kullanıcı yaratıldığına dikkat. Her yetkiyi vermiyoruz.
- wal_level için 9.6 öncesi sürümlerde "archive" veya "hot_standby" kullanılırdı. Şimdi bu ikisi yerine "replica" kullanılıyor.
- wal_log_hints ayarı, devreden çıkarılan bir master tekrar kümeye dahil edilmek isterse ihtiyaç duyulan bir ayar. Geçiş sırasında wal logların ayrılan timeline’ını birleştirmeye yarıyor.
- Güvenlik duvarı varsa iki sunucu arasında 5432 portuna izin verilmeli.
- SELinux etkinse, oluşturulan dosyaların SELinux context’lerine de dikkat edilmeli.

### Kurulum - Slave

Servisi durduralım, veri dizinini boşaltalım, ``pg_basebackup`` ile master’dan veritabanını alalım (data dizini boş olmalı):

```text
# systemctl stop postgresql-11
# su - postgres
$ rm -rf /var/lib/pgsql/11/data/*

$ pg_basebackup --pgdata=/var/lib/pgsql/11/data \
    --host=192.168.56.101 --username=replikasyon \
    --password --wal-method=stream --format=plain \
    --progress --verbose

Password:
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/2000060 on timeline 1
pg_basebackup: starting background WAL receiver
24442/24442 kB (100%), 1/1 tablespace
pg_basebackup: write-ahead log end point: 0/2000168
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: base backup completed
```

Gelen yedekle beraber ayar dosyaları da (postgresql.conf, pg_hba.conf) geldi. Bunlar slave’e göre düzenlenmeli. *postgresql.conf* dosyasına:

```yaml
listen_address = '*'
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 32
hot_standby = on
```

``/var/lib/pgsql/11/data/recovery.conf`` dosyası şuna benzer içerikte oluşturulur:

```yaml
standby_mode = 'on'
primary_conninfo = 'host=192.168.56.101 port=5432 application_name=standby1 user=replikasyon password=parola'
trigger_file = '/var/lib/pgsql/11/data/promote_db'
```

{% include note.html content=" primary_conninfo parametresinin tek satırda olması gereklidir."%}

Servis başlatılır ve loglar izlenir:

```sql
$ exit
# systemctl start postgresql-11

# tail -f /var/lib/pgsql/11/data/log/postgresql-Thu.log
2018-03-22 20:25:43.200 +03 [1650] LOG:  database system was interrupted; last known up at 2018-03-22 20:13:08 +03
2018-03-22 20:25:43.470 +03 [1650] LOG:  entering standby mode
2018-03-22 20:25:43.473 +03 [1650] LOG:  redo starts at 0/4000028
2018-03-22 20:25:43.475 +03 [1650] LOG:  consistent recovery state reached at 0/4000130
2018-03-22 20:25:43.475 +03 [1646] LOG:  database system is ready to accept read only connections
2018-03-22 20:25:44.008 +03 [1654] LOG:  started streaming WAL from primary at 0/5000000 on timeline 1
```

Bu noktadan sonra Master’da yapılan her işlem Slave’de gözlenir. Slave’in senkron durumda olduğunu kontrol etmek için Master ve Slave’de şu sorgular çalıştırılır (sonuçları eşit ise senkrondur):

**Master:**

```sql
postgres=# SELECT pg_current_wal_lsn();
 pg_current_wal_lsn
--------------------
 0/5000140
(1 row)
```

**Slave:**

```sql
postgres=# SELECT pg_last_wal_receive_lsn();
 pg_last_wal_receive_lsn
-------------------------
 0/5000140
(1 row)
```

- `pg_basebackup` için standby’ın veri dizini boş olmalı.
- `standby_mode` ayarı, sunucunun standby çalıştığını belirtir ve salt okunur çalıştırır.
- recovery.conf: Standby sunucu ayarları buradan yapılıyor. Geçmişe dönük uyumluluk için adı recovery.conf.
- Replikasyon ayarlandıktan sonra Master’a veri insert edilip Slave’de aynı veri sorgulanarak katılımcılara gösterilebilir.

### Failover

`recovery.conf` dosyasında belirtilen `trigger_file` dosyası oluşturulduğu anda failover başlar.

```shell
touch /var/lib/pgsql/11/data/promote_db
```

Loglarda trigger dosyası bulunduğu ve failover gerçekleştiğine dair kayıtlar görülür.

```sql
< 2018-01-08 12:16:19.957 +03 > LOG:  trigger file found: /var/lib/pgsql/11/data/promote_db
< 2018-01-08 12:16:19.957 +03 > FATAL:  terminating walreceiver process due to administrator command
< 2018-01-08 12:16:19.959 +03 > LOG:  invalid record length at 0/13000140: wanted 24, got 0
< 2018-01-08 12:16:19.959 +03 > LOG:  redo done at 0/13000108
< 2018-01-08 12:16:20.043 +03 > LOG:  selected new timeline ID: 2
< 2018-01-08 12:16:20.337 +03 > LOG:  archive recovery complete
```

*recovery.conf* dosyasının adı recovery.done olarak değişir. Böylece bir sonraki açılışta da master olarak çalışır.

```shell
-rw-r--r-- 1 postgres postgres 194 Jan  5 13:43 recovery.done
```

### WAL Düzeyleri

**minimal**: Çökmeden dönecek kadar (öntanımlı)

**replica**: Arşivleme ve streaming replication için gerekli bilgiler (eski sürümlerde archive + hot_standby)

**logical**: Dış kaynak gönderim eklentileri için ek bilgiler

{% include note.html content="Her log düzeyi, bir öncekinin verilerini içerir ve log düzeyi arttıkça, logun boyutu büyür."%}

### WAL Arşivlemek

WAL dosyalarının ``pg_wal`` dizininden silinmesi yerine başka bir dizine arşivlenebiliyor:
postgresql.conf ayarları:

```yaml
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /srv/wal_archive/%f && cp %p /srv/wal_archive/%f'
```

{% include note.html content="%f = dosya ismi, %p = dosya yolu (pg_wal)"%}

### Standby WAL Kaynakları

Standby başlatıldığında sırasıyla, `restore_command` tanımlandıysa,

- WAL arşiv dizini, örnek recovery.conf dosyasında:

```yaml
restore_command = 'cp /srv/wal_archive/%f "%p" 2>>/var/lib/pgsql/11/standby.log'
```

- pg_wal dizinindeki WAL
- Master’dan TCP’den bağlanıp WAL alıp, kendindeki eksikleri uygular.

{% include links.html %}
