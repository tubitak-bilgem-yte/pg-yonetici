---
title: "Tablespace"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 25, 2020
sidebar: mydoc_sidebar
permalink: mydoc_tablespace.html
folder: mydoc
---

## Tablespace Kavramı ve Kullanımı

PostgreSQL'de tablespace' ler, veritabanı yöneticilerinin veritabanı nesneleri için farklı saklama yerleri belirtmelerine olanak sağlar.

Tablespace kullanıldığında, veritabanı yöneticisi PostgreSQL kurulumunun disk üzerindeki yapısını kontrol edebilir. Bunun birçok yararı bulunmaktadır. Bunlardan ilki, eğer PostgreSQL kurulumunu gerçekleştirdiğiniz disk bölümünde yer kalmadıysa ve mantıksal ya da diğer yollarla bu alan büyütülemiyorsa, farklı ve uygun büyüklükte bir disk bölümü üzerinde bir tablespace yaratılır ve sistem yeniden yapılandırılana kadar bu tablespace üzerinde işlemler devam ettirilir. Diğer bir konu da şudur: Tablespace'ler veritabanı yöneticisine veritabanı nesnelerinin kullanım durumlarına göre veri yerleşimlerini düzenleme imkanı verir. Örneğin, çok sık kullanılan bir indeks çok hızlı ve sorunsuz çalışan bir diske yerleştirilebilir. Benzer şekilde, arşivlenmiş bilgi saklayan, çok seyrek kullanılan ve başarımın pek önemli olmadığı tablolar da daha yavaş bir diske yerleştirilebilir.

Veritabanları, şemalar, tablolar, indeksler ve sequence'ler tablespace'ler içine yerleştirilebilirler. Bir tablespace sadece superuser tarafından yaratılabilir. Diğer kullanıcılar tablespaceleri sadece kullanabilirler. Yapılması gereken şey, ilgili kullanıcılara o tablespace için `CREATE` izninin verilmesidir. CREATE izni olan kullanıcı ilgili komuta tablespace adını bir parametre olarak vermelidir.

Bir tablespace'in nasıl yaratılacağını aşağıdaki örnekle de açıklayabiliriz. Öncelikle, yeni tablespace'in yer alacağı dizini yaratalım. Bu dizinler *postgres* kullanıcısına ait olmalıdır:

```bash
mkdir /var/lib/pgsql/13/mgmtblspc
```

Şimdi veritabanına bağlanalım:

```sql
CREATE TABLESPACE yte LOCATION ' /pgsql2/mgm_tblspace' ;
\db

        List of tablespaces     
    Name    |  Owner   |        Location         
------------+----------+-------------------------  
    yte     | postgres | /var/lib/pgsql/13/mgmtblspc 
 pg_default | postgres |  
 pg_global  | postgres | 

(3 rows)
```

Artık tablomuzu yeni tablespace içinde yaratabiliriz:

```sql
CREATE TABLE t1 (c1 int) TABLESPACE yte; 

SELECT   schemaname,tablename,tablespace   FROM   pg_tablesWHERE tablename='t1' ;
 schemaname | tablename | tablespace
------------+-----------+------------
 public     | t1        | yte 
```

Benzer şekilde bir index de yaratalım:

```sql
CREATE INDEX c1_idx ON t1 USING btree (c1) TABLESPACE yte;
```

Kullanıcıların yarattığı tüm tablespace'lerin sembolik linkleri *$PGDATA/pg_tblspace* dizininde tutulur.

```sql
ls -l /var/lib/pgsql/13/data/pg_tblspc/
total 0 
lrwxrwxrwx  1  postgres  postgres  31  Jun  2  23:25  20955->/var/lib/pgsql/13/mgmtblspc
```

**Tablespace üzerinde işlem yapmak:**

- Bir tablespace' in adını ve sahibini değiştirebilirsiniz.

```sql
ALTER TABLESPACE tblspcad RENAME TO yeni_adı;
ALTER TABLESPACE tblspcad OWNER TO yeni_sahibi;
```

- Bir tablonun ya da bir index'in üzerinde olduğu tablespace'i değiştirmek için yine `ALTER` kullanılır.

```sql
ALTER TABLE tablo_adı SET TABLESPACE yeni_tablespace_adı;
ALTER INDEX index_adı SET TABLESPACE yeni_tablespace_adı;
```

### Geçici Nesneler için Tablespace Yaratma

PostgreSQL'de `temp_tablespaces` parametresi bulunmaktadır. Bu parametre, geçici tabloların, indexlerin ve de dosyaların yaratılabileceği tablespace için dizin belirtir. Geçici dosyalar büyük sıralama işlemleri için kullanılırlar. Bu parametrenin önemli bir özelliği, birden fazla tablespace belirtilebilmesidir. Her gereksinimde bu tablespacelerden rasgele birisi kullanılır. Bu, yük dengeleme için idealdir.

