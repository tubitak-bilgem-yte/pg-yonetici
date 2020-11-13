---
title: "PostgreSQL Sürüm Yükseltme"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 13, 2020
summary: "PostgreSQL Sürüm Yükseltme"
sidebar: mydoc_sidebar
permalink: mydoc_postgresql_surum_yukseltme.html
folder: mydoc
---

## PostgreSQL Sürüm Yükseltme

10 sürümü önceside PostgreSQL’de majör sürümler ilk iki basamak (Örn: 9.6), üçüncü basamak minör sürümü ifade ediyordu (Örn: 9.6.6). 10 sürümü ve sonrasıda PostgreSQL’de majör sürümler tek basamak (Örn: 11), ikinci basamak minör sürümü ifade eder (Örn: 11.5).
Minör-majör sürümlerde, minör sürüm güncellemelerinde veri formatlarında asla değişme olmaz. Ancak majör sürümlerde veri formatı değişebilir, bundan ötürü belirli bir güncelleme prosedürü izlenmelidir.

### Minör Sürüm Güncelleme

Sadece çalışabilir dosyalar değişir ve PostgreSQL yeniden başlatılır.Paket yönetim sistemi ile:

```shell
yum update postgresql11*
systemctl restart postgresql-11
```

### Majör Sürüm Güncelleme

```shell
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Yeni PostgreSQL sürümü kurulur:

```shell
yum install postgresql12-server postgresql12 postgresql12-contrib
```

Yeni sürümün ilklendirmesi yapılır:

```shell
/usr/pgsql-12/bin/postgresql-12-setup initdb
```

Eski PostgreSQL sürümü durdurulur ve açılıştan kaldırılır:

```shell
systemctl stop postgresql-11
systemctl disable postgresql-11
```

Eski sürümün veritabanı yeni sürümün veritabanına aktarılır:

```shell
su - postgres
$ /usr/pgsql-12/bin/pg_upgrade -b /usr/pgsql-11/bin \
    -B /usr/pgsql-12/bin -d /var/lib/pgsql/11/data \
    -D /var/lib/pgsql/12/data
$ exit
```

Öntanımlıları değiştirilen veritabanı ayar dosyaları (postgresql.conf, pg_hba.conf, vs) karşılaştırılarak elle yeni sürüme aktarılır. Yeni sürüm PostgreSQL başlatılır ve servis açılışta otomatik çalışacak biçimde ayarlanır:

```shell
systemctl start postgresql-12
systemctl enable postgresql-12
```

Veritabanının aktarımı sonrasında sistemin istatistiklerinin oluşturulması için `ANALYZE` çalıştırılır:

```shell
su - postgres
$ ./analyze_new_cluster.sh
$ exit
```

Eski veri dizini ve eski PostgreSQL paketleri kaldırılabilir (ya da yedek olarak saklanabilir):

```shell
rm -rf /var/lib/pgsql/11/data/*
yum remove postgresql11-server postgresql11 postgresql11-contrib
```

{% include links.html %}
