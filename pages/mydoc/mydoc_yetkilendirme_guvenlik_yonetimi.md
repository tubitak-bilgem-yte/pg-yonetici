---
title: "PostgreSQL Yetkilendirme ve Güvenlik Yönetimi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 12, 2020
summary: "PostgreSQL Yetkilendirme ve Güvenlik Yönetimi"
sidebar: mydoc_sidebar
permalink: mydoc_yetkilendirme_guvenlik_yonetimi.html
folder: mydoc
---

## PostgreSQL Yetkilendirme ve Güvenlik Yönetimi

### Host Bazlı Yetkilendirme

Bir istemci PostgreSQL’e bağlantı yapmak istediğinde`/var/lib/pgsql/13/data/pg_hba.conf` dosyasından geçmelidir.`pg_hba.conf` dosyasından kurallar sıralı olarak okunur ve olumlu / olumsuz ilk eşleşen satırda izin verme / kısıtlama gerçekleşir.

*pg_hba.conf* satırlarının yazım şekli:

```sql
# TYPE  DATABASE     USER      ADDRESS       METHOD     [OPTIONS]
```

- **TYPE** ⇒ local / host / hostssl / hostnossl

Yerel ya da uzaktan TCP/IP üzerinden yapılacak bağlantılar için `TYPE = "host"`

```sql
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             zabbix          127.0.0.1/32            trust
host    alfresco         alfrescous       192.168.3.0/24        md5
host    all                postgres        0.0.0.0/0            reject
```

- TCP/IP üzerinden yapılacak bağlantılardan sadece SSL olanları tanımlamak ⇒ "hostssl"
- Sadece SSL dışındakileri tanımlamak ⇒ "hostnossl"

```sql
# TYPE         DATABASE      USER          ADDRESS               METHOD
host             all         zabbix       127.0.0.1/32           trust
hostssl          all         zabbix       192.168.1.0/0          trust
hostnossl        all         zabbix       0.0.0.0/0              reject
```

Unix soketi üzerinden yapılacak bağlantılar için "local" tipinde satır tanımlanır. Bu satırda IP/ağ adresi bulunmaz ve diğer alanlar "host" tipindekilerle aynı şekilde ayarlanır.

```sql
#   TYPE    DATABASE        USER            ADDRESS                METHOD
    local     all           postgres                                peer
```

Yetkilendirmenin hangi yöntemle yapılacağı: **METHOD**

|-------|--------|
| `md5` | Parola doğrulaması (md5 hashli) yap |
| `password` | Parola doğrulaması (açık) yap |
| `ident` | İstemcinin sistem kullanıcısını ident sunucudan alarak veritabanı kullanıcısı olarak kullan (TCP/IP bağlantılarında) |
| `peer` | İstemcinin sistem kullanıcısını kernel’dan alarak veritabanı kullanıcısı olarak kullan (unix soketi bağlantılarında) |
| `trust` | Koşulsuz izin ver |
| `reject` | Koşulsuz reddet |

*pg_hba.conf*'ta bir satırda USER alanına şunlardan biri gelir:

|-------|--------|
| `all` | Tüm veritabanı kullanıcıları |
| `user` | "user" adlı kullanıcı/rol |
| `+group` | "group" adlı grup/rol ve bunun üyesi tüm kullanıcı/roller |
| `@file` | PostgreSQL dizininde "file" adlı dosyada yazan tüm kullanıcı/roller |

"peer" ve "ident" doğrulama metotlarında sistem kullanıcısını veritabanı kullanıcısına eşleyen bir "map" kullanmak da mümkündür. `pg_ident.conf`’ta yazan mappingler ile *pg_hba.conf*’ta yazan izinler eşleştirilir.

### Veritabanına Bağlantı Yetkilendirmesi

Host bazlı yetkilendirme satırlarınki `DATABASE` kısmında ne yazıldıysa sadece oralara izin verilir. Bu aşama geçildikten sonra veritabanına bağlanabilme iki koşula bağlıdır:

- datallowconn’un true olması.
- Kullanıcı/rolün veritabanı şemasını kullanma izninin olması.

``datallowconn false`` olarak ayarlanmış veritabanlarına kimse bağlanamaz (template).

```sql
postgres=> select datname,datallowconn from pg_database;
  datname  | datallowconn
-----------+--------------
 postgres  | t
 template1 | t
 template0 | f
 hede      | t
 pg02      | t
template1=# \c template0
FATAL:  database "template0" is not currently accepting connections
Previous connection kept
```

Üzerinde oynama yapılmasını istemediğimiz ve klonlamak için kullandığımız veritabanlarında `false` vermek işimizi kolaylaştırabilir.

```sql
postgres=# UPDATE pg_database SET datallowconn='false'
                  WHERE datname='ulkeler';
UPDATE 1

postgres=# \c ulkeler
FATAL:  database "ulkeler" is not currently accepting connections
Previous connection kept
```