Geçici nesneleri Linux'da ramdiskte tutmak başarımı arttıracaktır. Linux'da öntanımlı olarak 16 tane ramdisk yaratılır:

```bash
$ ls -l /dev/ram* lrwxrwxrwx 1 root root   4 Jan   8 22:09 /dev/ram->ram1
brw-r----- 1 root disk 1,  0 Jan  8 22:09 /dev/ram0 
brw-r----- 1 root disk 1,  1 Jan  8 22:09 /dev/ram1 
brw-r----- 1 root disk 1, 10 Jan  8 22:09 /dev/ram10 
brw-r----- 1 root disk 1, 11 Jan  8 22:09 /dev/ram11 
brw-r----- 1 root disk 1, 12 Jan  8 22:09 /dev/ram12 
brw-r----- 1 root disk 1, 13 Jan  8 22:09 /dev/ram13 
brw-r----- 1 root disk 1, 14 Jan  8 22:09 /dev/ram14 
brw-r----- 1 root disk 1, 15 Jan  8 22:09 /dev/ram15 
brw-r----- 1 root disk 1,  2 Jan  8 22:09 /dev/ram2 
brw-r----- 1 root disk 1,  3 Jan  8 22:09 /dev/ram3 
brw-r----- 1 root disk 1,  4 Jan  8 22:09 /dev/ram4 
brw-r----- 1 root disk 1,  5 Jan  8 22:09 /dev/ram5 
brw-r----- 1 root disk 1,  6 Jan  8 22:09 /dev/ram6 
brw-r----- 1 root disk 1,  7 Jan  8 22:09 /dev/ram7 
brw-r----- 1 root disk 1,  8 Jan  8 22:09 /dev/ram8 
brw-r----- 1 root disk 1,  9 Jan  8 22:09 /dev/ram9 lrwxrwxrwx 1 root root   4 Jan  8 22:09 /dev/ramdisk->ram0
```

ram0 dışındaki herhangi birisini kullanabilirsiniz; ya da kendiniz yeni bir ramdisk yaratabilirsiniz:

```bash
mknod -m 0 /dev/pgramdisk b 1 1
```

Bu bölümü formatlayalım:

```sql
mkfs.exfs /dev/pgramdisk
```

Mount edeceğimiz dizini yaratalım:

```sql
mkdir /media/pgramdiskmount
```

...ve mount edelim:

```sql
mount /dev/pgramdisk /media/pgramdiskmount
```

{% include callout.html content=" Bu ramdisk'in açılışta sürekli olarak mount edilmesi için */etc/fstab* içine gerekli satırları eklemeyi unutmayın." type="primary" %}

PostgreSQL'de tablespace'lerin sadece sembolik link destekleyen işletim sistemlerinde kullanılabilir. NTFS sembolik linkleri Junction ile destekler. Ayrıntılı bilgiyi ve Junction eklentisini Microsoft'un web sitesinden edinebilirsiniz.

Initdb aşamasında yaratılan tablespaceler *$PGDATA* altında yaratılır.

PostgreSQL'deki *default_tablespace* parametresi, `CREATE TABLE` aşamasında kullanılacak öntanımlı değeri belirtir. Kurulum aşamasında bu değer atanmamıştır:

```sql
SHOW default_tablespace ;
 default_tablespace
--------------------
(1 row)
```

Bu değer `SET default_tablespace TO <tblspaceadı>` ile değiştirilebilir. Eğer yaratma işleminde `TABLESPACE` parametresi verilmezse, nesnenin yaratıldığı şablon (template) veritabanının bulunduğu tablespace değeri kullanılır.

Bir tablespace'i kaldırmak için `DROP TABLESPACE` komutu kullanılır.

PostgreSQL' de initdb aşamasında iki tablespace oluşturulur:

- **`pg_default`**: *template1* ve *template0* veritabanlarının öntanımlı tablespace'idir.
- **`pg_global`**: Sistem katalog tablolarının (paylaşılmış olanlar) tutulduğu tablespace'dir.

Tüm tablespace'ler  `pg_tablespace` sistem katalog tablosunda tutulur:

```sql
# select * from pg_tablespace ;
spcname   | spcowner | spcacl | spcoptions
-----------+----------+--------+------------
pg_default |       10 |        |
pg_global  |       10 |        |
mgm        |       10 |        |
```

Bu bilgi, psql'de `\db` ile de alınabilir.

{% include note.html content=" PostgreSQL'de tablespace'ler, genelde ayrı disklerde ya da en azından dizinlerde oluştururlurlar. Bu nedenle, tablespace'ler dolduğunda bu disk dolması demek olacaktır ve PostgreSQL PANIC verip kapanacaktır. Tablespace kullanıldığında disk dolulukları öncelikli olarak gözlemlenmelidir." %}

{% include links.html %}
