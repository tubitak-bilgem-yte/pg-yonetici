---
title: "FDW"
parent: Veritabanı Yönetimi
layout: default
nav_order: 8
--- 

## FDW ve SQL/MED kavramları

FDW, SQL/MED kavramı SQL:2003 standartı ile tanımlanmış ve PostgreSQL’e 9.1 ile birlikte gelmiştir. MED, “Modification of External Data” anlamına gelmektedir. SQL/MED, SQL diline Foreign Data Wrapper (FDW) ekleyerek diğer  veritabanı sunucularına doğal yollarla bağlantı yapılabilmesini sağlayan altyapıdır. En basit hali ile dblink’in yerine kullanılır, ve dblink’in sorunlarının hiçbirini barındırmaz. Burada “external” sözcüğü, o andaki veritabanının dışında olan başka bir veri kaynağını ifade eder. PostgreSQL’in gözünden bu yine bir PostgreSQL sunucusu olabilir, bir metin dosyası olabilir, başka bir storage yöntemi (columnar storage) olabilir, başka bir veritabanı sunucusu olabilir (Oracle, SQL Server, MySQL,Hadoop) ve hatta başka bir veri kaynağı da olabilir (Mailchimp, Twitter,WWW, ogr (GIS için)). Bu bölümde PostgreSQL’in kendi içinde gelen `postgres_fdw` ve `file_fdw`’yu inceleyeceğiz.

### postgres_fdw

postgres_fdw ile uzaktaki PostgreSQL sunucularından veri okuyabilir ve onlara veri yazabilirsiniz. “uzak” sunucu, aynı sunucudaki başka bir veritabanı, aynı sunucudaki başka bir instance içindeki başka bir veritabanı, ve başka bir sunucudaki başka bir veritabanı olabilir. Burada belirtmek gerekli ki, tüm veri uzaktaki sunucuda durmaya devamet mektedir. 9.6 sürümü itibariyle de veriler yine uzaktaki sunucuda işlenir ve sadece sonucu sorgunun çalıştırıldığı sunucuya aktarılır.

postgres_fdw, dblink ile benzer özellikler taşır, ancak hem daha güvenli, hem de daha hızlıdır.

postgres_fdw, PostgreSQL’in contrib paketleri içinde gelir. RHEL/CentOS üzerinde *postgresql96-contrib* paketini yüklediğinizde postgres_fdw eklentisi de gelmiş olur.

Şimdi adım adım kullanmayı anlatalım. Tüm komutları uygulamaların bağlanacağı sunucudaki veritabanında çalıştırmalısınız.

**1.** Öncelikle eklentiyi yaratalım. Birden fazla veritabanı varsa, bu komutu postgres_fdw eklentisini kullanacağınız her veritabanında ayrı ayrı çalıştırmalısınız:

```sql
CREATE EXTENSION postgres_fdw;
```

**2.** Şimdi, FOREIGN SERVER tanımını yaratacağız. Burada birçok yerden alışık olduğumuz bağlantı bilgilerini yazacağız:

```sql
CREATE SERVER ytefdw 
    FOREIGN DATA WRAPPER postgres_fdw 
    OPTIONS (host 'localhost', port '54961', dbname 'fdwtest');
```

{% include callout.html content=" Bu aşamada kullanıcı adı ve parola belirtmiyoruz. Eğer birden fazla veritabanına FDW ile bağlanacaksanız, bu komutu her biri için ayrı ayrı çalıştırmanız gerekiyor." type="primary" %}

**3.**  Şimdi, üstteki her FOREIGN SERVER için tüm her veritabanlarına bağlanacak tüm kullanıcıların eşleştirmelerini yapacağız. Burada uzaktaki kullanıcı adı ve parolasına ihtiyaç olacak. Öncelikle bir superuser bağlantısı örneği verelim:

```sql
CREATE USER MAPPING FOR postgres 
    SERVER ytefdw
    OPTIONS (user 'postgres', password 'deneme');
```

{% include callout.html content=" Bu komut ile yerel sunucudaki postgres kullanıcısını, uzaktaki postgres ile eşleştirdik. Her koşulda çalışan, ama tabi ki güvensiz bir yöntemdir." type="primary" %}

Şimdi farklı bir işlem yapalım:

```sql
CREATE USER MAPPING FOR yteuser 
    SERVER ytefdw 
    OPTIONS (user 'fdwuser', password 'deneme');
```

{% include callout.html content=" Burada ise üsttekinden farklı bir komut çalıştırdık. Bu komut, komutu çalıştırdığınız sunucudaki *yteuser* kullanıcısını, uzaktaki fdwuser kullanıcısına eşleştirdi. Peki bu ne demek? Bunu tabloları yarattıktan/aktardıktan sonra inceleyeceğiz." type="primary" %}

**4.** Şimdi uzaktaki tabloları nasıl aktabileceğimizi görelim. Bunun için 2 farklı yol var. İlki, uzaktaki şemayı olduğu gibi aktarmak. Bunu mevcuttaki bir şemaya ya da yeni bir şemaya aktarabilirsiniz. Biz yeni bir şemaya aktaralım:

```sql
CREATE SCHEMA yteschema;

IMPORT FOREIGN SCHEMA public 
    FROM SERVER ytefdw
    INTO yteschema;
```

{% include callout.html content=" Bu komutlarla birlikte, artık uzaktaki tüm tablolar yteschema içindedir!" type="primary" %}

```sql
[postgres] # INSERT INTO yteschema.fdwtable VALUES (1);
INSERT 0 1

[postgres] # SELECT * FROM yteschema.fdwtable;
c1 
---- 
1
(1 row)

[postgres] # UPDATE yteschema.fdwtable SET c1 = 2;
UPDATE 1

[postgres] # SELECT * FROM yteschema.fdwtable; 
c1 
----  
2
```

