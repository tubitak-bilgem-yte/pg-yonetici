---
title: "PostgreSQL Temel Veritabanı İşlemleri"
tags: [PostgreSQL]
keywords: postgres, yönetim, tablo, index 
last_updated: October 30, 2020
summary: "Temel Veritabanı İşlemleri"
sidebar: mydoc_sidebar
permalink: mydoc_postgres_veritabanı_islemleri.html
folder: mydoc
---

## Temel Veritabanı İşlemleri

Mevcut veritabanlarını listeleme:

```sql
postgres=# \l
                                  List of databases
   Name    |  Owner   | Enc. |   Collate   |    Ctype    |   Access
                                                            privileges
-----------+----------+------+-------------+-------------+--------------+
 postgres  | postgres | UTF8 | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8 | en_US.UTF-8 | en_US.UTF-8 | =c/postgres +
                                                     postgres=CTc/postgres
 template1 | postgres | UTF8 | en_US.UTF-8 | en_US.UTF-8 | =c/postgres +
                                                     postgres=CTc/postgres
(3 rows)
```

Yeni bir veritabanı oluşturma:

```sql
postgres=# CREATE DATABASE pagila;
CREATE DATABASE
```

Sahip belirterek veritabanı oluşturma:

```sql
postgres=# CREATE DATABASE pg02 OWNER postgres;
CREATE DATABASE
```

```sql
CREATE DATABASE testdb
    WITH
    OWNER = postgres
    TEMPLATE = template0
    ENCODING = 'UTF8'
    LC_COLLATE = 'C'
    LC_CTYPE = 'C'
    CONNECTION LIMIT = 20;
```

Tek bir veritabanının sunucudaki bütün kaynakları kullanmasını istemiyorsak veritabanı bazlı bağlantı limiti koyulabilir.

```sql
postgres=# CREATE DATABASE test CONNECTION LIMIT 5
CREATE DATABASE
```

{% include note.html content="Superuser ve arkaplan işlemleri için limit işlemi geçerli değildir!" %}

Veritabanı silme:

```sql
postgres=#  DROP DATABASE testdb;
DROP DATABASE
```

Türkçe sıralama destekli veritabanı oluşturma:

```sql
postgres=# CREATE DATABASE guzel_turkcem ENCODING 'UTF8' LC_COLLATE
postgres-# 'tr_TR.UTF-8' LC_CTYPE 'tr_TR.UTF-8' TEMPLATE template0;
CREATE DATABASE

postgres=# \l
                                    List of databases
     Name      |  Owner   | Enc. |   Collate   |    Ctype    | Access
                                                               privileges
---------------+----------+------+-------------+-------------+--------
 guzel_turkcem | postgres | UTF8 | tr_TR.UTF-8 | tr_TR.UTF-8 |
```

## Tablo İşlemleri

Bir veritabanı içinde yeni bir tablo oluşturma:

```sql
postgres=# \c pagila
You are now connected to database "pagila" as user "postgres".

pagila=# CREATE TABLE personel (
  ad            varchar(40),
  soyad      varchar(40),
  kidem      int,
  uid           int  PRIMARY KEY
);
CREATE TABLE
```

Tabloları listeleme:

```sql
pagila=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+----------
 public | personel | table | postgres
(1 row)
```

{% include note.html content="PostgreSQL’de bir tablo sahibini tablo oluşmadan belirlemek mümkün değildir. Tablo, onu oluşturan kullanıcıya aittir." %}

Tablo sahipliğini değiştirmek için:

```sql
pagila=# CREATE USER yildirim;
CREATE ROLE
pagila=# ALTER TABLE personel OWNER TO yildirim;
ALTER TABLE
pagila=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+-------
 public | personel | table | yildirim

```

Tablo yapısını gösterme:

```sql
pagila=# \d personel
          Table "public.personel"
 Column |         Type          | Modifiers
--------+-----------------------+-----------
 ad     | character varying(40) |
 soyad  | character varying(40) |
 kidem  | integer               |
 uid    | integer               | not null
Indexes:
    "personel_pkey" PRIMARY KEY, btree (uid)
```

