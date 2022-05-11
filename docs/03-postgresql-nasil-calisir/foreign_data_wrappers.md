---
title: "Foreign Data Wrappers"
layout: default
parent: PostgreSQL Nasıl Çalışır?
nav_order: 4
---

## Foreign Data Wrappers (FDW)

Yabancı veri sarmalayıcıları (FDW), kullanıcının veritabanı dışındaki harici veri kaynaklarına yerel veritabanındaki bir tabloymuş gibi sorgular atmasını sağlar. Postgres ve SQL standartlarına uygun diğer ilişkisel veritabanları (MySQL, Oracle, mongoDB), key/value (NoSQL) kaynaklar, flat file veritabanları gibi harici veri kaynaklarına erişimi sağlayan pek çok FDW vardır. FDW'ler PostgreSQL üzerinde bir extension (eklenti) olarak uygulanmaktadır. Geliştirilmiş FDW'lere [Postgres wiki](https://wiki.postgresql.org/wiki/Foreign_data_wrappers) üzerinden ulaşabilirsiniz. PostgreSQL Global Development Group tarafından geliştirilen **`postgres_fdw`**[](https://www.postgresql.org/docs/current/postgres-fdw.html) dışında hemen hemen tüm uzantılar resmi olarak desteklenmemektedir.

{% include image.html file="fdw-1.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-1.png" caption="Şekil 1. [http://www.interdb.jp/pg/img/fig-4-fdw-1.png]" %}

Postgres 9.1 sürümünden itibaren bir veritabanı yönetim sisteminin, veritabanı dışında depolanan verileri nasıl entegre edebileceğini tanımlayan [SQL Management of External Data](https://wiki.postgresql.org/wiki/SQL/MED) (SQL/MED) stardartlarını kullanır. SQL/MED'de, uzak sunucudaki bir tabloya `foreign tablo` denir. postgres FDW'leri bu tabloları yönetmek için SQL/MED'i kullanır.

PostgreSQL üzerine FDW kurulumu ve daha fazlasına [FDW](mydoc_fdw.html) sayfasından ulaşabilirsiniz.

Sırasıyla PostgreSQL ve MySQL üzerinde *foreign_pg_tbl* ve *foreign_my_tbl* tablosuna sahip iki uzak sunucu olduğunu varsayalım. Gerekli eklentileri yükledikten ve uygun ayarları yaptıktan sonra, uzak sunuculardaki tablolara tıpkı yereldeki bir tabloymuş gibi sorgular atabilirsiniz. Örneğin, SELECT sorguları ile uzaktaki harici tablolara erişilebilir.

<script src="https://gist.github.com/berkanyiildirim/b83e21ed016d2984c184d96b0a24111f.js"></script>

Ayrıca, yerel tablolar ile farklı sunucularda depolanan harici tablolar arasında join işlemleri yapılabilir.

<script src="https://gist.github.com/berkanyiildirim/ae5ff94c31190b64e3c4284c623b7c5e.js"></script>

## FDW Nasıl Çalışır?

FDW özelliğini kullanmak için uygun eklentiyi kurmanız ve [CREATE FOREIGN TABLE](https://www.postgresql.org/docs/current/sql-createforeigntable.html), [CREATE SERVER](https://www.postgresql.org/docs/current/sql-createserver.html) ve [CREATE USER MAPPING](https://www.postgresql.org/docs/current/sql-createusermapping.html) gibi kurulum komutlarını çalıştırmanız gerekir. Gerekli ayarlamalar yapıldıktan sonra sorgular işlenirken foreign tabloya erişmek için uzantıda tanımlanan fonksiyonlar çağrılır.

{% include image.html file="fdw-2.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-2.png" caption="Şekil 2. [http://www.interdb.jp/pg/img/fig-4-fdw-2.png]" %}

- (1) Analyzer/Analyser gelen SQL'den query tree (sorgu ağacı) oluşturur.
- (2) Planner (ya da executor) uzak sunucuya bağlanır.
- (3) Eğer [use_remote_estimate](https://www.postgresql.org/docs/current/postgres-fdw.html#id-1.11.7.43.10.4) seçeneği açıksa (varsayılan kapalıdır) planner her bir plan yolu maliyeti tahmini için EXPLAIN komutu çalıştırır.
- (4) Planner, plan ağacından düz metin SQL ifadesi oluştur. Bu işleme **`deparsing`** denir.
- (5) Executor düz metin SQL ifadesini uzak sunucuya gönderir ve dönen sonucu alır.

Bir FDW'nin çalışma prensibi temel olarak bu şekildedir. Tüm bu adımlardan sonra executor duruma göre alınan verileri yerel sunucuda işler. Örneğin, çok tablolu bir sorgu yürütülürken executor alınan verileri diğer tablolar ile joinleme işlemleri gerçekleştirebilir.

### Query Tree Oluşturma

Analyzer/Analyser foreign tablo tanımlarını kullanarak gelen SQL'in sorgu ağacını oluşturur. Bu tanımlar [pg_catalog.pg_class](https://www.postgresql.org/docs/current/catalog-pg-class.html) ve [pg_catalog.pg_foreign_table](https://www.postgresql.org/docs/current/catalog-pg-foreign-table.html) kataloglarında tutulur.

### Uzak Sunucuya Bağlanma

Planner (executor) uzak veritabanı sunucusuna bağlanmak için belirli kütüphaneler kullanır. Örneğin, uzak PostgreSQL sunucusuna bağlanırken `postgres_fdw` [libpq](https://www.postgresql.org/docs/current/libpq.html) kütüphanesini, EnterpriseDB tarafından geliştirilen ve uzak MySQL sunucusuna bağlanırken kullanılan [mysql_fdw](https://www.postgresql.org/docs/current/libpq.html) ise `libmysqlclient` kütüphanesini kullanır.

Bağlantı için gerekli olan kullanıcı adı, sunucunun IP adresi, port numarası gibi parametreler [CREATE USER MAPPING](https://www.postgresql.org/docs/current/sql-createusermapping.html) ve [CREATE SERVER](https://www.postgresql.org/docs/current/sql-createserver.html) komutları kullanılarak [pg_catalog.pg_user_mapping](https://www.postgresql.org/docs/current/catalog-pg-user-mapping.html) ve [pg_catalog.pg_foreign_server](https://www.postgresql.org/docs/current/catalog-pg-foreign-server.html) kataloglarında saklanır.

### EXPLAIN Komutlarını Kullanarak Plan Ağacı Oluşturma (isteğe bağlı)

Postgres FDW'leri gelen sorgunun plan ağacını tahmini için foreign tabloların istatistik bilgilerinden faydalanabilir.

ALTER SERVER komutuyla `use_remote_estimate` seçeneği açık olarak ayarlanırsa, planner EXPLAIN komutu ile planların maliyetini uzak sunucuda sorgular. Bu seçeneğin kapalı olduğu durumda, maliyet hasabı yapılırken varsayılan sabit değerler kullanılır. İsteğe bağlı bir özelliktir, aktifleştirmek için:

```sql
localdb=# ALTER SERVER remote_server_name OPTIONS (use_remote_estimate 'on');
```

{% include note.html content=" EXPLAIN komutu her DMBS'de aynı değerleri döndermediğinden sonuçlar her DBMS fdw eklentisi tarafından planlama için kullanılamaz. Örneğin, mysql'nin EXPLAIN komutu yalnızca tahmini satır sayısını dönderir. PostgreSQL EXPLAIN komutu hem başlangıç ​​hem de toplam maliyeti dönderir, planner'ın ihtiyacı olan değerleri yalnızca postgres_fdw verebilir." %}

### Deparsing

Planner, plan ağacını oluşturmak için foreign tablonun plan ağacındaki tarama yollarından düz metin SQL ifadesi oluşturur. Örneğin, şekil 3 aşağıdaki SELECT ifadesinin plan ağacını göstermektedir.

```sql
localdb=# SELECT * FROM tbl_a AS a WHERE a.id < 10;
```

{% include image.html file="fdw-3.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-3.png" caption="Şekil 3. Foreign tabloyu tarayan plan ağacı örneği. [http://www.interdb.jp/pg/img/fig-4-fdw-3.png]" %}

{% include callout.html content=" Şekil 3'te görüldüğü gibi; PlannedStmt'ın plan ağacından bağlanan ForeignScan düğümü düz bir SELECT metni içerir. postgres_fdw, parser ve analyzer tarafından oluşturulan sorgu ağacından yediniden düz bir SELECT metni oluşturur. Postgres'te bu işleme **`deparsing`** denir." type="primary" %}

Aynı şekilde mysql_fdw kullamında MySQL için sorgu ağacından SELECT metni oluşturulur.

### SQL İfadeleri Gönderme ve Sonuçları Alma

Deparsing işleminden sonra, executor deparse edilmiş SQL ifadelerini uzak sunucuya gönderir ve sonuçları alır. SQL ifadelerinin uzak sunucuya gönderilme yöntemi her uzantıya göre değişir. Örneğin, mysql_fdw SQL ifadelerini transaction kullanmadan gönderir. mysql_fdw içinde bir SELECT sorgusu çalışırken gerçekleşen SQL komutlarının sırası: (Şekil 4)

{% include image.html file="fdw-4.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-4.png" caption=" Şekil 4. [http://www.interdb.jp/pg/img/fig-4-fdw-4.png]" %}

- (5–1) SQL_MODE'u 'ANSI_QUOTES' olarak ayarla.
- (5–2) Uzak sunucuya bir SELECT ifadesi gönder.
- (5–3) Uzak sunucudan sonuçları al. mysql_fdw, burada sonuçları PostgreSQL tarafından okunabilir verilere dönüştürür. Sonuçları PostgreSQL tarafından okunabilir verilere dönüştürme özelliği tüm FDW uzantıları için ortaktır.

postgres_fdw'de SQL komutlarının sırası biraz daha karmaşıktır. Postgres_fdw içinde bir SELECT sorgusu çalışırken gerçekleşen SQL komutları sırası: (Şekil 5)

{% include image.html file="fdw-5.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-5.png" caption="Şekil 5. [http://www.interdb.jp/pg/img/fig-4-fdw-5.png]" %}

- (5–1) Uzak transaction başlat. Varsayılan uzak transaction izolasyon düzeyi REPEATABLE READ.
- (5–2) - (5–4) İmleci bildir. SQL ifadesi temelde bir cursor (imleç) olarak çalıştırılır.
- (5–5) Sonucu almak için FETCH komutunu çalıştır. Varsayılan olarak 100 satır FETCH komutu tarafından getirilir.
- (5–6) Uzak sunucudan sonuçları al.
- (5–7) İmleci kapat.
- (5–8) transaction'ı commit et.

Uzak sunucuda loglar:

```sql
LOG:  statement: START TRANSACTION ISOLATION LEVEL REPEATABLE READ
LOG:  parse : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  bind : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  execute : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  statement: FETCH 100 FROM c1
LOG:  statement: CLOSE c1
LOG:  statement: COMMIT TRANSACTION
```

## Postgres_fdw

`postgres_fdw` eklentisi PostgreSQL Global Development Group tarafından resmi olarak geliştirilen ve kaynak kodu PostgreSQL kaynak kodu ağacında bulunan özel bir modüldür. Postgres 9.3 sürümü ile yayınlanan bu modül gelişimini yeni sürümlerle devam ettirmektedir.

Yukarıda postgres_fdw'nin tek tablolu sorguları nasıl işlediği açıklandı. Bu kısımda ise postgres_fdw'de çok tablolu sorguları, sort operasyonlarını ve aggregate fonksiyonlarını nasıl işlediği açıklanmaktadır. Örnekler SELECT ifadeleriyle verilmiş olsada postgres_fdw'le diğer DML (INSERT, UPDATE ,DELETE) ifadeleride işlenebilir.

### Çok Tablolu Sorgular

Çok tablolu bir sorgu çalıştığında postgres_fdw, her bir foreign tabloyu tek tablolu SELECT ifadesi gibi local sunucuya getirir ve join işlemlerini burada yapar. 9.6 sürümü ve sonrasında gelişen postgres_fdw remote join işlemleri yapabilmektedir. Bu özellik foreign tablolar aynı sunucuda ve [use_remote_estimate](https://www.postgresql.org/docs/current/postgres-fdw.html) seçeneği açık olduğunda kullanılabilir. Çalışma ayrıntıları aşağıda açıklanmıştır.

PostgreSQL'in iki foreign tabloyu (tbl_a ve tbl_b) joinleyen şu sorguyu nasıl işlediğini inceleyelim:

```sql
localdb=# SELECT * FROM tbl_a AS a, tbl_b AS b WHERE a.id = b.id AND a.id < 200;
```

`use_remote_estimate` seçeneği açıksa (varsayılan kapalı), postgres_fdw foreign tablolarla ilgili tüm planların maliyetlerini elde etmek için bir dizi EXPLAIN komutu gönderir. Planner tarafından en uygun plan seçilir.

```sql
(1) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((id < 200))
(2) EXPLAIN SELECT id, data FROM public.tbl_b
(3) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST
(4) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((((SELECT null::integer)::integer) = id)) AND ((id < 200))
(5) EXPLAIN SELECT id, data FROM public.tbl_b ORDER BY id ASC NULLS LAST
(6) EXPLAIN SELECT id, data FROM public.tbl_b WHERE ((((SELECT null::integer)::integer) = id))
(7) EXPLAIN SELECT r1.id, r1.data, r2.id, r2.data FROM (public.tbl_a r1 INNER JOIN public.tbl_b r2 ON (((r1.id = r2.id)) AND ((r1.id < 200))))
```

Planner tarafından seçilen planı görmek için localde EXPLAIN komutu çalıştırıldığında, uzak sunucuda işlenen en verimli sorgu olan inner join'i seçtildiği görülür.

<script src="https://gist.github.com/berkanyiildirim/012b459b087df23561bf816e3ace6545.js"></script>

postgres_fdw'nin bunu nasıl gerçekleştirdiğine bakalım:

{% include image.html file="fdw-7.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-7.png" caption="Şekil 6. [http://www.interdb.jp/pg/img/fig-4-fdw-7.png]" %}

- (3–1) Uzak transaction'ı başlat.
- (3–2) Her bir plan yolunun maliyetini tahmin etmek için EXPLAIN komutlarını göder. Bu örnekte, yedi EXPLAIN komutu gönderilmiş ve planner, yürütülen EXPLAIN komutlarının sonuçlarını kullanarak en verimli olanı seçmiştir.
- (5–1) SELECT ifadesini ifade eden c1 imlecini tanımla.

```sql
SELECT r1.id, r1.data, r2.id, r2.data FROM (public.tbl_a r1 INNER JOIN public.tbl_b r2 
  ON (((r1.id = r2.id)) AND ((r1.id < 200))))
```

- (5–2) Uzak sunucudan dönen sonuçları al.
- (5–3) c1 imlecini kapat.
- (5–4) transaction'ı commit et.

{% include note.html content=" `use_remote_estimate` seçeneği kapalıysa (varsayılan böyle), bir romote join sorgusunun nadiren seçildiğini unutmayın çünkü maliyet tahmini çok büyük dahili değerler kullanılarak yapılacaktır." %}

### Sort Operasyonları

Postgres sürüm 9.5 öncesinde `ORDER BY` gibi sıralama operasyonlarında uzak sunucudan tüm hedef satırlar alınıp local sunucuda sıralama işlemi yapılırdı. Bunu ORDER BY ifadesi içeren basit bir sorgu üzerinde EXPLAIN komutuyla gösterecek olursak:

<script src="https://gist.github.com/berkanyiildirim/2ade90ccc7ac375365745c9bec5c9f66.js"></script>

{% include callout.html content=" 6. satırda Executor `SELECT id, data FROM public.tbl_a WHERE ((id < 200))` sorgusunu uzak sunucuya gönderir ve dönen sonucu alır. 4. satırda ise yerel sunucuda executor getirilen *tbl_a* satırlarını sıralar." type="primary" %}

9.6 ve sonraki sürümlerle birlikte postgres_fdw uzak sunucudaki `SELECT` ifadelerini `ORDER BY` ile yürütebilir.

<script src="https://gist.github.com/berkanyiildirim/949e21fb90e7c9a45d4ad44d91bc5478.js"></script>

{% include callout.html content=" 4. satırda Executor uzak sunucuya `SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST` sorgusunu gönderir ve sıralanmış sonuçları alır. Bu gelişme ile local sunucuda iş yükü azaltılır." type="primary" %}

uzak sunucudaki loglar:

```sql
LOG:  statement: START TRANSACTION ISOLATION LEVEL REPEATABLE READ
LOG:  parse : DECLARE c1 CURSOR FOR
	   SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST
LOG:  bind : DECLARE c1 CURSOR FOR
	   SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST
LOG:  execute : DECLARE c1 CURSOR FOR
	   SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST
LOG:  statement: FETCH 100 FROM c1
LOG:  statement: FETCH 100 FROM c1
LOG:  statement: CLOSE c1
LOG:  statement: COMMIT TRANSACTION
```

### Aggregate Fonksiyonları

Sürüm 9.6 ve öncesinde, daha önce bahsedilen sıralama işlemine benzer şekilde `AVG()`, `COUNT()` gibi aggregate fonsiyonları da local sunucuda aşağıdaki adımlarda yapılırdı.

<script src="https://gist.github.com/berkanyiildirim/3d6d32cb20df4bbd935f37306d5be470.js"></script>

{% include callout.html content=" 5. satırda Executor `SELECT id, data FROM public.tbl_a WHERE ((id < 200))` sorgusunu gönderir ve dönen sonucu alır. 4. satırda Executor getirilen *tbl_a* satırlarının ortalamasını yerel sunucuda hesaplar. Çok sayıda satır göndermek yoğun ağ trafiği tükettiği ve uzun zaman aldığından bu işlem maliyetlidir." type="primary" %}

Sürüm 10 ve sonrasında postgres_fdw uzak sunucudaki SELECT deyimini ile aggregate fonksununu uzak sunucuda birlikte yürütür.

<script src="https://gist.github.com/berkanyiildirim/e5d1755b6352b164ed1e7abb2ff6c3d3.js"></script>

{% include callout.html content=" 4. satırda Executor uzak sunucuya AVG() fonksiyonunu içeren `SELECT avg(data) FROM public.tbl_a WHERE ((id < 200))` sorgusunu gönderir ve dönen sonucu alır. Bu işlem uzak sunucu ortalamayı hesapladığından ve sonuç olarak yalnızca bir satır gönderdiğinden daha verimlidir." type="primary" %}

Uzak sunucudaki loglar:

```sql
LOG:  statement: START TRANSACTION ISOLATION LEVEL REPEATABLE READ
LOG:  parse : DECLARE c1 CURSOR FOR
	   SELECT avg(data) FROM public.tbl_a WHERE ((id < 200))
LOG:  bind : DECLARE c1 CURSOR FOR
	   SELECT avg(data) FROM public.tbl_a WHERE ((id < 200))
LOG:  execute : DECLARE c1 CURSOR FOR
	   SELECT avg(data) FROM public.tbl_a WHERE ((id < 200))
LOG:  statement: FETCH 100 FROM c1
LOG:  statement: CLOSE c1
LOG:  statement: COMMIT TRANSACTION
```

**Kaynak**:

[1]. [The Internals of PostgreSQL](http://www.interdb.jp/pg/pgsql04.html)

{% include links.html %}
