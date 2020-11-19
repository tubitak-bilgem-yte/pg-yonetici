---
title: "PostgreSQL Yedekleme"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 13, 2020
summary: "PostgreSQL Yedekleme"
sidebar: mydoc_sidebar
permalink: mydoc_postgresql_yedekleme.html
folder: mydoc
---

## PostgreSQL Yedekleme

PostgreSQL veritabanı üzerinde makine ya da disk hataları, yanlış sorgular, test ya da analiz için yedekleme yapmak faydalıdır.

## Yedekleme Yaklaşımları

- PostgreSQL servisini durdurarak mı ayaktayken mi?
- Belirli veritabanı ve tablolar mı hepsi mi?
- Belirli bir ana geri dönüş (PITR)?
- SQL dump alma
- Dosya sistemi seviyesinde yedekleme
- Sürekli arşivleme ve PITR
- Dış servisler ile yedekleme

### pg_dump ile Yedekleme

- SQL dump'ı alınır
- Dump dosyasında veritabanında değişiklik yaratan tüm işlemler bulunur. Amaç sıfırdan uygulandığında veritabanını dump’ın alındığı noktaya getirmektir.
- pg_dump tek veritabanı, pg_dumpall tüm veritabanları için yedek alır.

Varsayılan olarak localhost 5432 portundaki PostgreSQL’e bağlanıp dump alınır:

```sh
$ pg_dump pg02 > /tmp/pg02.sql
```

Dump dosyası varsayılan olarak düz text biçiminde çıkar, okunabilir:

```sh
$ cat /tmp/pg02.sql
```

pg_dump, psql’e verilen parametrelerin çoğunu kullanabilir:

```sh
$ pg_dump -h 10.0.0.58 -p 5432 -U begum hede > /tmp/hede.sql
```

pg_dumpall ile tüm veritabanlarını yedekleme:

```sh
$ pg_dumpall > /tmp/all.sql
```

- Bu yedeğin içinde veritabanları, roller, erişim hakları v.b. her şey bulunur

Alınan bir text dump’ı geri yükleme:

```sh
$ createdb pg02_yeni

$ psql pg02_yeni < /tmp/pg02.sql
```

psql ile geri yükleme sırasında veritabanı yoksa yaratılmaz, hata basar. Roller yoksa yaratılmaz, veritabanındaki nesnelerin sahiplikleri değişir. Roller için:

```sh
$ pg_dumpall -r > /tmp/roller.sql
```

Belirli bir tablonun dump’ını alma:

```sh
$ pg_dump -d pg02 -t hodo > /tmp/hodo.sql
```

Veritabanının belirtilen tablo hariç gerisinin dump’ını alma:

```sh
$ pg_dump -d pg02 -T hodo > /tmp/hodosız.sql
```

`-t` ve `-T` parametreleri aynı anda ve *wildcard* olarak da kullanılabir.

```sh
$ pg_dump -t 'detroit.emp*' -T detroit.employee_log mydb > db.sql
$ pg_dump -t "\"MixedCaseName\"" mydb > mytab.sql
```

Dump alırken sıkıştırma:

```sh
$ pg_dump pg02 | gzip > /tmp/pg02.sql.gz
```

Açıp geri yükleme:

```sh
$ gunzip -c /tmp/pg02.sql.gz | psql pg02_yeni

# veya

$ zcat /tmp/pg02.sql.gz | psql pg02_yeni
```

Dump alırken parçalama:

```sh
$ pg_dump pg02 | split -b 1k - /tmp/pg02.sql
```

Birleştirip geri yükleme:

```sh
$ cat /tmp/pg02.sql* | psql pg02_yeni
```

Veri olmadan sadece şema dump’ı alma:

```sh
$ pg_dump -s pg02 > /tmp/pg02_schema.sql
```

pg_dump ile düz metin dışında farklı biçimlerde de dump alınabilir: `custom (-Fc)`, `directory (-Fd)`, `tar (-Ft)`

```sh
$ pg_dump -Fc pg02 > /tmp/pg02.custom
```

Custom biçimde alınan yedeği psql anlamaz. `pg_restore` ile geri yüklenebilir. Custom yedeğin içinde erişim hakları da gelir:

```sh
$ createdb pg02_new
$ pg_restore -d pg02_new < /tmp/pg02.custom
```

- Restore aynı veritabanına yapılacaksa, veritabanı dolu olduğu için duplicate hataları ile karşılaşılır. `pg_restore -c` (clean) parametresi kullanılırsa nesneler oluşturulmadan önce temizleneceği için sorun olmaz.

