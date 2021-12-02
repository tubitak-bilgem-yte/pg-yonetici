---
title: "EXPLAIN ve Log Analizi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 16, 2020
summary: "EXPLAIN ve Log Analizi"
sidebar: mydoc_sidebar
permalink: mydoc_explain.html
folder: mydoc
---

## EXPLAIN

`EXPLAIN` bir sorgu için oluşturulacak sorgu planını gösterir. Önceki istatistiklere dayanır.

`EXPLAIN ANALYZE` şeklinde kullanılır ve sorguyu gerçekten çalıştırır. 

|Option|Default|Value|Tanım|
|--- |--- |--- |--- |
|ANALYZE|TRUE|Boolean|Sorguyu çalıştırıp gerçekte çalışan değerleri verir.|
|VERBOSE|TRUE|Boolean|Her bir sorgu nodu, adımı seviyesinde çıktılar gösterir.|
|COSTS|TRUE|Boolean|Her bir akış için maliyetleri ayrı ayrı gösterir.|
|BUFFERS|FALSE|Boolean|Bellek kullanımıyla ilgili bilgileri gösterir.|
|TIMING|TRUE|Boolean|Gerçekleştirme zamanlarını gösterir.|
|FORMAT|TEXT|(TEXT,XML,JSON,YAML)| Çıktı türünü verir.|


### Kavramlar
#### Seq Scan
Planlayıcı diski blok blok okuyacak demektir. 

#### Cost
8K boyutundaki disk page'ini okumanın maliyeti 1 olarak kabul edilir. "sequential_page_cost" parametresiyle belirlenir.

**İlk kayda ulaşmanın maliyeti:** 0.00

**Tüm kayıtları getirmenin maliyeti:** 18584.82

Yukarıdaki **"sequential page cost"** birimi olarak hesaplanır.

#### Rows
Sorguda gelen kayıt sayısı

#### Witdh
Byte olarak tekil kayıtların ortalaması

#### actual time 
Sadece `ANALYZE` çıktısında görünür.

Bu sorgunun gerçekte ne kadar süre aldığını gösterir. Bu yüzden DML sorgularını transaction içinde yapıp sonradan rollback yapmak gerekir. *actual time* ile başlayan bölüm gerçekleşme bilgileridir. 

#### Buffers: shared read
Sadece `ANALYZE` çıktısında görünür.

Disten okuduğu blok sayısı

#### Buffers: shared hit
Sadece `ANALYZE` çıktısında görünür.

Bellekten okuduğu blok sayısı

#### Loops 
Sadece `ANALYZE` çıktısında görünür.

İşlemi kaç kere yaptığını gösterir. 



```
# ilk sorguda tamamı disk okumasından geliyor. 
# Buffers: read

postgres=# explain (analyze, buffers) select * from foo;
                                                  QUERY PLAN                                                   
---------------------------------------------------------------------------------------------------------------
 Seq Scan on foo  (cost=0.00..18334.00 rows=1000000 width=37) (actual time=0.874..96.455 rows=1000000 loops=1)
   Buffers: shared read=8334
 Planning time: 0.055 ms
 Execution time: 139.152 ms
(4 rows)

```


```
# aynı sorguyu tekrar çalıştırınca bir kısmı bellekten gelmekte. 
# Buffers: hit

postgres=# explain (analyze, buffers) select * from foo;
                                                  QUERY PLAN                                                   
---------------------------------------------------------------------------------------------------------------
 Seq Scan on foo  (cost=0.00..18334.00 rows=1000000 width=37) (actual time=0.099..86.507 rows=1000000 loops=1)
   Buffers: shared hit=224 read=8110
 Planning time: 0.055 ms
 Execution time

```

* Not: Eğer **"shared buffers"** değerini yeteri kadar büyütürsek hiç read olmadığını tablonun doğrudan bellekten çağrıldığını görebiliriz.


### Filter
*where* kelimesi ile bir filtre koyarsak artık *filter* diye yeni bir satırın geldiğini ve kaç kaydın filtrelendiğini görürüz.

```sql
EXPLAIN (analyze, buffers) select * from foo where c1 > 500;
                                                  QUERY PLAN                                                  
--------------------------------------------------------------------------------------------------------------
 Seq Scan on foo  (cost=0.00..20834.00 rows=999567 width=37) (actual time=0.150..113.683 rows=999500 loops=1)
   Filter: (c1 > 500)
   Rows Removed by Filter: 500
   Buffers: shared hit=8334
 Planning time: 0.076 ms
 Execution time: 149.473 ms
(6 rows)

```

* Test ortamımızda pg_class tablosu da bu durumu doğrular. 

```sql
SELECT relpages*current_setting('seq_page_cost')::float4
		+ reltuples*current_setting('cpu_tuple_cost')::float4
		+ reltuples*current_setting('cpu_operator_cost')::float4 AS total_cost
FROM pg_class
WHERE relname='foo';

 total_cost 
------------
      20834
(1 row)

```
### Yukarıdaki hesaplama ile explain değerlerini bulmak
Dikkat edersek, tahmin edilen maliyet, istatistikleri inceyerek gördüğümüzle aynıdır.

