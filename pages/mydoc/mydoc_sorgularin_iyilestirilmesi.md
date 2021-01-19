---
title: "Sorguların İyileştirilmesi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 13, 2020
summary: "Sorguların İyileştirilmesi"
sidebar: mydoc_sidebar
permalink: mydoc_sorgularin_iyilestirilmesi.html
folder: mydoc
---

## Sorguların İyileştirilmesi

### Yavaş Sorguların Belirlenmesi

- postgresql.conf dosyasında
  - logging_collector = on
  - log_directory = 'pg_log'
  - #log_statement = 'all'
  - log_min_duration_statement = 1000
  - log_line_prefix = '%m'

Yavaş sorgu logu örnekleri:

```sql
> LOG:  duration: 1117.957 ms  statement: commit;
> LOG:  duration: 1377.860 ms  statement: UPDATE sessions SET lastaccess=1515423465 WHERE userid='14' AND sessionid='667a73c4605b46bec3f3aca713b52134'
> LOG:  duration: 2037.666 ms  statement: insert into trends (itemid,clock,num,value_min,value_avg,value_max) values
...
```

### İndeksler

- Okuma performansını arttırır
- Yazma performansını düşürür
- Harcanan disk alanını arttırır
- Bir indeksin olması != kullanılması
- İndeks denemeleri gerçek ve tüm veri üzerinde yapılmalı
- Index-only tarama ve görünürlük haritası

#### İndeks Türleri

- Tek kolon
- Çok-kolon: Belirli bir sorgu yapısı için oluşturulur. Esnek değil. Sadece ilk kolonu tek kolon kullanılabiliyor.
- Unique: İndekslenen kolonda birden fazla aynı değer olmayacağı garantili. Daha hızlı çalışıyor.
- Kısmi: Tablonun belirli bir kısmının indekslenmesi. Hızlıdırlar, az yer kaplarlar, yazma performansını az düşürürler.
- Örtülü (Implicit): Birincil/yabancı anahtar ilişkileri için otomatik oluşturulurlar.
- B-Tree: Öntanımlı. Her derde deva. Bir değerin bir karşılığı olacak biçimde optimize.
- Hash: Eşitlik kıyaslamaları. B-Tree’den daha küçük. Çökerse elle oluşturulması gerekiyor.
- GIN: Tam metinde arama + diziler için.
- GiST: Geometrik veri tipleri + tam metinde arama için.

İndekslerden Ne Zaman Kaçınmalı?

- Az tetiklenen sorgular için özel indeks yapmayarak
- Küçük tablolarda
- Sık ve büyük güncelleme ya da veri girişi olan tablolarda
- Çoğu verinin NULL olduğu kolonlarda
- Kolon yapısı ile sık oynanan kolonlarda

İndeks Kullanım Bilgileri

- pg_index: indeks kullanan aramalar
- pg_stat_user_tables: indeks kullanmayan aramalar
- pg_*index*
- Önemli kolonlar:
  - idx_scan: Index Scan yapılma sayısı
  - idx_tup_read: İndeks ile okunan kayıt sayısı
  - idx_tup_fetch: İndeks ile alınan kayıt sayısı

İndeksler ve kullanım durumları nedir?

```sql
SELECT * FROM pg_stat_user_indexes;
```

İndeks kullanım durumu, index ve tablo boyutlarıyla beraber:

```sql
SELECT t.tablename, indexname, c.reltuples AS num_rows,
 pg_size_pretty(pg_relation_size(quote_ident(t.tablename)::text)) AS table_size,
 pg_size_pretty(pg_relation_size(quote_ident(indexrelname)::text)) AS index_size,
 CASE WHEN indisunique THEN 'Y'
 ELSE 'N'
 END AS UNIQUE,
 idx_scan AS number_of_scans, idx_tup_read AS tuples_read, idx_tup_fetch AS tuples_fetched
FROM pg_tables t
LEFT OUTER JOIN pg_class c ON t.tablename=c.relname
LEFT OUTER JOIN
 ( SELECT c.relname AS ctablename, ipg.relname AS indexname, x.indnatts AS number_of_columns, idx_scan, idx_tup_read, idx_tup_fetch, indexrelname, indisunique FROM pg_index x
  JOIN pg_class c ON c.oid = x.indrelid
  JOIN pg_class ipg ON ipg.oid = x.indexrelid
  JOIN pg_stat_all_indexes psai ON x.indexrelid = psai.indexrelid )
 AS foo
 ON t.tablename = foo.ctablename
WHERE t.schemaname='public' ORDER BY 1,2;
```

Seq tarama, indeks taramadan fazla mı?

```sql
SELECT schemaname, relname, seq_scan-idx_scan AS too_much_seq, case when seq_scan-idx_scan>0
  THEN 'Missing Index?' ELSE 'OK' END,
  pg_relation_size(format('%I.%I', schemaname, relname)::regclass) AS rel_size, seq_scan, idx_scan
 FROM pg_stat_user_tables
 WHERE pg_relation_size(format('%I.%I', schemaname, relname)::regclass)>80000
 ORDER BY too_much_seq DESC;
```

Çift (aynı) indeks var mı?

```sql
SELECT pg_size_pretty(SUM(pg_relation_size(idx))::BIGINT) AS SIZE,
       (array_agg(idx))[1] AS idx1, (array_agg(idx))[2] AS idx2,
       (array_agg(idx))[3] AS idx3, (array_agg(idx))[4] AS idx4
FROM (
    SELECT indexrelid::regclass AS idx, (indrelid::text ||E'\n'|| indclass::text ||E'\n'|| indkey::text ||E'\n'||
                                         COALESCE(indexprs::text,'')||E'\n' || COALESCE(indpred::text,'')) AS KEY
    FROM pg_index) sub
GROUP BY KEY HAVING COUNT(*)>1
ORDER BY SUM(pg_relation_size(idx)) DESC;
```

### Tablo İstatistikleri

- Sorgu planlamasında kullanılıyor.
- pg_statistic kataloğunda tutuluyor.
- İstatistikler canlı güncellenmiyor.
- ANALYZE komutu tarafından oluşturuluyor.
- autovacuum’un bir parçası olarak otomatik çalıştırılıyor.
- Güncel olsa bile tüm istatistikler yaklaşık veri içerir.
- Büyük veri değişikliklerinden sonra elle ANALYZE çalıştırmak yararlı.

### Sorgu Planlayıcısı

- Bir sorguyu gerçekleştirmenin birçok yöntemi var
- Sorgu planlayıcısı en verimli sorguya karar veriyor
- Planlama için tablo istatistiklerini kullanıyor
- Performansı doğrudan etkiliyor
- Hayaller Paris gerçekler Eminönü olursa, elle müdahale mümkün

{% include links.html %}