Tabloyu düzenleme: Yeni kolon ekleme

```sql
ALTER TABLE public.personel
    ADD COLUMN pagila text;
```

Tabloyu düzenleme: Bir kolon tipini değiştirme

```sql
ALTER TABLE public.personel
ALTER COLUMN ad TYPE character varying (50);
```

## Veri İşlemleri

Tabloya bir satır ekleme:

```sql
pagila=# INSERT INTO personel VALUES ('John','Doe',5,01);
INSERT 0 1
```

Tabloya birden fazla satır ekleme:

```sql
pagila=# INSERT INTO personel VALUES ('Jane','Doe',1,02),
           ('Richard','Roe',3,03), ('Fred','Bloggs', 7,04),
       ('Juan','Perez',11,05);
INSERT 0 4

```

Satır sorgulama:

```sql
pagila=# SELECT * FROM personel;
   ad    | soyad  | kidem | uid
---------+--------+-------+-----
 John    | Doe    |     5 |   1
 Jane    | Doe    |     1 |   2
 Richard | Roe    |     3 |   3
 Fred    | Bloggs |     7 |   4
 Juan    | Perez  |    11 |   5
(5 rows)
```

## İndeks İşlemleri

PostgreSQL Primary Key ya da Unique Constraint için indeksi otomatik olarak oluşturur.

```sql
postgres=# \c pagila
You are now connected to database "pagila" as user "postgres".
pagila=# \d personel
          Table "public.personel"
 Column |         Type          | Modifiers
--------+-----------------------+-----------
 ad     | character varying(40) |
 soyad  | character varying(40) |
 kidem  | integer               |
 uid    | integer               | not null
Indexes:
    "personel_pkey" PRIMARY KEY, btree (uid)
```

Standart indeks oluşturma:

```sql
pagila=# CREATE INDEX soyad_idx ON personel (soyad);
CREATE INDEX
```

{% include note.html content="PostgreSQL’in sorgu planlayıcısı hangi indexleri kullanıp sorguyu optimize edeceğini belirler." %}

## Referans Verme İşlemleri

Bir tablodan başka bir tabloya o tablonun Primary Key alanı aracılığıyla referans verilir.

```sql

pagila=# CREATE TABLE items
(
  code int PRIMARY KEY,
  name text,
  price numeric(10,2)
);
CREATE TABLE


pagila=# CREATE TABLE orders
(
  no int PRIMARY KEY,
  date date,
  amount numeric,
  item_code int REFERENCES items (code)
);
CREATE TABLE

```

Referans veren tablo:

```sql
pagila=# \d orders
      Table "public.orders"
  Column   |  Type   | Modifiers
-----------+---------+-----------
 no        | integer | not null
 date      | date    |
 amount    | numeric |
 item_code | integer |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (no)
Foreign-key constraints:
    "orders_item_code_fkey" FOREIGN KEY (item_code) REFERENCES items(code)
```

## Çalışma Zamanı Parametreleri

``SHOW`` ile belirli bir çalışma parametresinin bilgisi alınabilir:

```sql
postgres=# SHOW DateStyle;
 DateStyle
_____________
 ISO, MDY
(1 row)
```

Tüm parametrelerin listesine ve bilgisine erişmek için:

```sql
postgres=# SHOW ALL;
            name         | setting |                description
-------------------------+---------+-------------------------------------------------
 allow_system_table_mods | off     | Allows modifications of the structure
                     of ...
    .
    .
    .
 xmloption               | content | Sets whether XML data in implicit
                     parsing ...
 zero_damaged_pages      | off     | Continues processing past damaged
                     page headers.
(290 rows)
```

``SET`` komutu ile bir parametre çalışma zamanında değiştirilebilir:

```sql
postgres=# SET timezone='Europe/Rome';
SET
```

{% include note.html content="SET komutu ile değiştirilen parametre sadece o oturumda geçerlidir, oturum kapandığında geçerliliğini kaybeder. Parametrelerin kalıcı olması için **postgresql.conf** dosyası üzerinde ayarlama yapılmalıdır." %}

{% include links.html %}