#### relpages
1. satırda `sequential scan` yaptığında gelen disk page sayısı. Bunu "seq_page_cost" ile çarpıyoruz. Çünkü her bir satırı getirmenin maliyeti.

#### reltuples
2. satırda dönen sonuçtaki kayıtların her birini işlemcinin işlemesi gerektiğinden *"cpu_tuple_cost"* ile çarpıyoruz. 
3. satırda yine filtre için row sayısını "cpu_operator_cost" ile çarparak toplam maliyeti hesaplıyoruz.

##### [cpu_operator_cost](https://postgresqlco.nf/doc/en/param/cpu_operator_cost/)
Planlayıcının her bir operatör veya işlev çağrısını işleme maliyetine ilişkin tahmini


### İndexler

#### Index Scan 
Eğer sorgu bir indexi kullanarak sonuçlar döndürüyorsa explain çıktısında *Index Scan* görürüz. 

```sql
EXPLAIN (analyze) SELECT * FROM foo
        WHERE c1 < 500 AND c2 LIKE 'abcd%';
                                                    QUERY PLAN                                                    
------------------------------------------------------------------------------------------------------------------
 Index Scan using foo_c1_idx on foo  (cost=0.42..24.51 rows=1 width=37) (actual time=0.362..0.362 rows=0 loops=1)
   Index Cond: (c1 < 500)
   Filter: (c2 ~~ 'abcd%'::text)
   Rows Removed by Filter: 499
 Planning time: 0.184 ms
 Execution time: 0.392 ms
(6 rows)
```

Eğer sadece c2 alanında arama yapsaydık, c2 alanını indexlemediğimiz için *Seq Scan* görecektik.

```sql
EXPLAIN (analyze) SELECT * FROM foo
        WHERE c2 LIKE 'bcd%';
                                               QUERY PLAN                                               
--------------------------------------------------------------------------------------------------------
 Seq Scan on foo  (cost=0.00..20834.00 rows=100 width=37) (actual time=2.265..113.159 rows=240 loops=1)
   Filter: (c2 ~~ 'bcd%'::text)
   Rows Removed by Filter: 999760
 Planning time: 0.100 ms
 Execution time: 113.194 ms
(5 rows)
```

#### Index Only Scan 
Eğer aradığımız veri sadece indexte bulunabiliyorsa

```sql
 EXPLAIN (analyze) SELECT c1 FROM foo
        WHERE c1 < 500 ;
                                                        QUERY PLAN                                                        
--------------------------------------------------------------------------------------------------------------------------
 Index Only Scan using foo_c1_idx on foo  (cost=0.42..23.37 rows=454 width=4) (actual time=0.066..0.295 rows=499 loops=1)
   Index Cond: (c1 < 500)
   Heap Fetches: 499
 Planning time: 0.134 ms
 Execution time: 0.373 ms
(5 rows)
 
```


#### Diğer Explain Örnekleri:

```sql
EXPLAIN SELECT * FROM tenk1;

                         QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Seq Scan on tenk1  (cost=0.00..458.00 rows=10000 width=244)
```

```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 7000;

                         QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Seq Scan on tenk1  (cost=0.00..483.00 rows=7001 width=244)
   Filter: (unique1 < 7000)
```

```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 100;

                                  QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Bitmap Heap Scan on tenk1  (cost=5.07..229.20 rows=101 width=244)
   Recheck Cond: (unique1 < 100)
   ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..5.04 rows=101 width=0)
         Index Cond: (unique1 < 100)
```


```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 = 42;

                                 QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Index Scan using tenk1_unique1 on tenk1  (cost=0.29..8.30 rows=1 width=244)
   Index Cond: (unique1 = 42)
```

```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 100 AND unique2 > 9000;

                                     QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Bitmap Heap Scan on tenk1  (cost=25.08..60.21 rows=10 width=244)
   Recheck Cond: ((unique1 < 100) AND (unique2 > 9000))
   ->  BitmapAnd  (cost=25.08..25.08 rows=10 width=0)
         ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..5.04 rows=101 width=0)
               Index Cond: (unique1 < 100)
         ->  Bitmap Index Scan on tenk1_unique2  (cost=0.00..19.78 rows=999 width=0)
               Index Cond: (unique2 > 9000)
```

```sql
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 100 AND unique2 > 9000 LIMIT 2;

                                     QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Limit  (cost=0.29..14.48 rows=2 width=244)
   ->  Index Scan using tenk1_unique2 on tenk1  (cost=0.29..71.27 rows=10 width=244)
         Index Cond: (unique2 > 9000)
         Filter: (unique1 < 100)
```

```sql
EXPLAIN SELECT *
FROM tenk1 t1, tenk2 t2
WHERE t1.unique1 < 10 AND t1.unique2 = t2.unique2;

                                      QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Nested Loop  (cost=4.65..118.62 rows=10 width=488)
   ->  Bitmap Heap Scan on tenk1 t1  (cost=4.36..39.47 rows=10 width=244)
         Recheck Cond: (unique1 < 10)
         ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..4.36 rows=10 width=0)
               Index Cond: (unique1 < 10)
   ->  Index Scan using tenk2_unique2 on tenk2 t2  (cost=0.29..7.91 rows=1 width=244)
         Index Cond: (unique2 = t1.unique2)
```