Peki, uzaktaki şemaya yeni bir tablo eklerseniz ne olur? O zaman bu tabloyu yerele de tanımlamamız gerekecek. Ya da uzaktaki şemadan tüm tabloları değil de sadece 1 ya da 1’den fazlasını almak isterseniz? Bu durumlarda CREATE FOREIGN TABLE komutunu kullanabilirsiniz. Bu komut, üstteki IMPORT FOREIGN SCHEMA komutundan farklıdır ve uzaktaki tablonun şemasını aynen girmenizi bekler:

```sql
CREATE FOREIGN TABLE yteschema.fdwtable3 (c1 int)
    SERVER ytefdw 
    OPTIONS (schema_name 'public');
```

{% include callout.html content=" Bu komut, ytefdw ile tanımladığımız foreign server’daki public şemasındaki *fdwtable3* tablosunu yereldeki yteschema şeması içinde foreign table olarak yaratır." type="primary" %}

Üstte de yazdığımız gibi, bu komutta verdiğiniz şema yapısı, remote’daki tablo ile birebir aynı olmalıdır. Örneğin tabloyu gerçekteki integer tipi ile değil de date tipi ile yaratırsak:

```sql
CREATE FOREIGN TABLE yteschema.fdwtable3 (c1 date)
    SERVER ytefdw 
    OPTIONS (schema_name 'public');
```

şu hatayı alırız:

```sql
[postgres] # SELECT * from fdwtable3 ;
ERROR:  22007: invalid input syntax for type date: "1"
CONTEXT:  column "c1" of foreign table "fdwtable3"
```

Tabloları yerelde uzaktakinden farklı adda da yaratmak mümkündür:

```sql
CREATE FOREIGN TABLE yteschema.fdwtable4 (c1 int) 
    SERVER ytefdw 
    OPTIONS (schema_name 'public', table_name 'fdwtable3');
```

Şimdi, 3. maddede referans verdiğimiz kullanıcı konusuna gelelim. Tekrar anımsarsak, superuser dışındaki kullanıcılar ile mapping yaratırsak ne olur?

```sql
CREATE USER MAPPING FOR yteuser 
    SERVER ytefdw
    OPTIONS (user 'fdwuser', password 'deneme');
```

{% include callout.html content=" superuser haricindeki tüm mappinlerde, uzaktaki sunucu mutlaka parola kontrol eden bir hba ayarına gereksinim duyar (trust hata verir):" type="primary" %}

```sql
[postgres] # SELECT * from yteschema.fdwtable;
ERROR:  2F003: password is required
DETAIL:  Non-superuser cannot connect if the server does not request a password.
HINT:  Target server's authentication method must be changed.
```

Yereldeki *yteuser* kullanıcısının, öncelikle şema ve sonrasında foreign table üzerinde hakkının olması gerekir, yoksa hata alırız:

```sql
[postgres] # SELECT * from yteschema.fdwtable3 ;
ERROR:  42501: permission denied for schema yteschema
LINE 1: SELECT * from yteschema.fdwtable3 ;

---<yetkili bir kullanıcı ile >
[postgres] # GRANT ALL ON SCHEMA yteschema TO yteuser;GRANT


---<şimdi tekrar yteuser kullanıcısı ile>
[postgres] # SELECT * from yteschema.fdwtable ;
ERROR:  42501: permission denied for relation fdwtable


---<yetkili bir kullanıcı ile >
[postgres] # GRANT ALL ON TABLE yteschema.fdwtable3 TO yteuser;


---<şimdi tekrar yteuser kullanıcısı ile> 
[postgres] # SELECT * from yteschema.fdwtable3 ;
ERROR:  42501: permission denied for relation fdwtable3
CONTEXT:  Remote SQL command: SELECT c1 FROM public.fdwtable3
```

Burada hata mesajı değişti ve uzaktaki sunucudan hata almaya başladık. Bunun nedeni de USER MAPPING tanımında göreceğiniz ve uzakta olan *fdwuser* kullanıcısının o tabloda yetkisinin olmaması. Uzak veritabanında şunu çalıştıralım:

```sql
[fdwtest] # GRANT ALL ON fdwtable3 TO fdwuser ;

--ve şimdi yerel sunucuya dönersek:

[postgres] # SELECT * from yteschema.fdwtable3 ; 

c1 
----  
1
(1 row)
```

Bu tabloya yazabiliriz de! Yerel sunucuda:

```sql
[postgres]# INSERT INTO yteschema.fdwtable3 VALUES (2);
INSERT 0 1

[postgres] # SELECT * from yteschema.fdwtable3 ;
 c1 
 ----  
 1  
 2
```

### file_fdw

file_fdw, PostgreSQL’in dosya sistemindeki dosyaları okumasını sağlayan bir FDW’dur (şu anda yazma desteği yoktur). Bu dosyaları okurken COPY’nin kullandığı dosya biçimlerini, yani csv ve text biçimlerini kullanmaktadır. Kullanımı `postgres_fdw`’ya göre biraz daha basittir:

```sql
CREATE EXTENSION file_fdw ;

CREATE SERVER ytefilefdw FOREIGN DATA WRAPPER file_fdw ;

CREATE FOREIGN TABLE filetest (c1 int, c2 timestamp) 
    SERVER ytefilefdw 
    OPTIONS ( filename '/var/lib/pgsql/9.6/ytedata.csv', format'csv' );
```

Bundan sonra,

```sql
SELECT * FROM filetest;
```

ile dosyayı okuyabilirsiniz.

{% include links.html %}
