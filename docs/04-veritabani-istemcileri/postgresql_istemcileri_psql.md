---
title: psql
layout: default
parent: Veritabanı İstemcileri
nav_order: 1
---

psql PostgreSQL sunucu interaktif terminal istemcisidir. PostgreSQL sunucuda sorgu çalıştırma, sorgu sonuçlarını görüntüleme, kabuk parametreleri ile dosya veya komut gönderme, betik içerisinde kullanarak otomatik işlemler yaptırabilir.

## Genel Kullanımı

```sh
psql [seçenekler...] [veritabanı[kullanıcı]]
```

psql terminali ile sunucuya bağlanma:

```sh
su - postgres
$ psql
psql (11.5)
Type "help" for help.

postgres=#
```

{% include note.html content="Parametresiz çalıştırıldığında bağlantıyı yaptığımız sistem kullanıcısı ile yereldeki PostgreSQL soketine bağlanır!" %}

| psql için kullanılan parametreler |
|-----|------|
| **-h (--host=)** | Sunucu adı veya IP adresi |
| **-p (--port=)** | PostgreSQL port numarası |
| **-U (--username=)** | Kullanıcı adı |
| **-W (--password)** | Parola sor |
| **-w** | Parola sorma |
| **-d (--dbname=)** | Bağlanılacak veritabanı adı |

```sh
$ psql "service=myservice sslmode=require"
$ psql postgresql://dbmaster:5433/mydb?sslmode=require
```

Kullanıcı/parola ile TCP üzerinden veritabanına bağlanma:

```sh
$ psql -h 127.0.0.1 -U testuser -W test
Password for user testuser:
psql (11.5)
Type "help" for help.

test=>
```

| Sık kullanılan parametreler |
|-----|------|
| **-V (--version)** | PostgreSQL sunucu sürüm bilgisi görüntüle |
| **-? (--help)** | Yardım görüntüle |
| **-c (--command=)** | Belirtilen SQL komutlarını çalıştır |
| **-f (--file=)** | Dosyadan SQL komutları çalıştır |
| **-o (--output=)** | Komut çıktısını dosyaya yazdır |

Etkileşimli (interaktif) kabuk kullanma:

```sh
$ psql
psql (11.5)
Type "help" for help.

postgres=# \c ulkeler
You are now connected to database "ulkeler" as user "postgres".
ulkeler=# SELECT * FROM yerel_adlari;
```

Etkileşimsiz kabuk kullanma (dışardan komut yollama):

```sh
$ psql -c 'SELECT * FROM doviz;' ulkeler
```

Komut çıktısını kullanma (pipe):

```sql
$ echo '\c ulkeler \\ SELECT * FROM yerel_adlari;' | psql
```

Dosyayı girdi olarak kullanma:

```sh
$ psql ulkeler < sorgu.sql
```

Dosyayı çıktı olarak kullanma:

```sh
$ psql -c 'SELECT * FROM diller;' ulkeler > sonuc.sql
```

Koşullu belirteç kullanma (EOF):

```sql
$ psql <<EOF
> \c ulkeler
> SELECT * FROM doviz;
> EOF
```

Oracle’ın *sqlplus* komut satırı aracı öntanımlı *autocommit off* ile gelirken, psql öntanımlı **autocommit on** olarak çalışır. ``\set AUTOCOMMIT off`` komutu ile autocommit o oturum için kapatılabilir. ``\echo :AUTOCOMMIT`` ile autocommit durumu görülebilir. Kalıcı ekleme için ~/.psqlrc dosyasının içine \set AUTOCOMMIT off komutu eklenebilir.
Autocommit açıkken transaction yapmak için:

```sql
BEGIN;
  INSERT ...;
  UPDATE ...;
COMMIT;
```

| | psql istemci temel komutları  |
|-----|------|-----|------|
| **\l** | Veritabananlarını listeleme |**\q** | psql'den çıkış|
| **\c** | Belirtilen veritabanına bağlanma | **\help (\?)** | Yardım |
| **\dt** | Tabloları listeleme |**\copyright** | Lisans bilgileri |
| **\dT** | Veri tiplerini listeleme | **\conninfo** | Sunucu bağlantı bilgileri |
| **\du (\dg)** | Veritabanı rol/kullanıcı listeleme | **\password** | Rol parolası belirleme |
| **\dx** | Yüklü olan eklentileri listeleme | **\encoding** | Tanımlı olan karakter kodlaması |

Öntanımlı olarak sql sorgularının çıktıları sql biçeminde gelir psql üzerinden csv biçiminde çıktı almak için:

```sh
$ psql -d ulkeler -A -F"," -c "select * from doviz" > doviz.csv
```

```sql
hede=# \f ','
Field separator is ",".
hede=# \a
Output format is unaligned.
hede=# select * from personel;
```

psql üzerinden örnek veri yükleme:

```sh
su - postgres
$ psql <veritabanı_ismi>  < <sql dosyası>
```

{% include links.html %}