Varsayılan olarak tablo, fonksiyon v.b. nesneler **PUBLIC** şeması ile oluşur, herkes tarafından kullanılabilir. Yani; herhangi bir kullanıcı tüm veritabanları ve tabloların listesini ve tüm tabloların yapılarını görebilir, verileri ise göremez. Birbirinden tamamen izole ortamlar isteniyorsa PUBLIC şemasının herkese açık hali kapatılır; ayrı kullanıcılar ayrı veritabanlarına belirli düzeylerde yetkilendirilir.

### SQL Erişim Yetkileri

GRANT komutu ile veritabanındaki nesnelere SQL erişim yetkileri ve rollere, grup rollerinin yetkileri verilir. Verilebilecek yetkiler:

|-------|--------|
| `SELECT` | Tablo, view ya da sequence için okuma, COPY TO 'yu da içerir |
| `INSERT` | Tabloda satır yazma, COPY FROM 'u da içerir |
| `UPDATE` | Tabloda satır değiştirme (SELECT de gerektirir) |
| `DELETE` | Tabloda satır silme (SELECT de gerektirir) |
| `TRUNCATE` | Tablo boşaltma |
| `REFERENCES` | Foreign Key oluşturma için bağlanan iki tabloda da bu yetki olmalı |
| `TRIGGER` | Trigger oluşturma |
| `CREATE` | Veritabanı için yeni şema oluşturma, şema için yeni nesne oluşturma, tablespace için tablo ve index oluşturma |
| `CONNECT` | Veritabanına bağlanma |
| `EXECUTE` | Fonksiyonu kullanabilme |
| `TEMPORARY` | Veritabanında geçici tablolar oluşturma |
| `USAGE` | Şema için nesnelere erişme, prosedürel dili fonksiyonda kullanma |
| `ALL PRIVILEGES` | Tam yetki |

Kullanıcılara bir tablonun tamamına değil kolon bazlı SELECT, INSERT, UPDATE ve REFERENCES gibi haklar da verilebilir.

örnek olarak, "hodo" tablosunda PUBLIC’e (dolayısıyla tüm rollere) SELECT yetkisi verelim:

```sql
hede=# GRANT SELECT ON hodo TO PUBLIC;
GRANT
hede=# \dp hodo
                        Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+----------
 public | hodo | table | postgres=arwdDxt/postgres+|                   |
        |      |       | =r/postgres               |                   |
(1 row)
```

Kendi kullanıcımıza *hodo* tablosunda tam yetki verelim:

```sql
hede=# GRANT ALL PRIVILEGES ON hodo TO bilgemyte;
GRANT
hede=# \dp hodo
                        Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+----------
 public | hodo | table | postgres=arwdDxt/postgres+|                   |
        |      |       | =r/postgres              +|                   |
        |      |       |bilgemyte=arwdDxt/postgres    |                   |
(1 row)
```

Kendi kullanıcımız ve "tubitak" kullanıcısı ile bu tabloya veri girmeyi deneyelim:

```sql
# psql -U tubitak -d hede -h 127.0.0.1
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
hede=> INSERT INTO hodo VALUES ('1','2','1');
ERROR:  permission denied for relation hodo
```

```sql
# psql -U bilgemyte -d hede -h 10.0.0.58 -W
Password for user bilgemyte:
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
hede=> INSERT INTO hodo VALUES ('5','5','1');
INSERT 0 1
```

Kendi kullanıcımıza *pg02* veritabanında “hodo” tablosunun sadece belirli kolonlarını okuma ve güncelleme yetkisi verelim:

```sql
# su - postgres
Last login: Thu Dec  7 17:35:09 +03 2017 on pts/0
-bash-4.2$ psql -d pg02
psql (11.5)
Type "help" for help.
pg02=# GRANT SELECT (x, y) ON hodo TO bilgemyte;
GRANT
pg02=# GRANT UPDATE (x) ON hodo TO bilgemyte;
GRANT
```

Tablolardaki yetkileri görelim:

```sql
pg02=> \dp
                             Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+----------
 public | hodo | table |                   | x:                 +|
        |      |       |                   |  bilgemyte=rw/postgres+|
        |      |       |                   | y:                 +|
        |      |       |                   |   bilgemyte=r/postgres  |
(1 row)
```

Kendi kullanıcımızla bağlanıp denemeler yapalım:

```sql
# psql -U bilgemyte -d pg02 -h 10.0.0.58
Password for user bilgemyte:
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
pg02=> SELECT * FROM hodo;
ERROR:  permission denied for relation hodo

pg02=> SELECT x,y FROM hodo;
 x | y
---+---
 1 | 2
 1 | 2
(2 rows)

pg02=> UPDATE hodo SET x = 3, y = 1 WHERE x = 1;
ERROR:  permission denied for relation hodo
pg02=>
pg02=> UPDATE hodo SET x = 3 WHERE x = 1;
UPDATE 2
```

### SQL Erişim Yetkileri: Satır Bazlı Güvenlik Politikası

Tablo içerisinde belirli satırlar için de SQL yetkileri tanımlanabilir. Bir tabloda satır bazlı politika aktif edildiyse her kullanıcı için bir politika ile izinler tanımlanmalıdır. Aksi takdirde varsayılan tablo sahibi hariç olarak herkes izinsizdir.

