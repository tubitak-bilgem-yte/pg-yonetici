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

- Primary-Standby yapısında replikasyon sağlıyor
- Primary sunucuda yapılan değişiklikler → WAL ( Write-Ahead Log ) arşivleri
- Öntanımlı asenkron, senkron olmaya zorlanabiliyor
- Standby WAL arşivlerini Primary’dan alıp kendine uyguluyor
- Dosya sistemindeki dosyaların farkları (blok bazlı) aktarılıyor

**Planlama**:

- PostgreSQL sunucuları aynı sürümde, işletim sisteminde ve mimaride olmalı
- PostgreSQL sunucuların dizin yerleri aynı olmalı
- CREATE TABLESPACE ile Primary’da yaratılan yeni tablespace’ler olursa, standby’larda da tek tek elle yaratılmalı
- Herhangi bir sorun olmaması için, tablespace’leri elle oluşturmak / düzenlemek yerine, standby’ın sıfırdan oluşturulması da önerilenler arasında.

### Kurulum - Primary

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
- wal_log_hints ayarı, devreden çıkarılan bir Primary tekrar kümeye dahil edilmek isterse ihtiyaç duyulan bir ayar. Geçiş sırasında wal logların ayrılan timeline’ını birleştirmeye yarıyor.
- Güvenlik duvarı varsa iki sunucu arasında 5432 portuna izin verilmeli.
- SELinux etkinse, oluşturulan dosyaların SELinux context’lerine de dikkat edilmeli.

### Kurulum - Standby

Servisi durduralım, veri dizinini boşaltalım, ``pg_basebackup`` ile Primary’dan veritabanını alalım (data dizini boş olmalı):

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

Gelen yedekle beraber ayar dosyaları da (postgresql.conf, pg_hba.conf) geldi. Bunlar Standby’e göre düzenlenmeli. *postgresql.conf* dosyasına:

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

Bu noktadan sonra Primary’da yapılan her işlem Standby’de gözlenir. Standby’in senkron durumda olduğunu kontrol etmek için Primary ve Standby’de şu sorgular çalıştırılır (sonuçları eşit ise senkrondur):

**Primary:**

```sql
postgres=# SELECT pg_current_wal_lsn();
 pg_current_wal_lsn
--------------------
 0/5000140
(1 row)
```

**Standby:**

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
- Replikasyon ayarlandıktan sonra Primary’a veri insert edilip Standby’de aynı veri sorgulanarak katılımcılara gösterilebilir.

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

*recovery.conf* dosyasının adı recovery.done olarak değişir. Böylece bir sonraki açılışta da Primary olarak çalışır.

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
- Primary’dan TCP’den bağlanıp WAL alıp, kendindeki eksikleri uygular.

### Sürekli Arşivleme ve PITR

Belirli aralıklarla istenen sunucudan base backup alınır:

```shell
pg_basebackup --pgdata=/srv/backup_$(date +%F) \
    --host=192.168.56.101 --username=replikasyon \
    --password

Password:
NOTICE:  pg_stop_backup complete, all required WAL segments have been archived
```

Yedekten dönülecek yerde çalışan PostgreSQL varsa durdurulur. Veri dizini boşaltılır,

```sql
systemctl stop postgresql-11
rm -rf /var/lib/pgsql/11/data/*
```

İstenen base backup, veri dizinine kopyalanır,

```shell
cp -r /srv/backup_2018-01-08/* /var/lib/pgsql/11/data/
```

Base backuptan gelen postgresql.conf, daha önce Standby ayarlarında yaptığımız gibi değiştirilir (listen_address, archive’ın kapatılması gibi). Ayrıca Wal log dizini temizlenir, çünkü bunlar bayat Wal loglar, biz arşivlenmiş Wal loglardan seçip kullanacağız,

```shell
rm -rf /var/lib/pgsql/11/data/pg_wal/*
```

Arşivlenen Wal logları sunucuya alınır:

```shell
scp -r 192.168.56.101:/srv/wal_archive /srv/wal_archive_backup
chown -R postgres:postgres /srv/wal_archive_backup
```

Basit bir recovery.conf dosyası şu içerikte oluşturulur,

```yaml
restore_command = 'cp /srv/wal_archive_backup/%f "%p" >> /var/lib/pgsql/11/pitr.log'
recovery_target_time='2017-01-08 13:00:00'
```

Tüm veri dizininin izinleri düzenlenir:

```shell
chown -R postgres:postgres /var/lib/pgsql/11/data
```

PostgreSQL başlatılır,

```shell
systemctl start postgresql-11
```

Loglar takip edilerek veritabanının istenen zamana döndüğü görülür,

```shell
< 2018-01-08 13:26:43.868 +03 > LOG:  database system was interrupted; last known up at 2018-01-08 13:14:18 +03
< 2018-01-08 13:26:43.868 +03 > LOG:  creating missing WAL directory "pg_xlog/archive_status"
< 2018-01-08 13:26:45.338 +03 > LOG:  starting point-in-time recovery to 2017-01-08 13:00:00+03
< 2018-01-08 13:26:45.350 +03 > LOG:  restored log file "000000010000000000000014" from archive
< 2018-01-08 13:26:45.625 +03 > LOG:  redo starts at 0/14000028
< 2018-01-08 13:26:45.717 +03 > LOG:  consistent recovery state reached at 0/140000F8
< 2018-01-08 13:26:45.720 +03 > LOG:  redo done at 0/140000F8
< 2018-01-08 13:26:45.731 +03 > LOG:  restored log file "000000010000000000000014" from archive
< 2018-01-08 13:26:45.982 +03 > LOG:  selected new timeline ID: 2
< 2018-01-08 13:26:46.275 +03 > LOG:  archive recovery complete
```

