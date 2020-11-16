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

`EXPLAIN` bir sorgu için oluşturulacak sorgu planını gösterir.

Explain Örnekleri:

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
EXPLAIN SELECT * FROM tenk1 WHERE unique1 < 100 AND stringu1 = 'xxx';

                                  QUERY PLAN
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 Bitmap Heap Scan on tenk1  (cost=5.04..229.43 rows=1 width=244)
   Recheck Cond: (unique1 < 100)
   Filter: (stringu1 = 'xxx'::name)
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
