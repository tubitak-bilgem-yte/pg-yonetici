---
title: "Sorguların İşlenmesi"
tags: [PostgreSQL]
keywords: postgres, binary
last_updated: December 31, 2020
summary: "Bu bölümde PostgreSQL’de query processing, tek tablolu bir sorgunun optimal planını elde etmek için izlenen adımlar, maliyet tahmini, plan ağacını oluşturma işlemleri, executor’un çalışması gibi konular daha çok sorgu optimizasyonu üzerine odaklanarak özetlenmiştir"
sidebar: mydoc_sidebar
permalink: mydoc_query_processing.html
folder: mydoc
---

## Query Processing (Sorgu İşleme) Genel Bakış

Query Processing, postgres’in en karmaşık alt sistemlerinden birisidir. Postgres, SQL standartlarının gerektirdiği çok sayıda özelliği destekler ve sorguları en verimli şekilde işler.

{% include image.html file="Query-Processing-1.png" alt="http://www.interdb.jp/pg/img/fig-3-01.png" caption="http://www.interdb.jp/pg/img/fig-3-01.png" %}

PostgreSQL’de, sürüm 9.6 ile gelen [paralel sorgu](https://www.percona.com/blog/2019/02/21/parallel-queries-in-postgresql/) işleminde birden çok background worker processes kullanılıyor olsa da temel olarak bağlı istemci tarafından verilen tüm sorgular bir backend process tarafından işlenir. Backend process; **`Parser`**, **`Analyzer/Analyser`**, **`Rewriter`**, **`Planner`** ve **`Executor`** olarak beş alt sistemden oluşur.

### Parser

PostgreSQL sunucusu, istemci uygulamadan bir sorgu aldıktan sonra, sorgu metnini Parser’a teslim eder. Parser gelen SQL ifadesini sonraki alt sistemlerce okunabilen bir Parse tree(Ayrıştırma ağacı)’ye dönüştürür. Bir örnek ile ele alırsak;

```sql
testdb=# SELECT id, data FROM tbl_a WHERE id < 300 ORDER BY data;
```

Parse tree; [parsenodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/parsenodes.h) dosyasından tanımlanmış, root düğümü `SelectStmt` olan bir veri yapısıdır. Şekil 1.2 **(b)**, Şekil 1.2 **(a)** örnek sorgunun parse tree’sini ifade eder.

{% include image.html file="Query-Processing-1.2.png" alt="http://www.interdb.jp/pg/img/fig-3-02.png" caption="Şekil 1.2. SELECT sorgusunun her bir öğesi parse tree’de bu öğelere karşılık gelecek şekilde sıralıdır. (1) tablonun ‘id’ sütunu ve targetList alanının ilk öğesidir. [http://www.interdb.jp/pg/img/fig-3-02.png]" %}

{% include callout.html content=" Bu adımda yalnızca girdinin sözdizimi kontrol edilir ve yanlışlık varsa hata mesajı dönderilir. Parser, gelen sorgusuyu anlamsal olarak ele almaz yani sorgu varolmayan bir tablo içeriyor olsa bile, parser bir hata döndürmez. **Anlamsal kontroller Analyzer/Analyser tarafından yapılır.**" type="primary" %}

### Analyzer

Analyzer, parser tarafından oluşturulan parse tree’nin anlamsal analizini gerçekleştirir ve [Query tree](https://www.postgresql.org/docs/current/querytree.html) oluşturur. Query tree root düğümü [parsenodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/parsenodes.h) dosyasından tanımlanmış **`Query`** structure’ı yapısındadır. Query tree gelen sorgunun metadatasını içerir.

{% include image.html file="Query-Processing-1.3.png" alt="http://www.interdb.jp/pg/img/fig-3-03.png" caption="Şekil. 1.3. *targetList, sorguda istenen sütunların listesidir. Örnekte, liste ‘id’ ve ‘data’ sütunları içerir. Gelen query tree '*' içeriyorsa analyzer/analyser bunu tüm sütunlarla değiştirir. *rtable, sorguda kullanılan ilişkilerin listesini tutar.*jointree FROM yan tümcesini ve WHERE yan tümcelerini saklar. *sortClause, SortGroupClause öğesinin bir listesidir. [http://www.interdb.jp/pg/img/fig-3-03.png]" %}

### Rewriter

Rewriter, [rule system](https://www.postgresql.org/docs/current/rules.html) içerisindeki kuralları dikkate alacak şekilde sorguları değiştirir ve sorguyu planlama ve yürütme için planner’a iletir. Kısaca, Query tree’yi [pg_rules](https://www.postgresql.org/docs/current/view-pg-rules.html) sistem kataloğunda saklanan kurallara göre dönüştüren sistemdir. [View](https://www.postgresql.org/docs/current/rules-views.html)’lar bir kural sistemi örneğidir. `CREATE VIEW` ile bir view tanımlandığında, karşılık gelen kural otomatik olarak oluşturulup katalogta saklanır.

```sql
sampledb=# CREATE VIEW employees_list AS SELECT e.id, e.name, d.name AS department 
    FROM employees AS e, departments AS d WHERE e.department_id = d.id;

sampledb=# SELECT * FROM employees_list;
```

{% include callout.html content=" Oluşturulan view’ı sorgu içinde gönderdiğimizde bu kısım *rtable düğümü tarafından tutulur ve rewriter, pg_rules sistem kataloğunda eşleşen view’dan bir alt sorgu oluşturup işler." type="primary" %}

{% include image.html file="Query-Processing-1.4.png" alt="http://www.interdb.jp/pg/img/fig-3-04.png" caption="Şekil 1.4 [http://www.interdb.jp/pg/img/fig-3-04.png]" %}

### Planner ve Executor

Planner, query tree’yi taramak ve sorguyu yürütmek için tüm olası planları bulmaktan sorumludur. Bu adımda rewriter’dan gelen query tree alınır ve executor’ın en verimli şekilde işleyebileceği bir plan tree oluşturulur. Üretilen plan sequential scan veya yararlı bir index tanımlıysa index scan içerebilir. Sorgu iki veya daha fazla tablo içeriyorsa, planner tablolara join’lemek için birkaç farklı yöntem önerebilir. Diğer RDBMS’lerde olduğu gibi [EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html) komutu ile planner’ın bir sorguyu yürütmeye nasıl karar verdiğini görebilirsiniz.

```sql
testdb=# EXPLAIN SELECT * FROM tbl_a WHERE id < 300 ORDER BY data;
                       QUERY PLAN
---------------------------------------------------------------
Sort  (cost=182.34..183.09 rows=300 width=8)
  Sort Key: data
  ->  Seq Scan on tbl_a  (cost=0.00..170.00 rows=300 width=8)
         Filter: (id < 300)
```

{% include image.html file="Query-Processing-1.5.png" alt="Şekil 1.5. Plan tree ile EXPLAIN komutunun sonucu arasındaki ilişki" caption="Şekil 1.5. Plan tree ile EXPLAIN komutunun sonucu arasındaki ilişki. [http://www.interdb.jp/pg/img/fig-3-05.png]" %}

{% include callout.html content=" Plan tree, plan düğümleri olarak adlandırılan elemanlardan oluşur ve PlannedStmt structure’ın *plantree listesine bağlanır. Bu öğeler [plannodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/plannodes.h) dosyasında tanımlanmıştır." type="primary" %}

Her plan düğümü, executor’ın işlemek için ihtiyaç duyduğu bilgilere sahiptir ve executor tek tablo sorgularında plan tree’nin sonundan root düğümüne doğru işler. Örneğin, Şekil 3.5'te verilen plan tree, sort düğümü ve sequential scan düğümünden oluşan bir listedir. Executor tabloyu önce sıralı olarak tarar ve elde edilen sonuçları sıralar.

{% include image.html file="Query-Processing-1.6.png" alt="Şekil. 1.6. Executor, arabellek yöneticisi ve geçici dosyalar arasındaki ilişki. [http://www.interdb.jp/pg/img/fig-3-06.png]" %}

Executor, Buffer Manager aracılığıyla tablo ve indexler üzerinde okuma yazma işlemlerini yapar. Bir sorgu işlenirken executor önceden ayrılmış `temp_buffers` ve `work_mem` gibi bellek alanlarını kullanır ve gerekli durumlarda geçici dosyalar oluşturabilir. Ayrıca, PostgreSQL tuple’lara erişirken, çalışan işlemlerin tutarlılığını ve izolasyonunu korumak için concurrency control mekanizmasını kullanır.

**Kaynak**:

- [The Internals of PostgreSQL](http://www.interdb.jp/pg/pgsql03.html)

- [Understanding How PostgreSQL Executes a Query](http://etutorials.org/SQL/Postgresql/Part+I+General+PostgreSQL+Use/Chapter+4.+Performance/Understanding+How+PostgreSQL+Executes+a+Query/)


{% include links.html %}