Geri yükleme işlemi çok işlemli olarak da çalıştırılabilir (Sadece "custom" ya da "directory" biçimde alınmış dump’lar)

```sh
$ pg_restore -j 5 -c -d pg02 /tmp/pg02.custom
```

``--disable-triggers`` parametresi verilmediği sürece, triggerların da yedeği alınıp geri yüklendiğinde kullanılabiliyor. Geri yükledikten sonra kontrol amaçlı olarak triggerlarımızı bu şekilde listeleyebiliriz.

```sql
SELECT * FROM pg_trigger;
```

Sadece şemaların dumpını almak da mümkün

```sh
$ pg_dump -s database_name > db.sql
```

### Dosya Sistemi Seviyesinde Yedekleme

PostgreSQL veri dizinini kopyalayarak da yedek alabiliriz

```sh
$ tar cvfz /tmp/yedek.tar.gz /var/lib/pgsql/11/data/
```

Ancak bu yöntemin eksileri var!

- Tutarlı bir yedek olması için PostgreSQL servisinin kapalı olması gerekir.
- Servis kapalı değilse WAL logları lazım, o zaman PostgreSQL kendini kurtarabilir.
- Bu şekilde alınan yedekten tek tek veritabanı ya da tabloları geri yüklemek mümkün olmaz

### WAL Logları

- PostgreSQL, veritabanında değişiklik yapan tüm işlemleri WAL logları olarak `pg_wal` altına yazar.
- WAL logları asıl olarak çökme durumunda PostgreSQL’i tekrar tutarlı hale getirmek için tutulur.
- Son Checkpoint’ten itibaren WAL logları yeniden oynatılarak PostgreSQL otomatik düzeltilir.
- `wal_level`, WAL’a ne kadar bilginin yazıldığını belirler.
- Varsayılan olarak `wal_level = minimal`
- Minimal’ken WAL dosyaları yalnızca çökme sonrası kurtarma için gereken bilgileri içerir

### Sürekli Arşivleme ve PITR

- WAL logları, çökme kurtarması dışında şunları da yapabilmemizi sağlar:
  - Replikasyon
  - Düzenli olarak WAL loglarını başka makineye taşıyarak burada bir ılık yedek oluşturma
  - WAL logu sonuna bir yere kadar oynatıp istenen zamana dönebilme (PITR)
- Bunlar için WAL loglarını arşivlemek isteriz

- Arşivleme varsayılan olarak kapalı gelir
- Aktifleştirmek için
  - `archive_mode = on` yapılır
  - Arşivlemenin nasıl yapılacağı `archive_command` ile söylenir
  - Yeterince anlamlı bilgi biriktirmek için `wal_level` en az `replica` seviyesine çekilir
- Arşivleme yapıldıkça WAL logları `pg_wal` dizininden belirlenen dizine taşınır

Arşivleme ayarları örneği:

```sql
# vim /var/lib/pgsql/11/data/postgresql.conf

archive_mode = on
archive_command = 'test ! -f /mnt/server/archivedir/%f && \
                   cp %p /mnt/server/archivedir/%f'
wal_level = replica
```

Belirli aralıklarla veri dizininin yedeği alınmalı

- `pg_basebackup` ya da alelade `tar` komutu
- `pg_dump` ile alınan yedek üzerinde WAL oynatılamaz, çünkü içerisinde gerekli bilgiler yok

Geri dönüş prosedürü:

- PostgreSQL kapatılır ve veri dizini komple boşaltılır,
- Dizin yedeği (base backup) veri dizinine geri dönülür, sahipliği düzenlenir,
- `pg_wal` içeriği doluysa temizlenir,
- Arşivlenen logları geri dönecek komutun (**restore_command**) tanımlı olduğu bir *recovery.conf* dosyası hazırlanır.
- Tüm log yerine bir yere kadar log işletilecekse bu da aynı dosyada `recovery_target_time` şeklinde tanımlanır.
- PostgreSQL başlatılır, düzgün kurtarma olduysa recovery.conf, *recovery.done olur.

### Dış Servisler ile Yedekleme: Barman

- Barman, 2ndQuadrant ve topluluk tarafından geliştirilen bir özgür yazılım. [](http://www.pgbarman.org/)
- Sıcak yedek sunucular, PITR, tam ve arttırımlı yedekleme, çoklu ana ve yedek sunucu desteği
- Barman katalogu sayesinde yedek ve geri dönüşleri kolay yönetme

{% include links.html %}
