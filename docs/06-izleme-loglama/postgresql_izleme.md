---
title: "PostgreSQL’de İzleme"
parent: Veritabanı İzleme ve Loglama
layout: default
nav_order: 3
--- 

PostgreSQL MVCC ve VACUUM
mydoc_mvcc_vacuum.html

## PostgreSQL’de İzleme

PostgreSQL sunucusu üzerinde gözetlenmesi gereken temel konular:

- Servis ayakta mı?
- RAM / CPU / Disk durumu
- Anlık bağlantı sayısı
- Uzun süren sorgular
- Tablo ve İndex erişim istatistikleri
- Cache hit oranı
- Kilitlenme miktarı
- Replikasyon varsa standby gecikmesi
Anlık görüntüleme, sorgu istatistikleri, dış servisler ile anlık izleme ve alarm üretme, dış servisler ile metrik toplama başlıca izleme biçimleridir.

### Unix Komutları ile Anlık İzleme

ps komutu sistemdeki mevcut süreçlerin bilgilerini gösterir.

```sql
# ps auxf | grep postgres
```

Ana süreç, arka plan süreçleri ve alt süreçler ps çıktısında görülen süreçlerdir.

```sh
postgres   617  0.0  3.4 297112 17436 ?   Ss   06:14   0:00 /usr/pgsql-12/bin/postmaster -D /var/lib/pgsql/12/data/
postgres   631  0.0  0.4 149372  2040 ?   Ss   06:14   0:00  \_ postgres: logger
postgres   634  0.0  0.4 297112  2068 ?   Ss   06:14   0:00  \_ postgres: checkpointer
postgres   635  0.0  0.6 297244  3352 ?   Ss   06:14   0:00  \_ postgres: background writer
postgres   636  0.0  1.2 297112  6252 ?   Ss   06:14   0:00  \_ postgres: walwriter
postgres   637  0.0  0.6 297664  3304 ?   Ss   06:14   0:00  \_ postgres: autovacuum launcher
postgres   638  0.0  0.4 151624  2324 ?   Ss   06:14   0:00  \_ postgres: stats collector
postgres   639  0.0  0.5 297664  2808 ?   Ss   06:14   0:00  \_ postgres: logical replication launcher
```

- Alt süreçler her bir PostgreSQL bağlantısı için açılır.

**top** komutu ile süreçlerin ne kadar bellek ve işlemci harcadıkları görülür:

```text
# top
```

- Bellek kullanımında paylaşılan bellek ve toplam sürece ayrılmış bellek de görülür. Tek sürecin bellek kullanımı için gerçeğe en yakın değer RES kolonudur.

**netstat** komutu ile PostgreSQL süreçlerine bağlantı bilgileri gözlenir:

```text
# netstat -apl --numeric-hosts | grep postgres
```

PostgreSQL verilerinin diskte kapladığı toplam boyutu **du** komutu ile görebiliriz:

```sql
# du -sh /var/lib/pgsql/11/data
```

- Bu miktar veritabanlarındaki tabloların içindeki gerçek veri miktarından fazla görünebilir.

### PostgreSQL Komutları ile Anlık İzleme

Statistics Collector veritabanı üzerinde; tablo ve indexlere erişim sayısı (disk ve tek satır bazında), tablolardaki satır sayısı, vakum ve analiz eylemlerinin bilgileri, fonksiyonların çağrılma sayısı ve çalışma süreleri gibi sistem aktivitelerini izler.

Statistics Collector’ün ayarları ``postgresql.conf`` dosyasından yapılır:

```sql
#track_activities = on
#track_counts = on
#track_io_timing = off
#track_functions = none                 # none, pl, all
#track_activity_query_size = 1024       # (change requires restart)
#stats_temp_directory = 'pg_stat_tmp'
```

*Statistics Collector*’ün topladığı veritabanına kaç bağlantı var, transaction miktarları, toplam diske yazma, toplam cache’ten okuma, eklenen silinen satır sayısı verileri **pg_stat** ile başlayan tablolarda tutulur.

```sql
SELECT * FROM pg_stat_database;
```

Indexler ve kullanım durumları için:

```sql
SELECT * FROM pg_stat_user_indexes;
```

Veritabanlarındaki mevcut sorgular, başlama zamanları ve durumları için:

```sql
SELECT * FROM pg_stat_activity;
```

- Mevcut sorgularla ilgili bilgiler içeren kolonları sadece *superuser* yetkililer görebilir.

Mevcut sorgular ne zamandır çalıştığını görmek için:

```sql
SELECT pid,datname,usename,
now() - query_start AS runtime,
state,query FROM pg_stat_activity;
```

Hangi alt süreçlerde hangi sorgular çalıştığını görmek için:

```sql
SELECT pg_stat_get_backend_pid(s.backendid) AS procpid,
pg_stat_get_backend_activity(s.backendid) AS current_query
FROM (SELECT pg_stat_get_backend_idset() AS backendid) AS s;
```

Bir sorguyu durdurmak için:

