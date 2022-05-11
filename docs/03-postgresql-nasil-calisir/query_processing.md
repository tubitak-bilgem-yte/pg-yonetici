---
title: "Sorguların İşlenmesi"
layout: default
parent: PostgreSQL Nasıl Çalışır?
nav_order: 2
---

## Query Processing (Sorgu İşleme)

Query Processing, PostgreSQL’in en karmaşık alt sistemlerinden birisidir. PostgreSQL, SQL standartlarının gerektirdiği çok sayıda özelliği destekler ve sorguları en verimli şekilde işler.

{% include image.html file="Query-Processing-1.png" alt="http://www.interdb.jp/pg/img/fig-3-01.png" caption=" Şekil 1. Query Processing. [http://www.interdb.jp/pg/img/fig-3-01.png]" %}

PostgreSQL’de, sürüm 9.6 ile gelen [paralel sorgu](https://www.percona.com/blog/2019/02/21/parallel-queries-in-postgresql/) işleminde birden çok background worker process kullanılıyor olsa da temel olarak bağlı istemci tarafından verilen tüm sorgular bir backend process tarafından işlenir. Backend process; **`Parser`**, **`Analyzer/Analyser`**, **`Rewriter`**, **`Planner`** ve **`Executor`** olarak beş alt sistemden oluşur.

| **`Parser`** | Düz metin SQL ifadesinden parse tree oluşturur. |
| **`Analyzer/Analyser`** | Parse tree'nin anlamsal analizini gerçekleştirir ve bir sorgu ağacı (query tree) oluşturur. |
| **`Rewriter`** | Varsa rule system'deki kuralları kullanarak sorgu ağacını dönüştürür. |
| **`Planner`** | Sorgu ağacından en etkin şekilde yürütülebilecek plan ağacını (plan tree) oluşturur. |
| **`Executor`** | Plan ağacı tarafından oluşturulan sırada, tablo ve indekslere erişerek sorguyu yürütür. |

### Parser

PostgreSQL sunucusu istemciden bir sorgu aldığında sorgu metnini Parser’a teslim eder. Parser gelen SQL ifadesini sonraki alt sistemlerce okunabilir Parse tree’ye (ayrıştırma ağacı) dönüştürür. Örnek ile ele alırsak;

```sql
testdb=# SELECT id, data FROM tbl_a WHERE id < 300 ORDER BY data;
```

Parse tree: [parsenodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/parsenodes.h) dosyasından tanımlanmış, root düğümü `SelectStmt` olan bir veri yapısıdır. Şekil 2 **(b)**, **(a)** örnek sorgunun parse tree’sini ifade eder.

{% include image.html file="Query-Processing-1.2.png" alt="http://www.interdb.jp/pg/img/fig-3-02.png" caption=" Şekil 2. SELECT sorgusunun her bir öğesi parse tree’de bu öğelere karşılık gelecek şekilde numaralıdır. (1) targetList alanının ilk öğesi ve tablodaki ‘id’ sütunudur. [http://www.interdb.jp/pg/img/fig-3-02.png]" %}

{% include callout.html content=" Bu adımda yalnızca girdinin sözdizimi kontrol edilir, yanlışlık varsa hata mesajı dönderilir. Parser, gelen sorgusuyu anlamsal olarak ele almaz yani sorgu varolmayan bir tablo içeriyor olsa bile, parser bir hata döndürmez. **Anlamsal kontroller Analyzer/Analyser tarafından yapılır.**" type="primary" %}

### Analyzer

Analyzer, parser tarafından oluşturulan parse tree’nin anlamsal analizini gerçekleştirir ve [Query tree](https://www.postgresql.org/docs/current/querytree.html) oluşturur. Query tree root düğümü [parsenodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/parsenodes.h) dosyasından tanımlanmış **`Query`** structure’ı yapısındadır. Query tree gelen sorgunun metadatasını içerir.

{% include image.html file="Query-Processing-1.3.png" alt="http://www.interdb.jp/pg/img/fig-3-03.png" caption="Şekil 3. Query tree örneği. [http://www.interdb.jp/pg/img/fig-3-03.png]" %}

- *targetList, sorguda istenen sütunların listesidir. Örnekte, liste ‘id’ ve ‘data’ sütunları içerir. Gelen query tree `*` (asterisk) içeriyorsa analyzer bunu tüm sütunlar için değiştirir.
- *rtable (range table), sorguda kullanılan ilişkilerin listesini tutar.
- *jointree FROM ve WHERE yan tümcelerini tutar.
- *sortClause, SortGroupClause öğesinin bir listesidir.

### Rewriter

Rewriter, Query tree’yi [pg_rules](https://www.postgresql.org/docs/current/view-pg-rules.html) sistem kataloğunda saklanan kurallara göre dönüştüren sistemdir. [View](https://www.postgresql.org/docs/current/rules-views.html)’lar bir kural sistemi örneğidir. `CREATE VIEW` ile bir view tanımlandığında, karşılık gelen kural otomatik olarak oluşturulup katalogta saklanır. Aşağıdaki view'ın önceden tanımlanmış olduğunu ve ilgili kuralın pg_rules sistem kataloğunda depolandığını varsayalım.

```sql
sampledb=# CREATE VIEW employees_list AS SELECT e.id, e.name, d.name AS department 
    FROM employees AS e, departments AS d WHERE e.department_id = d.id;

```

View'ı içeren bir sorgu verildiğinde, parser Şekil 4(a)'da gösterildiği gibi parse tree'yi oluşturur.

```sql
sampledb=# SELECT * FROM employees_list;
```

Rewriter bu aşamada range table düğümünü alt sorgunun parse tree'si için işler. bu, pg_rules'da karşılık gelen view'dır.

{% include image.html file="Query-Processing-1.4.png" alt="http://www.interdb.jp/pg/img/fig-3-04.png" caption="Şekil 4. Rewriter aşaması örneği. [http://www.interdb.jp/pg/img/fig-3-04.png]" %}

### Planner ve Executor

Planner'ın sorumluluğu, query tree’yi taramak ve sorguyu yürütmek için tüm olası planları bulmaktır. Bu adımda rewriter’dan query tree'yi alınır ve executor’ın en verimli şekilde işleyebildiği bir plan tree oluşturulur. Üretilen plan sequential scan veya yararlı bir indeks tanımlıysa index scan içerebilir. Sorgu iki veya daha fazla tablo içeriyorsa, planner tablolara joinlemek için birkaç farklı yöntem önerebilir. Diğer RDBMS’lerde olduğu gibi [EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html) komutu ile planner’ın bir sorguyu yürütmeye nasıl karar verdiği görülebilir.

```sql
testdb=# EXPLAIN SELECT * FROM tbl_a WHERE id < 300 ORDER BY data;
                       QUERY PLAN
---------------------------------------------------------------
Sort  (cost=182.34..183.09 rows=300 width=8)
  Sort Key: data
  ->  Seq Scan on tbl_a  (cost=0.00..170.00 rows=300 width=8)
         Filter: (id < 300)
```

{% include image.html file="Query-Processing-1.5.png" alt="Şekil 5. Plan tree ile EXPLAIN komutunun sonucu arasındaki ilişki" caption="Şekil 5. Örnek Plan tree ve bununla EXPLAIN komutu sonucu arasındaki ilişki. [http://www.interdb.jp/pg/img/fig-3-05.png]" %}

{% include callout.html content=" Plan tree, plan düğümleri (plan nodes) olarak adlandırılan elemanlardan oluşur, PlannedStmt structure’ın *plantree listesine bağlanır. Bu öğeler [plannodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/plannodes.h) dosyasında tanımlanmıştır." type="primary" %}

Her plan düğümü, executor’ın işlemek için ihtiyaç duyduğu bilgilere sahiptir. Executor tek tablo sorgularında plan tree’nin sonundan root düğümüne doğru işler. Örneğin, Şekil 5'te verilen plan tree, sort düğümü ve sequential scan düğümünden oluşan bir listedir. Executor tabloyu önce sıralı olarak tarar ve elde edilen sonuçları sıralar.

{% include image.html file="Query-Processing-1.6.png" alt=" Şekil 6. Executor, buffer manager ve geçici dosyalar arasındaki ilişki. [http://www.interdb.jp/pg/img/fig-3-06.png]" %}

Executor, Buffer Manager aracılığıyla küme içerisindeki tablo ve indeksler üzerinde okuma yazma işlemleri yapar. Sorgular işlenirken executor, önceden ayrılmış `temp_buffers` ve `work_mem` bellek alanlarını kullanır, gerekli durumlarda geçici dosyalar oluşturabilir. Ayrıca, PostgreSQL tuple’lara erişirken, çalışan transaction'ların tutarlılığı ve izolasyonu için concurrency control mekanizmasını kullanır.

**Kaynak**:

[1]. [The Internals of PostgreSQL](http://www.interdb.jp/pg/pgsql03.html)

[2]. [Understanding How PostgreSQL Executes a Query](http://etutorials.org/SQL/Postgresql/Part+I+General+PostgreSQL+Use/Chapter+4.+Performance/Understanding+How+PostgreSQL+Executes+a+Query/)

{% include links.html %}
