---
title: "PostgreSQL Kullanıcı Yönetimi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 12, 2020
summary: "Temel Veritabanı İşlemleri"
sidebar: mydoc_sidebar
permalink: mydoc_kullanici_yonetimi.html
folder: mydoc
---

## PostgreSQL Kullanıcı Yönetimi

PostgreSQL her veritabanı üzerinde bir sistem kataloğu tutar ve tablo, kullanıcı vb bilgileri burada saklar. Sisteme bağlanacak kullanıcı kısıtlamaları da bir ayar dosyasında tutulur.

Mevcut kullanıcıları listeleme:

```sql
postgres=# \du
                            List of roles
  Role name  |             Attributes              | Member of
-------------+-------------------------------------+-----------
 alfresco    |                                     | {}
 bilgemyte |                                     | {}
 postgres    | Superuser, Create role, Create DB,  | {}
                          Replication, Bypass RLS
 roundcube   |                                     | {}
 zabbix      |                                     | {}
```

**CREATE USER**: varsayılan olarak login yetkisi olan bir kullanıcı oluşturur.

**CREATE ROLE**: nologin bir kullanıcı oluşturur.

```sql
postgres=# CREATE ROLE yildirim;
CREATE ROLE
postgres=# CREATE USER bilgem;
CREATE ROLE
postgres=# \du
                            List of roles
  Role name  |             Attributes              | Member of
-------------+-------------------------------------+-----------
 yildirim        | Cannot login                        | {}
 bilgem       |                                     | {}
 postgres    | Superuser, Create role, Create DB,  | {}
                          Replication, Bypass RLS
```

Kullanıcı oluşturulurken attribute da belirtilir:

```sql
postgres=# CREATE ROLE deploy SUPERUSER LOGIN;
CREATE ROLE
```

Kullanılabilecek attribute’lar:

```sql
LOGIN
SUPERUSER
CREATEDB
CREATEROLE
REPLICATION LOGIN
PASSWORD
```

Ya da sonradan değiştirilir:

```sql
postgres=# ALTER ROLE deploy NOSUPERUSER CREATEDB;
ALTER ROLE
```

``SUPERUSER`` tam yetkili rol, örn: postgres kullanıcısının varsayılan olarak parolası yoktur. Tam yetkili olması tüm kısıtlamaları baypas etmesi demek değildir.

```sql
DROP ROLE bilgem;
```

Silinmek istenen rol kullanımda ise önce her bir veritabanında bu rolün sahiplendiği nesneler başka rollere devredilir ya da silinir, sonra rol silinir

```sql
REASSIGN OWNED BY bilgem TO yte;
DROP OWNED BY bilgem;
```

Grup rolleri de tekil roller gibi oluşturulur. Başka roller bu rollere eklenir ya da çıkarılır

```sql
CREATE ROLE admin NOLOGIN;
GRANT admin TO bilgem;
REVOKE admin FROM bilgem;
```

``INHERIT/NOINHERIT`` ile bir rolün üyesi olduğu grupların yetkilerini direk kullanıp kullanamayacağı belirtilir. INHERIT ile üye olunan grubun yetkileri direk kullanılabilir. NOINHERIT ile üye olunan grubun yetkileri sadece SET ROLE komutu ardından kullanılabilir.

```sql
CREATE ROLE developer LOGIN INHERIT;
CREATE ROLE admin NOINHERIT;
CREATE ROLE wheel NOINHERIT;
GRANT admin TO developer;
GRANT wheel TO admin;
```

Bir kullanıcıya başka bir kullanıcının hakları da ``GRANT`` ile aktarılır.

```sql
GRANT  yildirim TO dba;
```

Superuser hakları aktarılamaz sonra alter user yapılması gerekir.

```sql
ALTER USER dba WITH SUPERUSER;
```

Örnek olarak; *dbadmin* isimli bir grup rolü oluşturalım. Bu role ``CREATEDB`` hakkı verelim. Kendi adımızla ve *123abc* parolasıyla bir login rolü oluşturalım. *tubitak* isimli bir login rolü oluşturalım, buna *dbadmin* rolünü atayalım.

```sql

postgres=# CREATE ROLE dbadmin NOLOGIN;
CREATE ROLE

postgres=# ALTER ROLE dbadmin CREATEDB;
ALTER ROLE

postgres=# CREATE ROLE bilgem LOGIN PASSWORD '123abc';
CREATE ROLE

postgres=# CREATE ROLE tubitak LOGIN;
CREATE ROLE

postgres=# GRANT dbadmin TO tubitak;
GRANT ROLE

postgres=# \du
                           List of roles
 Role name |               Attributes              | Member of
-----------+---------------------------------------+-----------
 bilgem      |                                       | {}
 dbadmin   | Create DB, Cannot login               | {}
 tubitak      |                                       | {dbadmin}
 postgres  | Superuser, Create role, Create DB,    | {}
                          Replication, Bypass RLS
```

*dbadmin* grubu üyesi *tubitak* ile login olalım:

```sql
# psql -h 127.0.0.1 -U tubitak -d postgres
psql (11.5)
Type "help" for help.

postgres=>
```

*dbadmin* rolünün haklarını alalım ve veritabanı yaratalım:

```sql
postgres=> CREATE DATABASE pagila;
ERROR:  permission denied to create database
postgres=> SET ROLE dbadmin;
SET
postgres=> CREATE DATABASE pagila;
CREATE DATABASE
postgres=> RESET ROLE;
RESET
```

{% include links.html %}
