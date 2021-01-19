---
title: "PgPool-II"
tags: [PostgreSQL]
keywords: postgres, pgpool, connection pooling
last_updated: December 24, 2020
sidebar: mydoc_sidebar
permalink: mydoc_pgpool.html
folder: mydoc
---

## PgPool - II

Pgpool, PostgreSQL için yazılmış ilk açık kaynak kodlu connection pooler yazılımıdır. İlk başta sadece connection pooling yapması için tasarlanmış, ancak sonrasında başka özellikler de gelmiştir: Load balancing, replikasyon ve fazla bağlantıları kuyruğa alma. Bir başka özellik olan "online recovery" koddan kaldırılmıştır. Replikasyon da şu anda etkin olarak kullanılmamaktadır ve ileride kaldırılabilir. Uzun yıllar boyunca bu kadar fazla özellik eklenmesi koddaki kaliteyi azaltmış, ancak 2012 yılından sonra EnterpriseDB'nin koda doğrudan katkı vermeye başlaması ile kod kalitesi tekrar artmaya başlamıştır. Pgpool, PostgreSQL'in şu andaki defacto load balancer'ıdır.

Pgpool özelliklerini kısaca inceleyelim:

**`Connection pooling`**: Klasik connection pooling özelliğidir.

**`Load balancing`**:   Master sunucu ile replikasyon sunucusu ya da sunucularını kullanarak SELECT sorgularının sunucular  arasında dengelenmesini sağlar. Yükün hangi sunucuya hangi oranda aktarılabileceğini `backend_weigh[0,1,...]` parametresi ile belirleyebilirsiniz. Burada oranı belirleyecek teknik parametre sunucuların donanım özellikleri ve master sunucuya yapılacak yazma miktarıdır. Eğer replikasyon için streaming replication kullanıyorsanız ve replikasyon async ise, gecikme miktarı sizin belirleyeceğiniz byte'den fazla olursa load balancing durdurulur, bu da olası replikasyon gecikmeleri nedeniyle yanlış sonuç getirilmesinin önüne geçilmiş olur.

**`Fazla bağlantıları kuyruğa alma (limiting exceeding connections)`**: PostgreSQL sunucusu, normal şartlar altında `max_connections`  değerine gelince clientlara hata verir. Pgpool ise, belirlenen sayı aşılınca bu bağlantıları kendi içinde sıraya alır, böylece clientlar hata almazlar. Diğer clientlar işlemlerini bitirince, kuyrukta bekleyen bağlantılar sırasıyla PostgreSQL   sunucusuna aktarılırlar.

{% include callout.html content=" Load balancing modu ön tanımlı olarak kapalıdır. Connection poolerve fazla bağlantıları kuyruğa alma özellikleri ise açık gelirler." type="primary" %}

{% include callout.html content=" Pgpool'in bir başka özelliği de Watchdog'dur. Bu özellik sayesinde birden fazla Pgpool sunucusunun birbirini yedekleyecek şekilde kurulması sağlanır." type="primary" %}

Tüm Yetenekleri:

- Bağlantı Havuzu (Connection Pooling),
- Okuma Yükü Dengeleme,
- Otomatik Failover (Streaming Replikasyon’da),
- Bağlantı Sınırlandırma: Belli bir miktarın üzerinde PostgreSQL’e bağlantı akmasını ve onun o bağlantıları geri çevirerek yük yaşamasını engelliyor.
- Paralel Sorgulama: Büyük bir veri sorgusunu farklı sunucularda farklı parçalarını yaptırıp birleştirme,
- Aktif-Pasif Çalışma,
- Paralel Sorgulama,

**Bağlantı Havuzu**:

- Sunucuya yapılan bağlantıları saklar.
- Tekrar aynı kullanıcı aynı veritabanına bağlantı yapmaya çalıştığında bağlantıyı tekrar kullanır.
- Performans arttırır.
- PostgreSQL’in kendisinin böyle bir yeteneği yoktur.
- Daha hafif bir çözüm olan PgBouncer, yalnızca bağlantı havuzu oluşturuyor. Ek işlevleri yok. PgPool’un diğer yeteneklerine gereksinim duyulmuyorsa doğrudan PgBouncer kullanılabilir.