**Sorgu Planlayıcısına Farklı Plan Yaptırmak**:

Normal sorgu:

```sql
EXPLAIN SELECT *
FROM tenk1 t1, onek t2
WHERE t1.unique1 < 100 AND t1.unique2 = t2.unique2;

                                        QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Merge Join  (cost=198.11..268.19 rows=10 width=488)
   Merge Cond: (t1.unique2 = t2.unique2)
   ->  Index Scan using tenk1_unique2 on tenk1 t1  (cost=0.29..656.28 rows=101 width=244)
         Filter: (unique1 < 100)
   ->  Sort  (cost=197.83..200.33 rows=1000 width=244)
         Sort Key: t2.unique2
         ->  Seq Scan on onek t2  (cost=0.00..148.00 rows=1000 width=244)
```

Kullandığı bir yöntemi engelleyerek sonuç:

```sql
SET enable_sort = off;

EXPLAIN SELECT *
FROM tenk1 t1, onek t2
WHERE t1.unique1 < 100 AND t1.unique2 = t2.unique2;

                                        QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Merge Join  (cost=0.56..292.65 rows=10 width=488)
   Merge Cond: (t1.unique2 = t2.unique2)
   ->  Index Scan using tenk1_unique2 on tenk1 t1  (cost=0.29..656.28 rows=101 width=244)
         Filter: (unique1 < 100)
   ->  Index Scan using onek_unique2 on onek t2  (cost=0.28..224.79 rows=1000 width=244)
```

- Sonuç olarak bu işlem bize %12 daha pahalıya mal oldu.

### EXPLAIN ANALYZE

``ANALYZE`` dendiğinde ``EXPLAIN`` gerçekten sorguyu gerçekleştirir ve tahminleri ile beraber gösterir.

```sql
EXPLAIN ANALYZE SELECT *
FROM tenk1 t1, tenk2 t2
WHERE t1.unique1 < 10 AND t1.unique2 = t2.unique2;

                                                           QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Nested Loop  (cost=4.65..118.62 rows=10 width=488) (actual time=0.128..0.377 rows=10 loops=1)
   ->  Bitmap Heap Scan on tenk1 t1  (cost=4.36..39.47 rows=10 width=244) (actual time=0.057..0.121 rows=10 loops=1)
         Recheck Cond: (unique1 < 10)
         ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..4.36 rows=10 width=0) (actual time=0.024..0.024 rows=10 loops=1)
               Index Cond: (unique1 < 10)
   ->  Index Scan using tenk2_unique2 on tenk2 t2  (cost=0.29..7.91 rows=1 width=244) (actual time=0.021..0.022 rows=1 loops=10)
         Index Cond: (unique2 = t1.unique2)
 Planning time: 0.181 ms
 Execution time: 0.501 ms
```

### Log Analizi : PgBadger

PgBadger öngereksinimler ve kurulum:

```sql
# yum install perl-ExtUtils-MakeMaker make httpd
$ git clone https://github.com/dalibo/pgbadger.git
$ cd pgbadger
$ perl Makefile.PL
$ make && sudo make install
```

`postgresql.conf`’a aşağıdaki gibi log ayarlarını ekleyelim ve PostgreSQL’i yeniden başlatalım:

```sql
# vim /var/lib/pgsql/9.6/data/postgresql.conf
log_min_duration_statement = 0
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0

# systemctl restart postgresql-9.6
```

Derlenmiş binary’yi global bir yere taşıyıp Apache’yi başlatalım ve raporu günün loglarından yarattırıp web dizinine koyalım::

```sql
# cp /home/oyas/pgbadger/pgbadger /usr/bin/

# systemctl start httpd

# pgbadger -f stderr -s 10 -q -o /var/www/html/pgbadger/report_`date +\%Y-\%m-\%d`.html /var/lib/pgsql/9.6/data/pg_log/postgresql-Mon.log
```

### Log Analizi : pg_query_analyser

Öngereksinimler ve kurulum:

```sql
# yum install make gcc-c++ qt-devel
$ git clone https://github.com/WoLpH/pg_query_analyser.git
$ cd pg_query_analyser/
$ qmake-qt4
$ make && sudo make install
```

*postgresql.conf*’a pgbadger’daki gibi log ayarlarını ekleyelim, sadece prefix farklı:

```sql
# vim /var/lib/pgsql/9.6/data/postgresql.conf
log_line_prefix = '%t [%p]: [%l-1] host=%h,user=%u,db=%d,tx=%x,vtx=%v '

# systemctl restart postgresql-9.6
```

Yine PgBagder’da olduğu gibi Apache dizinine log analizi raporunu çıkartıyoruz:

```sql
# ./pg_query_analyser -i /var/lib/pgsql/9.6/data/pg_log/postgresql-Mon.log -o /var/www/html/report.html
```

{% include links.html %}
