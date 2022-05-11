---
title: "Logical Replication"
parent: Yüksek Erişilebilirlik
layout: default
nav_order: 4
---

## Logical Replication

- Dosya sistemindeki dosyaların farkı yerine veritabanı/tablo değişimlerinin farkının aktarılmasıdır.
- Publisher/Subscriber yapısında çalışır,
- Subscriber sunucular başka sunucular için publisher olabilir.
- Tüm sunucu yerine belirli bir veritabanı ya da tablonun replikasyonunu yapabilir.
- Çok sayıda veritabanını tek bir veritabanında birleştirebilir.
- Bir veritabanının belirli parçalarını farklı sunuculara dağıtabilme.
- Farklı PostgreSQL majör sürümleri arasında replikasyon yapılabilmesi
- Her bir değişimin sonucu triggerların uygulanabilmesi

### Logical Replication - Kısıtlamalar

- Şema ve DDL komutları replike edilmez. Şema değişiklikleri her iki tarafta elle çalıştırılmalıdır.
- Sekans verisi replike edilmez. Serial tipindeki bir kolona yazılı veriler replike edilsir ancak sekans hedef sunucudaki başlangıç değerinde kalır.
- TRUNCATE komutları replike edilmez, yerine DELETE kullanılmalıdır. (Kazara truncate çalıştırmamak için tablolardan truncate yetkisini kaldırmak iyi fikir olabilir)
- Büyük objeler (large object) replike edilmez.
- Sadece temel tablolardan temel tablolara veri replike edilebilir. Her iki tarafta da view, materialized view, partition root table, foreign table gibi tablolar replike edilemez.

### Logical Replication - Çalışma Biçimi

- Bir tablonun replikasyonu publisher (kaynak) veritabanı üzerinde verinin snapshot’ının alınır.
- Snapshot subscriber (hedef) üzerine kopyalanır.
- Publisher tarafındaki tüm değişiklikler gerçekleştikleri anda subsciber tarafına aktarılır.
- Subscriber sunucular değişiklikleri publisher ile aynı sıra ile uygular, böylece bir transactional tutarlılık korunmuş olur.

### Logical Replication - Publications

- Şu an sadece tablolar publish edilebilir.
- Her obje açık ismi ile eklenmek zorundadır.
- `CREATE PUBLICATION` komutu ile yaratılır.
- Tüm tabloları eklemek için ALL TABLES kısayolu kullanılabilir.
- Öntanımlı olarak tüm değişiklikler publish edilir, INSERT, UPDATE, DELETE operasyonlarından istenenler filtrelenebilir.

### Logical Replication - Subscriptions

- `CREATE SUBSCRIPTION` komutu ile yaratılır, `ALTER SUBSCRIPTION` komutu ile durdurup başlatılır, `DROP SUBSCRIPTION` komutu ile silinir.
- Subscription silinip yeniden yaratıldığında senkronizasyon bilgisi silindiği için tüm veri yeniden senkronize edilir.
- Tablolar tam tablo isimleri (fully qualified) ile replike edilir. Tabloları subscriber tarafında farklı isimle adlandırmak desteklenmez.
- Bir tablonun içindeki kolonlar da isimleri ile replike edilir. Hedef tabloda kolonların sıraları farklı olabilir, ana veri tipleri aynı olmalıdır. Hedef tablodaki kaynak tabloda olmayan fazladan kolonlar öntanımlı değerleri ile doldurulur.

### Logical Replication - Yetkiler

- Publication yaratacak kullanıcının veritabanında CREATE yetkisi bulunmalıdır.
- Publication’a tablo ekleyecek kullanıcı tablonun sahibi olmalıdır.
- Tüm tabloları otomatik olarak publish etmek için kullanıcı superuser olmalıdır.
- Sadece *superuser* subscription yaratabilir.
- Subsciption uygulama süreci hedef veritabanı içinde superuser yetkileri ile çalışır.
- Replikasyon bağlantısında kullanılan rol REPLICATION niteliğine veya superuser yetkisine sahip olmalıdır.
- Yetki kontrolleri sadece replikasyon bağlantısı başladığında kontrol edilir. Bundan sonra publisher veya subscriber tarafında yeniden yetki kontrolü yapılmaz.

### Logical Replication - Çakışmalar