```sql
select pg_cancel_backend(<sürecin pidi>)
```

Normal durmuyorsa öldürmek için:

```sql
select pg_terminate_backend(<sürecin pidi>)
```

Bir veritabanına olan tüm bağlantıları kapatmak için:

```sql
bash-4.2$ psql -c "select pg_terminate_backend(pid) from
pg_stat_activity where datname='ulkeler';"
 pg_terminate_backend
_____________________
 t
(1 row)
```

Kilitlenen nesneleri görmek için:

```sql
SELECT * FROM pg_locks;
```

Aynı anda iki işlemin birden aynı nesneyi değiştirmesi istenmeyeceği için bir transaction tarafından kilit konmuş nesne kilidi çözülene kadar beklenir.

Karşılıklı iki işlemin çakışması ve birbirinin kilidinin açılmasını beklediği duruma "deadlock" denir ve bu durum ancak işlemlerden birinin iptal edilmesiyle çözülebilir. Deadlock durumunun tespiti için PostgreSQL’in öntanımlı olarak beklediği süre (deadlock_timout) 1 sn’dir. Bu değer yoğun sistemlerde arttırılmalıdır. Aksi durumda hem çok sık bu kontrolün yapılması performans sorunu yaratacak hem de gözetlerken gereği olmayan uyarılar görmemize neden olacaktır.

Bir veritabanının boyutunu görme:

```sql
SELECT pg_size_pretty(pg_database_size('alfresco'));
```

Bir tablonun boyutunu görme:

```sql
\c alfresco
SELECT pg_size_pretty( pg_total_relation_size('alf_node_properties'));
```

Tablonun satır sayısını görme:

```sql
SELECT count(*) AS exact_count FROM alf_node_properties;
```

Yaklaşık satır sayısını görme:

```sql
SELECT reltuples::bigint AS estimate FROM pg_class
    where relname='alf_node_properties';
```

En büyük veritabanları görmek için:

```sql
SELECT
   pg_database.datname AS "database_name",
   pg_database_size(pg_database.datname)/1024/1024 AS size
   FROM pg_database
   ORDER by size DESC;
```

Bir veritabanındaki en büyük 10 nesneyi görmek için:

```sql
SELECT
  relname AS objectname,
  relkind AS objecttype,
  reltuples AS "#entries", pg_size_pretty(relpages::bigint*8*1024) AS size
  FROM pg_class
  ORDER BY relpages DESC
  LIMIT 10;
```

### Sorgu İstatistikleri Çıkarma

pg_stat_statements tüm veritabanları için sorgu istatistiklerini toplar. Bu istatistiklere ulaşmak için istenen veritabanında eklenti yaratılmalıdır.

**pg_stat_statements** kurulumu:

```sql
# yum install postgresql11-contrib

# vim /var/lib/pgsql/11/data/postgresql.conf
shared_preload_libraries = 'pg_stat_statements'

# systemctl restart postgresql-11
```

Veritabanı istatistiklerini görme:

```sql
# su - postgres
$ psql -d zabbix

zabbix=# CREATE EXTENSION pg_stat_statements;
CREATE EXTENSION

zabbix=# SELECT LEFT(query,50) AS query,
       calls, total_time, rows, shared_blks_hit
FROM pg_stat_statements;
```

Diğer Bazı PostgreSQL Eklentileri

- ``pg_stat_plans``
  - pg_stat_statements eklentisinin daha da genişletilmişidir.
  - Sorgu planlarını da kaydeder.
- ``pgstattuple``
  - Tablo ve indexler için canlı ve silinmiş satırları, boyutları v.b. istatistiğe döker
- ``pg_buffercache``
  - Tablo ve indexlerin paylaşılan bufferlarını, cache’te tuttukları sayfaları v.b. gösterir.

### Bazı İzleme Programcıkları

#### pg_activity

Anlık sorguları ve tükettikleri kaynakları "htop" benzeri bir komut satırı arayüzünde gösterir. Kurulumu:

#### pg_view

Yine pg_activity’e benzer olup PostgreSQL odaklı top ve uptime benzeri bir programcıktır. Kurulum:

```sql
# yum install pg_view
```

### Dış Servisler ile PostgreSQL İzleme

#### pgAdmin

Veritabanları istatistikleri ve grafikleri izlenebilir.

#### Zabbix

Zabbix, gelişkin bir merkezi izleme sunucusudur. İzleme şablonları ve sunucu tarafına kurulan connector’lar ile detaylı izleme yapar ve beklenmeyen durumlarda eposta, sms, v.b. yöntemlerle bildirimde bulunabilir. Eskiye dönük verileri biriktirme ve grafikleştirme gibi özelliklere de sahiptir.

#### Nagios

Nagios da en çok kullanılan merkezi izleme sunucularından biridir. Eklentiler ile detaylı izleme, alarm ve metrik toplama işlemlerini yapabilir. Grafikleştirme yeteneği kazandırmak için ek programa ihtiyaç duyar.

{% include links.html %}