```sql
pg02=# ALTER TABLE phonebook ENABLE ROW LEVEL SECURITY;
ALTER TABLE
```

"tubitak" ya da kendi kullanıcımızla bağlanıp tablo satırlarını görmeye çalışalım:

```sql
# psql -U tubitak -d pg02 -h 127.0.0.1
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
pg02=> SELECT * FROM phonebook;
ERROR:  permission denied for relation phonebook
```

Tekrar postgres ile bağlanıp "tubitak" tüm satırları görebilsin ve değiştirebilsin politikalarını oluşturalım:

```sql
pg02=# CREATE POLICY admin_all ON phonebook TO tubitak USING (true) WITH CHECK (true);
CREATE POLICY
```

Diğer kullanıcılar tüm satırları görebilsin ama sadece kendiyle alakalı satırları değiştirebilsin:

```sql
pg02=# CREATE POLICY all_view ON phonebook FOR SELECT USING (true);
CREATE POLICY
pg02=# CREATE POLICY user_mod ON phonebook FOR UPDATE
  USING (current_user = user_name)
  WITH CHECK (current_user = user_name);
CREATE POLICY
```

Genel SQL yetkilendirmelerini de yapalım. "tubitak" her şeye yetkili, herkes ise sadece görme ve değiştirmeye yetkili olsun:

```sql
pg02=# GRANT SELECT, INSERT, UPDATE, DELETE ON phonebook TO tubitak;
GRANT
pg02=# GRANT SELECT ON phonebook TO PUBLIC;
GRANT
pg02=# GRANT UPDATE (phone,uid) ON phonebook TO PUBLIC;
GRANT
```

Tablo yetkilerine bakalım:

```sql
pg02=# \dp phonebook
                             Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+---------
 public | phonebook | table | postgres=arwdDxt/postgres+| | admin_all:   +
        |           |       | tubitak=arwd/postgres       +| |   (u): true  +
        |           |       | =rw/postgres              | |   (c): true  +
        |           |       |                           | |   to: tubitak   +
        |           |       |                           | | all_view (r):
        |           |       |                           | |   (u): true  +
        |           |       |                           | | user_mod (w):
                              (u):  (("current_user"())::text = user_name)
                              (c):  (("current_user"())::text = user_name)
(1 row)
```

### Şifreli Bağlantı Kullanımı

PostgreSQL kütüphaneleri SSL/TLS şifreli bağlantıyı destekler. Ayar dosyasından SSL de dinlemesi belirtilir. Desteklenen SSL metodları kısıtlanabilir. Şifreli bağlantı da açık bağlantı ile aynı porttan (5432) gerçekleşir.

Hazır SSL anahtar ve sertifikası yoksa openssl ile self-signed olarak yaratalım:

```sql
# openssl req -new -text -out pg02.req
Generating a 2048 bit RSA private key
........................................+++
...+++
writing new private key to 'privkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

Özel anahtarın parolasını kaldıralım:

```sql
# openssl rsa -in privkey.pem -out pg02.key
Enter pass phrase for privkey.pem:
writing RSA key

# rm -rf privkey.pem
```

Sertifika isteğini anahtarla imzalayıp sertifika dosyasını oluşturalım:

```sql
# openssl req -x509 -in pg02.req -text -key pg02.key -out pg02.crt

# rm -rf pg02.req
```

Anahtar ve sertifikayı PostgreSQL dizinine atıp sahipliklerini ayarlayalım:

```sql
# mv pg02.{key,crt} /var/lib/pgsql/11/data/
# chown postgres:postgres /var/lib/pgsql/11/data/pg02.{key,crt}
# chmod 400 /var/lib/pgsql/11/data/pg02.key
```

Ayar dosyasında SSL dinlemeyi açıp, sertifika dosyalarının isimlerini yazalım. Şifreleme metodlarını da kısıtlayabiliriz:

```sql
# vim /var/lib/pgsql/11/data/postgresql.conf
ssl = on
ssl_ciphers = 'HIGH:!SSLv2:!SSLv3:!aNULL'
ssl_cert_file = '/var/lib/pgsql/11/data/pg02.crt'
ssl_key_file = '/var/lib/pgsql/11/data/pg02.key'
```

pg_hba.conf'ta istediğimiz "host" ile başlayan satırları sadece SSL bağlantıya izin vermesi için "hostssl"e çevirelim:

```text
# vim /var/lib/pgsql/11/data/pg_hba.conf
hostssl    all             +dbadmin        127.0.0.1/32          trust
hostssl    all             +dbadmin        ::1/128               trust
```

"host" olarak kalan satırlardaki koşullarda ise SSL ya da plain her tip bağlantıya izin var.

PostgreSQL’i yeniden başlatıp bağlantı kuralım:

```sql
# systemctl restart postgresql-11

# psql -U yildirim -d template1 -h 127.0.0.1
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
```

{% include links.html %}