**Yük Dengeleme**:

- Okuma yükünü standby’lara dağıtır.
- Oturum temelli yük dağıtıyor, sorgu temelli değil.
- Bir kullanıcı tarafından bir oturum açıldığında, o oturum boyunca tüm sorgular aynı sunucuya gönderiliyor.
- `weight` parametresine göre yük ağırlıklarını belirler.
- LRU ( Least Recently Used ) ya da benzeri bir algoritma desteği yoktur.

**Aktif-Pasif Çalışma**:

- PgPool’un kendinden aktif-pasif çalışabilir.
- Watchdog alt süreci bunu yapıyor.
- PgPool’a gelen servis isteklerini izler.
- Diğer Watchdog’larla haberleşir.
- Aktif serviste problem olunca, bir diğeri aktif alıp **Virtual IP**'yi üzerine çeker.
- Geri dönen sunucu standby olarak dönüşür.

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

{% include callout.html content=" pgpool paketinin farklı PostgreSQL majör sürümleri için farklı sürümleri bulunur. Paket isimleri, servis adları ve dizinleri majör sürüme göre farklılık gösterir." type="primary" %}

{% include callout.html content=" Uygulamayı bağlantı havuzu ile yük dengeleme için yapıyoruz (en yaygın kullanılan özellikleri). Diğer işlevler ayar dosyaları kurcalanarak gerçekleştirilebilir." type="primary" %}

### Pgpool Yapılandırma

*pgpool.conf* içindeki bazı parametreler:

**`backend_hostname0`**: Master sunucunun IP adresi. Burada *hostname* kullanılabilir, ancak DNS çözümlemesi nedeniyle vakit kaybedilecektir.

**`backend_port0`**: Master sunucunun portu.

**`backend_weight0`**: Master sunucuya verilecek yükün oranı

{% include note.html content= " Bu parametrelerin sonu 1 olanları, 1. standby sunucusu, 2 ve devamıda varsa diğer standby sunucuları için kullanılacaktır." %}

**`load_balance_mode`**: Load balance modunu açar, `on` yapmak gereklidir.

**`master_slave_mode`**: Master/slave modunu etkinleştirir, loadbalancing için gereklidir, `on` yapılmalıdır.

**`master_slave_sub_mode`**: Replikasyonun hangi modla yapıldığını belirtir. Bunu `stream` olarak değiştirmek gereklidir.

**`sr_check_period`**: Saniye cinsinden verilen bu süre, master ve standby sunucu(lar) arasındaki replikasyon gecikme kontrolü aralığını belirtir. 10 sn genelde uygun bir süredir.

**`sr_check_user`**: SR replikasyon kontrolünü yapacak kullanıcı adını belirtir. Bu kullanıcı sadece gecikmeler için değil, aynı zamanda master/standby ayrımını yapabilmek için de kullanılır, o nedenle bu mod kullanılmayacak olsa bile geçerli bir kullanıcı   adı kullanılması gereklidir.

**`sr_check_password`**: Üstteki kullanıcının parolasını buraya yazıyoruz.

**`sr_check_database`**: Kontrol için hangi veritabanına bağlanacağımızı yazıyoruz. Öntanımlı olan `postgres` değeri uygundur.

**`delay_threshold`**: Eğer Master ve standby sunucu arasındaki gecikme burada belirtilen değerden daha fazla byte olursa, load balance modu devre dışı kalır ve SELECT sorguları artık standby sunucuya/sunuculara gönderilmezler. Bu, sorguların doğru çalışması için gerekli bir özelliktir. Gecikme tekrar belirtilen değer ve altına düşerse, load balancing tekrar devreye alınır.

Örnek Pgpool Ayarlarları

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

{% include note.html content=" pgpool bir PostgreSQL servisi ile aynı sunucuda çalıştırılıyorsa, PostgreSQL servisinin ya da pgpool’un farklı bir porta çekilmesi gerekir." %}

*pgpool.conf* dosyasına kümedeki sunucu bilgileri eklenir:

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
- *192.168.56.101*: Primary, *192.168.56.102*: Standby, *192.168.56.105*: PGPool

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

- PgPool çocuk süreçleri yönetmek için bir servisdir.
- Öntanımlı olarak 9898 portunu dinler.
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