### WAL Temizliği

- `pg_archivecleanup` komutu ile yapılıyor,
Otomatik olması için recovery.conf dosyasına şu satır ekleniyor:

```shell
archive_cleanup_command = '/usr/pgsql-11/bin/pg_archivecleanup /srv/wal_archive %r'
```

Elle de çalıştırılabilir. Verilen dosyadan eski dosyaları siler:

```shell
/usr/pgsql-11/bin/pg_archivecleanup /srv/wal_archive/ \
    00000001000000000000000A.00000060.backup
```

Yedekleme ve/ya log amaçlı arşiv saklanabilir. Ayrıca WAL arşivleri, bir tam yedek ile beraber point-in-time-recovery yapmak için kullanılabilir.

{% include note.html content=" archive_cleanup_command parametresinin tek satırda olması gereklidir."%}

### Replikasyon Slotları

WAL dosyalarının tüm standby’lar çekmeden silinmemesini sağlıyor

Primary’ın postgresql.conf dosyasına ek:

```yaml
max_replication_slots = 3
```

Primary üzerinde slot oluşturuluyor:

```sql
<SELECT * FROM pg_create_physical_replication_slot('standby1');
Standby’ın recovery.conf’una ek yapılıyor:
```

```yaml
primary_slot_name = 'standby1'
```

Bu şekilde standby her Primary’a gittiğinde bilgisini slot’una kaydediyor. O bilgiye göre WAL dosyası yanlışlıkla temizlenmiyor. max_replication_slots değeri en az standby sayısı kadar olmalı.

### Standby Senkronlanmasını Zorlamak

- Veri kaybı riskini kaldırmak için,
- `synchronous_standby_names = *`,
- *Değer boşsa senkron zorlanmaz (öntanımlı),
- Belirli bir transaction için `synchronous_commit` değeri ile kapatılabilir.

### Gecikmeli Standby

- Uygulama düzeyindeki hatalardan geri dönmek için,
- x saat geriden takip eden bir standby,
- Üzerine WAL uygulanabilir,
- Hızla tekrar devreye alınabilir,
- Standart replikasyondaki gibi pg_basebackup alınır ve postgresql.conf ayarlanır:

```shell
pg_basebackup --pgdata=/var/lib/pgsql/11/data \
    --host=192.168.56.101 --username=replikasyon \
    --password --xlog-method=stream --format=plain \
    --progress --verbose

vim postgresql.conf
    listen_address = 192.168.56.102
    wal_level = replica
    max_wal_senders = 3
    wal_keep_segments = 32
    hot_standby = on
```

recovery.conf dosyası şuna benzer içerikte oluşturulur, gecikme ayarına dikkat:

```text
standby_mode = 'on'
primary_conninfo = 'host=192.168.56.101 port=5432 application_name=standby1 user=replikasyon password=parola'
trigger_file = '/var/lib/pgsql/11/data/promote_db'
recovery_min_apply_delay = 2h
```

- h (saat) yanı sıra, d, m, s, ms birimleri de kullanılabilir.

İzinler düzenlenir, servis ayağa kaldırılır:

```shell
chown -R /var/lib/pgsql/11/data
systemctl start postgresql-11
```

- Gecikmeli replikasyon ilk ayarlandığında base backup’tan daha geriye gitmez, x saat sonra ilk senkronlar yapılmaya başlanır.

### İzlenmesi Gereken Değerler

**Primary'de**:

```text
postgres=# SELECT * FROM pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 2324
usesysid         | 16384
usename          | replikasyon
application_name | standby1
client_addr      | 192.168.56.102
client_hostname  |
client_port      | 55330
backend_start    | 2018-01-08 13:44:10.154573+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/170005A0
write_lsn        | 0/170005A0
flush_lsn        | 0/170005A0
replay_lsn       | 0/170005A0
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
```

Primary’da replikasyon çalışıyor mu?

**Primary’de**:

```shell
ps auxf | grep sender | grep -v grep
postgres  2324  0.0  0.3 355864  3052 ?        Ss   13:44   0:00  \_ postgres: wal sender process replication 192.168.56.102(55330) streaming 0/17000680
```

**Standby’de**:

```shell
ps auxf | grep receiver | grep -v grep
postgres  2638  0.0  0.3 362216  3200 ?        Ss   13:44   0:00  \_ postgres: wal receiver process   streaming 0/17000680
```

Hangi sunucu Primary, hangi sunucu Standby?

**Primary’de**:

```shell
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 f
```

**Standby’de**:

```shell
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
```

Replikasyonda gecikme (lag) var mı?

**Standby’de**:

```shell
postgres=#  SELECT CASE WHEN
    pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn ()
    THEN 0
    ELSE EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp())
    END AS log_delay;
 log_delay
-----------
         0
```

Primary ve Standby’in senkronluk durumları:

**Primary’de**:

```shell
postgres=# SELECT pg_current_wal_lsn();
 pg_current_wal_lsn
--------------------------
 0/13000140
(1 row)
```

**Standby’de**:

```shell
postgres=# SELECT pg_last_wal_receive_lsn();
 pg_last_wal_receive_lsn
-------------------------------
 0/13000140
(1 row)
```

### Diğer Parametreler

- archive_timeout
- max_wal_senders
- wal_sender_timeout
- wal_receiver_timeout

{% include links.html %}
