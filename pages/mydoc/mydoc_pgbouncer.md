---
title: "pgBouncer"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 24, 2020
sidebar: mydoc_sidebar
permalink: mydoc_pgbouncer.html
folder: mydoc
---


## PgBouncer

[pgBouncer](http://www.pgbouncer.org/) yüksek performans sunan, sunucuya yük getirmeyen bir connection pooling yazılımıdır. Skype tarafından geliştirilmeye başlanmıştır, şu anda daha geniş bir topluluk tarafından geliştirilmektedir:

PgBouncer, diğer pooling yazılımlarına göre daha az bellek harcaması ile dikkat çeker. Bağlantı başına sadece 2kB kadar bellek harcanır. Yeniden başlama ve güncelleme sırasında client bağlantılarını öldürmez bu da önemli bir avantajdır. Pgpool’a göre tek “dezavantajı” pgBouncer Fedora ve EPEL depolarında, ayrıca [](https://yum.postgresql.org) adresindeki depoda bulunmaktadır. Debian/Ubuntu için de [](http://apt.postgresql.org) adresindeki depoya bakabilirsiniz. Paket kurulumlarında ayar dosyası olan `pgbouncer.ini` dosyası */etc/pgbouncer* dizini altında yaratılır. Bu dosyayı düzenlemek gerekir.

`[database]` bölümüne pgbouncer'ı hangi veritabanları için kullanmak istediğinizi belirtiniz. Örnek olarak sunucudaki mgm ve başka  bir sunucudaki pagila veritabanlarını verelim:

```text
mgm = host=/tmp port=5432 dbname=mgm
pagila2 = host=diger.sunucu.ipsi port=5432 dbname=pagila
```

Burada *pagila2* adını pagila veritabanı için kullandık. Yani veritabanının adı aslında pagila, ama biz farklı bir ad vererek bağlanabiliyoruz:

```sql
$ psql pagila -p 6432
psql: ERROR:  No such database 
$ psql pagila2 -p 6432
pagila2=#
```

Ardından `[pgbouncer]` bölümünde de aşağıdaki değişiklikleri yapınız:

```text
listen_addr = * (ya da istediğiniz doğru bir harici IP)
auth_type = md5
auth_file= /etc/pgbouncer/userlist.txt
server_reset_query = DISCARD ALL;
```

Ayrıca `max_client_conn` ve `default_pool_size` değişkenlerini de ayarlayabilirsiniz.

Burada önemli olan parametrelerden birisi `pool_mode`’dur. Öntanımlı değeri olan **session**, önerilen değerdir. session değerinde, o pool o session boyunca açık tutulur. **transaction** ve **statement** değerlerinde, o pool sırası ile transaction ya da SQL statement sonrasında bırakılır. Bu da performans sorununa neden olur. Ayrıca session, PostgreSQL’in tüm özelliklerini (SET/RESET, LISTEN/NOTIFY, PREPARE vs) kullanabileceğiz değerdir. Transaction pooling’de bunların önemli bir kısmı kullanılamaz.

pgBouncer’ı `systemctl start pgbouncer` ile başlatabilir, `systemctl stop pgbouncer` ile durdurulabilir, `systemctl restart pgbouncer` ile yeniden başlatılabilir. Log'unu */var/log/pgbouncer/pgbouncer.log* içine yazar.

Pgbouncer yönetimini PgBouncer’ın kendi içinden yapabilirsiniz:

```sql
$ psql pgbouncer -p 6432
pgbouncer=# show help;
NOTICE:  Console usage 
DETAIL:  
    SHOW HELP|CONFIG|DATABASES|POOLS|CLIENTS|SERVERS|
VERSION 
    SHOW FDS|SOCKETS|ACTIVE_SOCKETS|LISTS|MEM 
    SET key = arg 
    RELOAD 
    PAUSE [<db>] 
    SUSPEND 
    RESUME [<db>] 
    SHUTDOWN 
pgbouncer=#
```

Örneğin PAUSE ile o veritabanına yapılacak yeni bağlantıları geçici olarak durdurabilirsiniz, RESUME ile devam ettirebilirsiniz.

```sql
pgbouncer=# PAUSE mgm;
```

SHOW DATABASES; ile de pool bilgilerini görebilirsiniz:

```sql
pgbouncer=# SHOW DATABASES;
    name      |   host    | port | database  | force_user | pool_size 
    -----------+-----------+------+-----------+------------+-----------  
    mgm       | 127.0.0.1 | 5432 | mgm       |            |        20  
    pagila    | 127.0.0.1 | 5432 | pagila    |            |        20  
    pgbouncer |           | 5450 | pgbouncer | pgbouncer  |         2 
(3 rows) 
```

İstatistik de alabilirsiniz:

```sql
pgbouncer=# show stats;
-[ RECORD 1 ]----+----------
database         | mgm
total_requests   | 6
total_received   | 3192 
total_sent       | 4798 
total_query_time | 25437 
avg_req          | 0 
avg_recv         | 32 
avg_sent         | 48 
avg_query        | 4239 
-[ RECORD 2 ]----+----------
database         | pgbouncer 
total_requests   | 1 
total_received   | 0 
total_sent       | 0
total_query_time | 0 
avg_req          | 0 
avg_recv         | 0 
avg_sent         | 0 
avg_query        | 0
```

Üstte bahsettiğim gibi pgBouncer'ı kapatıp açarken bağlantılar kapatılmaz. Bunu aşağıdaki şekilde yapabilirsiniz:

```bash
pgbouncer -R -d config.ini
```

{% include callout.html content=" `-R`, online restart yapan parametredir. `-d`, daemon olarak çalıştırır." type="primary" %}

{% include links.html %}