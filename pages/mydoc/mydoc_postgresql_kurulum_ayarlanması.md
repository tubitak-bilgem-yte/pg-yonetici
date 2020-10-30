---
title: PostgreSQL Veritabanı Kurulumu ve Ayarlanması
tags: [kurulum, ayarlama]
keywords: postgres, kurulum, ayarlama
last_updated: October 30, 2020
summary: "PostgreSQL Veritabanı Kurulumu ve Ayarlanması"
sidebar: mydoc_sidebar
permalink: mydoc_postgresql_kurulum_ayarlanması.html
folder: mydoc
---

* Kaynak koddan kurulum
* Paket yönetim sistemi ile kurulum

## Paket Yönetim Sistemi ile Kurulum

PostgreSQL farklı majör sürümlerini farklı depolarda tutar. Kurulacak sürüme göre o deponun paketi seçilip kurulur. Paket Deposunun Edinilmesi:

```sh
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

|| PostgreSQL Paketleri |
|-------|--------|
| postgresql11.x86_64 | PostgreSQL istemcisi ve kütüphaneleri |
| postgresql11-contrib.x86_64 | PostgreSQL ile beraber opsiyonel kurulabilecek eklentiler. |
| postgresql11-devel.x86_64 | PostgreSQL geliştirici kütüphaneleri |
| postgresql11-libs.x86_64| PostgreSQL istemcileri ve sunucu tarafından paylaşılan ortak kütüphaneler |
| postgresql11-server.x86_64 | PostgreSQL sunucusu |

### PostgreSQL Sunucunun Kurulumu:

```sh
yum install postgresql11-server postgresql11-contrib
```

Kurulumun tamamlamasından sonra PostgreSQL sunucu ve istemci programları ve eklentiler kurulur. Postgres adlı bir sistem kullanıcısı oluşturulur. Veri dizini boş olarak oluşturulmuş ve postgres kullanıcısına verilmiş durumdadır. Kurulumdan sonra standart ayarlarla PostgreSQL ayağa kaldırılabilir. Ancak önce DB’nin bir seferliğine ilklendirilmesi (initialize) edilmesi gerekir:

```sh
/usr/pgsql-11/bin/postgresql-11-setup initdb
```

İlklendirme işlemiyle catalog cluster (database cluster) oluşur. PostgreSQL’in veri dizininde dosyaların oluştuğunu görürüz:

```sh
ls -la /var/lib/pgsql/11/data
```

İlklendirme sırasında catalog cluster’ın varsayılan "locale" değerleri de belirlenir. Veri dizini olarak başka bir yer seçip ilklendirmeyi oraya yapmak da mümkündür. Veri dizinini “ / “ bölümünden farklı bir bölümde konumlandırmak iyidir.

PostgreSQL sunucuyu ayağa kaldırma:

```sh
systemctl start postgresql-11
```

Sistem açılışında otomatik başlamasını sağlama:

```sh
systemctl enable postgresql-11
```

Servis betikleri dışında yöntemlerle de PostgreSQL başlatılabilir ama en iyisi her zaman servis betiklerini kullanmaktır:

```sh
/usr/pgsql-11/bin/postgres -D /usr/local/pgsql/data >logfile 2>&1 &
/usr/pgsql-11/bin/pg_ctl start -l logfile
su postgres -c '/usr/pgsql-11/bin/pg_ctl start -D \
             /usr/local/pgsql/data -l serverlog'