- Replikasyon ile gelen veri hedef tablodaki kısıtlamalara (constraint) aykırı ise replikasyon çakışma (conflict) hatası verir ve durur, elle müdahale edilip çakışma ortadan kaldırılmadan yeniden başlamaz. Çakışma ile ilgili ayrıntılı bilgi subsciber sunucunun loguna yazılır.
- Çakışmayı düzeltmek için iki yöntem vardır:
  - Subscriber veritabanı üzerinde çakışmaya neden olan veriyi düzeltmek
  - `pg_replication_origin_advance()` fonksiyonu ile çakışma yaratan transaction’ı uygulamadan atlamak
- UPDATE ve DELETE cümlelerini replike ederken hedef tabloda silinecek veya güncellenecek verinin bulunamaması hata olarak değerlendirilmez, bu değişiklikler atlanır ve replikasyon devam eder.

### Logical Replication - Adımlar (Kaynak)

postgresql.conf ayarları ve Publication oluşturma:

```text
wal_level = logical
listen_addresses = '*'
```

Replikasyon için kullanıcı oluşturulmalı:

```text
postgres=# CREATE USER repuser WITH LOGIN PASSWORD 'XXXXXX' replication;
```

Replike etmek istediğimiz "reptest" veritabanının "foo" tablosu olsun.

```text
postgres=# \c reptest
reptest=# CREATE PUBLICATION testpub FOR TABLE foo;
CREATE PUBLICATION
```

Replikasyon kullanıcısının hedef taraftan erişebilmesi için pg_hba.conf ayarları:

```shell
host all repuser 192.168.56.102/32 md5
```

Ayrıca, replike edilecek tablolarda replikasyon kullanıcısının SELECT yetkisi olmalıdır:

```text
postgres=# \c reptest
reptest=# GRANT SELECT ON foo to repuser;
```

### Logical Replication - Adımlar (Hedef)

postgresql.conf ayarları

```shell
listen_addresses = '*'
```

Logical replication ile şemalar replike edilmiyor. Önce veritabanı ve tablo(lar) oluşturulmalı:

```text
postgres=# CREATE DATABASE reptest;
postgres=# \c reptest
reptest=# CREATE TABLE foo (foo text PRIMARY KEY, quux text);
```

Subscription oluşturmak:

```text
reptest=# CREATE SUBSCRIPTION testsub CONNECTION 'dbname=reptest host=192.168.56.101 user=repuser password=XXXXXX' PUBLICATION testpub;
NOTICE:  created replication slot "testsub" on publisher
CREATE SUBSCRIPTION
```

PostgreSQL log’unda şunu görmeliyiz:

```text
2018-01-20 13:18:48.051 UTC [1713] LOG:  logical replication apply worker for subscription "testsub" has started
2018-01-20 13:18:48.061 UTC [1714] LOG:  logical replication table synchronization worker for subscription "testsub", table "foo" has started
2018-01-20 13:18:48.081 UTC [1714] LOG:  logical replication table synchronization worker for subscription "testsub", table "foo" has finished
```

Subscription durdurmak:

```text
reptest=# ALTER SUBSCRIPTION testsub DISABLE;
ALTER SUBSCRIPTION
```

Durdurunca PostgreSQL log’unda görümesi beklenen:

```text
2018-01-20 13:21:54.282 UTC [1713] LOG:  logical replication apply worker for subscription "testsub" will stop because the subscription was disabled
```

Subscription’ı yeniden başlatmak:

```text
reptest=# ALTER SUBSCRIPTION testsub ENABLE;
ALTER SUBSCRIPTION
```

Başlatınca PostgreSQL log’unda görülmesi beklenen:

```text
2018-01-20 13:23:08.330 UTC [1724] LOG:  logical replication apply worker for subscription "testsub" has started
```

Subscription’ı kalıcı olarak durdurmak:

```text
reptest=# DROP SUBSCRIPTION testsub;
NOTICE:  dropped replication slot "testsub" on publisher
DROP SUBSCRIPTION
```

Kalıcı olarak durdurunca PostgreSQL log’unda görülmesi beklenen:

```text
2018-01-20 13:24:34.447 UTC [1724] FATAL:  terminating logical replication worker due to administrator command
2018-01-20 13:24:34.452 UTC [1588] LOG:  worker process: logical replication worker for subscription 16396 (PID 1724) exited with exit code 1
```

İzleme:

```text
reptest=# SELECT * FROM pg_stat_subscription;
 subid | subname | pid  | relid | received_lsn |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time
-------+---------+------+-------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------
 16397 | testsub | 1787 |       | 0/16B36F0    | 2018-03-20 13:34:13.159409+00 | 2018-03-20 13:34:13.159548+00 | 0/16B36F0      | 2018-03-20 13:34:13.159409+00
(1 row)
```

{% include links.html %}
