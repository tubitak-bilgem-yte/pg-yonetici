---
title: "PgPool-II"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 19, 2020
summary: "PgPool-II"
sidebar: mydoc_sidebar
permalink: mydoc_pgpool.html
folder: mydoc
---

## PgPool II

PgPool istemci ile sunucu arasına giren bir ara katman aracıdır. Yetenekleri:

- Bağlantı Havuzu (Connection Pooling)
- Okuma Yükü Dengeleme
- Otomatik Failover (Streaming Replikasyon’da)
- Bağlantı Sınırlandırma: Belli bir miktarın üzerinde PostgreSQL’e bağlantı akmasını ve onun o bağlantıları geri çevirerek yük yaşamasını engelliyor.
- Paralel Sorgulama: Büyük bir veri sorgusunu farklı sunucularda farklı parçalarını yaptırıp birleştirme
- Aktif-Pasif Çalışma
- Paralel Sorgulama

### Bağlantı Havuzu

- Sunucuya yapılan bağlantıları saklıyor
- Tekrar aynı kullanıcı aynı veritabanına bağlantı yapmaya çalıştığında bağlantıyı tekrar kullanıyor
- Performans arttırıyor
- PostgreSQL’in kendisinin böyle bir yeteneği yok
- Daha hafif bir çözüm olan PgBouncer, yalnızca bağlantı havuzu oluşturuyor. Ek işlevleri yok. PgPool’un diğer yeteneklerine gereksinim duyulmuyorsa doğrudan PgBouncer kullanılabilir.

### Yük Dengeleme

- Okuma yükünü standby’lara dağıtıyor
- Oturum temelli yük dağıtıyor, sorgu temelli değil.
- Bir kullanıcı tarafından bir oturum açıldığında, o oturum boyunca tüm sorgular aynı sunucuya gönderiliyor.
- `weight` parametresine göre yük ağırlıklarını belirliyor
- LRU ( Least Recently Used ) ya da benzeri bir algoritma desteği yok

### Aktif-Pasif

- PgPool’un kendinden aktif-pasif çalışabiliyor
- Watchdog alt süreci bunu yapıyor
- PgPool’a gelen servis isteklerini izliyor
- Diğer Watchdog’larla haberleşiyor
- Aktif serviste problem olunca, bir diğeri aktif alıp "Virtual IP"yi üzerine çekiyor.
- Geri dönen sunucu standby olarak dönüyor.

### Kurulum

PGDG paket deposu ekli değilse eklenir:

```text
# yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Pgpool paketi kurulur:

```text
# yum install pgpool-II-11
```

Ayar dosyası örnek dosyadan oluşturulur:

```text
# cp -a /etc/pgpool-II-11/pgpool.conf.sample /etc/pgpool-II-11/pgpool.conf
```

- Pgpool paketinin farklı PostgreSQL majör sürümleri için farklı sürümleri bulunur. Paket isimleri, servis adları ve dizinleri majör sürüme göre farklılık gösterir.

- Uygulamayı bağlantı havuzu ile yük dengeleme için yapıyoruz (en yaygın kullanılan özellikleri). Diğer işlevler ayar dosyaları kurcalanarak gerçekleştirilebilir.

### Pgpool Ayarlar

Bağlantı havuzu ve yük dengeleme için pgpool.conf dosyasında şu ayarlar düzenlenir:

```text
listen_addresses = '*'
port = 5432

load_balance_mode = on
master_slave_mode = on
master_slave_sub_mode = 'stream'
sr_check_user = postgres
delay_threshold = 10000000

connection_cache = on
reset_query_list = 'ABORT; DISCARD ALL'
```

- pgpool bir PostgreSQL servisi ile aynı sunucuda çalıştırılıyorsa, PostgreSQL servisinin ya da pgpool’un farklı bir porta çekilmesi gerekir.

pgpool.conf dosyasına kümedeki sunucu bilgileri eklenir:

```text
backend_hostname0 = '192.168.56.101'
backend_port0 = 5432
backend_weight0 = 1
backend_data_directory0 = '/var/lib/pgsql/11/data'
backend_flag0 = 'ALLOW_TO_FAILOVER'

backend_hostname1 = '192.168.56.102'
backend_port1 = 5432
backend_weight1 = 1
backend_data_directory1 = '/var/lib/pgsql/11/data'
backend_flag1 = 'ALLOW_TO_FAILOVER'
```

- İstenilen kadar sunucu kümeye blok tanımları halinde eklenebilir.
- 192.168.56.101: Primary, 192.168.56.102: Standby, 192.168.56.105: PGPool

### Kullanım

Servis başlatılır ve açılışta otomatik çalışacak biçimde ayarlanır:

```text
# systemctl enable pgpool-II-11
# systemctl start pgpool-II-11
```

Birden fazla istemci ile pgpool IP’sine bağlanılır ve bağlantıları farklı sunuculara dağıtıldığı görülür:

```text
$ psql -U postgres -h 192.168.56.105
postgres=# select inet_server_addr();
 inet_server_addr
------------------
 192.168.56.102
(1 row)
```

Yazma sorgularının standby bir sunucuya bağlanıldığı zaman da primary’e uygulandığı görülebilir:

```text
$ psql -U postgres -h 192.168.56.105
postgres=# select inet_server_addr();
 inet_server_addr
------------------
 192.168.56.102
(1 row)

postgres=# CREATE DATABASE deneme;
CREATE DATABASE

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 deneme    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
```

### Log

- Öntanımlı `/var/log/messages` dosyasında log bilgileri görülebilir.
- `log_per_node_statement` → her bir sorgu ve hangi sunucuya gittiği bilgisi loglanır.

```text
Mar 24 00:25:33 pgpool11 pgpool: 2018-03-24 00:25:33: pid 4087: LOG:  DB node id: 1 backend pid: 2299 statement: select inet_server_addr();
Mar 24 00:25:43 pgpool11 pgpool: 2018-03-24 00:25:43: pid 4087: LOG:  DB node id: 0 backend pid: 2610 statement: CREATE DATABASE dene;
```

### PCP ( Pgpool Child Process ) Kullanımı

- PgPool çocuk süreçleri yönetmek için bir servis
- Öntanımlı olarak 9898 portunu dinliyor.
- Bir parolanın md5’i hash’i üretilir:

```text
# pg_md5 parola
8287458823facb8ff918dbfabcd22ccb
```

Hashlenmiş parola ve kullanıcı adı ayar dosyasına yazılır (vim /etc/pgpool-II-11/pcp.conf):

```text
pcpadmin:8287458823facb8ff918dbfabcd22ccb
```

- `pg_md5` komutunda "parola" yerine kullanılmak istenen parola girilir.
- `pcpadmin` yerine herhangi bir kullanıcı adı da yazılabilir.

Çeşitli örnekler:

```text
# pcp_node_count -U pcpadmin
2
# pcp_node_info 0 -U pcpadmin
192.168.56.101 5432 2 0.500000 up primary
# pcp_node_info 1 -U pcpadmin
192.168.56.102 5432 2 0.500000 up standby
# pcp_pool_status -U pcpadmin
```

{% include links.html %}