```

Sunucunun ayağa kalktığı loglar izlenerek takip edilir. Sunucu ilklendirme loglarını şuraya yazar: ``/var/lib/pgsql/11/initdb.log``. Sonraki zamanlarda ise log dosyaları log dizininde gün ekiyle tutulacaktır:

```sh
ls -l /var/lib/pgsql/11/data/log
total 4
-rw....--- 1 postgres postgres 3738 May 26 14:05 postgresql-Fri.log
```

## PostgreSQL Sunucu Ayarları

PostgreSQL varsayılan ayarlarına dokunulmadan ayağa kaldırılabilir. Ayar değişiklikleri için oynanacak temel ayar dosyası: ``/var/lib/pgsql/11/data/postgresql.conf``

Ayarların çoğu **reload** ile aktifleşir, **restart** gerektirenler dosyada belirtilmiştir. PostgreSQL reload edildiğinde servis kesintisi yapılmadan ayar dosyasındaki değişiklikler tekrar okunur. Mevcut bağlantıların düşmesine neden olmayacağı için restart gerektiren özel parametrelerin değişimi hariç tüm durumlarda reload tercih edilmelidir.

```sh
systemctl reload postgresql-11
```

Ayar dosyalarında "#" ile başlayan yorum satırları her bir parametrenin ön tanımlı değerlerini gösterir:

```sh
#port = 5432                                            # (change requires restart)
#superuser_reserved_connections = 3                     # (change requires restart)
#unix_socket_directories = '/var/run/postgresql, /tmp'  #(comma-separated list of directories)
```

Sadece mevcut oturum için (geçici) parametre değiştirme:

```sh
postgres=# SET TIME ZONE 'Europe/Rome';
postgres=# select * from pg_settings where pg_settings.name='TimeZone';
   name   |   setting   | unit |                      category       |
----------+-------------+------+-------------------------------------+
TimeZone  | Europe/Rome |      | Client Connection Defaults / Locale |
                                                      and Formatting |
(1 row)
```

**ALTER SYSTEM** komutlarıyla da parametre değişikliği yapılabilir, ancak bu şekilde set edilen parametre anında etkin olmaz. Değişiklik ``postgresql.auto.conf`` dosyasına yazılır, servisi **reload** ettikten sonra etkinleşir. postgresql.auto.conf’taki parametre postgresql.conf’takine göre önceliklidir!

```sh
postgres=# alter system set work_mem='16MB';
ALTER SYSTEM
```

### PostgreSQL Ayarları: Dosya Yerleri

PostgreSQL veri dizini ile yetkilendirme ayar dosyalarının yerleri özel olarak belirtilebilir. Özel olarak belirlenmezse varsayılanda olarak PostgreSQL sürecini başlatırken verilen ``-D`` parametresinden veya **PGDATA** çevre değişkeninden alınır.

Değiştirmek istenirse:

```sh
data_directory = '/srv/postgresql'
hba_file = '/srv/postgresql/pg_hba.conf'
ident_file = '/srv/postgresql/pg_ident.conf'
```

PostgreSQL sunucu varsayılan olarak loopback (127.0.0.1) IP’sinden servis verir
Dışarıdan erişilebilmesi için:

```sh
listen_addresses = '*'
```

Hiç TCP/IP hizmeti vermemesi için:

```sh
listen_addresses = ''
```

Servisin unix soketiyle ilgili ayarlarıyla ilgili şunlar değiştirilebilir:

```sh
#unix_socket_directories = '/var/run/postgresql, /tmp'
#unix_socket_group = ''
#unix_socket_permissions = 0777
```

PostgreSQL sunucunun aynı anda kaç bağlantı isteği kabul edeceği:

```sh
max_connections = 100
```

{% include note.html content="Bu değer bir süre gözlemleyip, sunucu kaynaklarına göre düzenlenmelidir!." %}

### PostgreSQL Ayarları: Zaman

PostgreSQL’in sistemin zaman bilgilerini kullanması için ``--with-system-tzdata`` parametresiyle derlenmiş olması gerekir (rpm kurulumunda bu şekilde). Veritabanının kullandığı zaman ve yerellik bilgileri ilklendirme sırasında sunucudan alınır

```sh
postgres=# show timezone;
 TimeZone
----------
 Turkey

postgres=# select current_time;
    current_time
--------------------
 14:25:00.358229+03
```

PostgreSQL’in sistem zamanından farklı bir zaman kullanması istenirse ayarlardan değiştirilebilir.

```sh
datestyle = 'iso, mdy'
timezone = 'Turkey'
lc_messages = 'en_US.UTF-8'
lc_monetary = 'en_US.UTF-8'
lc_numeric = 'en_US.UTF-8'
lc_time = 'en_US.UTF-8'
```

{% include links.html %}
