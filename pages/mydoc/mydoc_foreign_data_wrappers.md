---
title: "Foreign Data Wrappers"
tags: [PostgreSQL]
keywords: postgres, FDW, Foreign Data Wrappers, deparsing, Postgres_fdw, Aggregate Functions, SQL/MED
last_updated: December 31, 2020
summary: "Bu bölümde PostgreSQL’de query processing, tek tablolu bir sorgunun optimal planını elde etmek için izlenen adımlar, maliyet tahmini, plan ağacını oluşturma işlemleri, executor’un çalışması gibi konular daha çok sorgu optimizasyonu üzerine odaklanarak özetlenmiştir"
sidebar: mydoc_sidebar
permalink: mydoc_foreign_data_wrappers.html
folder: mydoc
---

## Foreign Data Wrappers (FDW)

Yabancı veri paketleyicileri (FDW), kullanıcının veritabanı dışındaki  harici veri kaynaklarına yerel veritabanındaki bir tabloymuş gibi sorgular atmasını sağlar. Postgres ve SQL startlarına uygun diğer ilişkisel veritabanları (MySQL, Oracle, mongoDB), key/value (NoSQL) kaynaklar, flat file veritabanları gibi harici veri kaynaklarına erişimi sağlayan pek çok FDW vardır. FDW'ler PostgreSQL üzerinde bir extension(eklenti) olarak uygulanmaktadır. Geliştirilmiş FDW'lere Postgres wiki üzerinden ulaşabilirsiniz. PostgreSQL Global Development Group tarafından geliştirilen postgres_fdw dışında hemen hemen tüm uzantılar resmi olarak desteklenmemektedir.

{% include image.html file="fdw-1.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-1.png" caption="Şekil 1. [http://www.interdb.jp/pg/img/fig-4-fdw-1.png]" %}

Postgres 9.1 sürümünden itibaren bir veritabanı yönetim sisteminin, veritabanı dışında depolanan verileri nasıl entegre edebileceğini tanımlayan [SQL Management of External Data](https://wiki.postgresql.org/wiki/SQL/MED) (SQL/MED) stardartlarını kullanır. SQL/MED'de, uzak sunucudaki bir tabloya foreign tablo denir ve postgres FDW'leri bu tabloları yönetmek için SQL/MED'i kullanır.

Postgres üzerine FDW kurulumu ve daha fazlasına [FDW](mydoc_fdw.html) sayfasından ulaşabilirsiniz.

Sırasıyla Postgres ve MySQL  üzerinde *foreign_pg_tbl* ve *foreign_my_tbl* tablosuna sahip iki uzak sunucu olduğunu varsayalım. Gerekli eklentileri yükledikten ve uygun ayarları yaptıktan sonra, uzak sunuculardaki tablolara tıpkı yereldeki bir tabloymuş gibi sorgular atabilirsiniz. Örneğin, SELECT sorguları ile yerel sunucudan uzaktaki harici tablolara erişilebilir.

<script src="https://gist.github.com/berkanyiildirim/b83e21ed016d2984c184d96b0a24111f.js"></script>

Ayrıca, yerel tablolar ile farklı sunucularda depolanan harici tablolar arasında join işlemleri yapılabilir.

<script src="https://gist.github.com/berkanyiildirim/ae5ff94c31190b64e3c4284c623b7c5e.js"></script>

## FDW Nasıl Çalışır?

FDW özelliğini kullanmak için uygun eklentiyi kurmanız ve [CREATE FOREIGN TABLE](https://www.postgresql.org/docs/current/sql-createforeigntable.html), [CREATE SERVER](https://www.postgresql.org/docs/current/sql-createserver.html) ve [CREATE USER MAPPING](https://www.postgresql.org/docs/current/sql-createusermapping.html) gibi kurulum komutlarını çalıştırmanız gerekir. Gerekli ayarlamalar yapıldıktan sonra foreign tabloya erişmek için uzantıda tanımlanan fonksiyonlar çağrılır.

{% include image.html file="fdw-2.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-2.png" caption="Şekil 2. [http://www.interdb.jp/pg/img/fig-4-fdw-2.png]" %}

- (1) Analyzer/Analyser gelen SQL'den  query tree(sorgu ağacı) oluşturur.
- (2) Planner (executor) uzak sunucuya bağlanır.
- (3) Eğer [use_remote_estimate](https://www.postgresql.org/docs/current/postgres-fdw.html#id-1.11.7.43.10.4) seçeneği açıksa (varsayılan kapalıdır) planner her bir plan yolunun maliyetini tahmin etmek için EXPLAIN komutu çalıştırır.
- (4) Planner, plan ağacından düz metin SQL ifadesi oluştur. Bu işleme **`deparsing`** denir.
- (5) Executor düz metin SQL ifadesini uzak sunucuya gönderir ve dönen sonucu alır.

Bir FDW'nin çalışma prensibi temel olarak bu şekildedir.  Tüm bu adımlardan sonra  executor duruma göre alınan verileri yerel sunucuda işler. Örneğin, çok tablolu bir sorgu yürütülürken executor alınan verileri diğer tablolar ile joinleme işlemleri gerçekleştirebilir.

### Query Tree Oluşturma

Analyzer/Analyser [CREATE FOREIGN TABLE](https://www.postgresql.org/docs/current/sql-createforeigntable.html) veya IMPORT FOREIGN SCHEMA komutu ile  [pg_catalog.pg_class](https://www.postgresql.org/docs/current/catalog-pg-class.html) ve [pg_catalog.pg_foreign_table](https://www.postgresql.org/docs/current/catalog-pg-foreign-table.html) kataloglarında tutulan foreign tablo tanımlarını kullanarak gelen SQL'in sorgu ağacını oluşturur.

### Uzak Sunucuya Bağlanma

Planner (executor) uzak veritabanı sunucusuna bağlanmak için belirli kütüphaler kullanır. Örneğin, uzak PostgreSQL sunucusuna bağlanırken `postgres_fdw` [libpq](https://www.postgresql.org/docs/current/libpq.html) kütüphanesini, EnterpriseDB tarafından geliştirilen ve uzak MySQL sunucusuna bağlanırken kullanılan [mysql_fdw](https://www.postgresql.org/docs/current/libpq.html) ise `libmysqlclient` kütüphanesini kullanır. Bağlantı için gerekli olan kullanıcı adı, sunucunun IP adresi, port numarası gibi parametreler ise [CREATE USER MAPPING](https://www.postgresql.org/docs/current/sql-createusermapping.html) ve [CREATE SERVER](https://www.postgresql.org/docs/current/sql-createserver.html) komutları kullanılarak [pg_catalog.pg_user_mapping](https://www.postgresql.org/docs/current/catalog-pg-user-mapping.html) ve [pg_catalog.pg_foreign_server](https://www.postgresql.org/docs/current/catalog-pg-foreign-server.html) kataloglarında saklanır.

### EXPLAIN Komutlarını Kullanarak Plan Ağacı Oluşturma (isteğe bağlı)

Postgres FDW'leri gelen sorgunun plan ağacını tahmin etmek için  foreign tabloların istatistik bilgilerinden faydalanabilir. ALTER SERVER komutuyla `use_remote_estimate` seçeneği açık olarak ayarlanırsa, planner EXPLAIN komutu ile planların maliyetini uzak sunucuda sorgular. Bu seçeneğin kapalı olduğu durumda, maliyet hasabı yapılırken varsayılan sabit değerler kullanılır. İsteğe bağlı bir özelliktir, aktif etmek için:

```sql
localdb=# ALTER SERVER remote_server_name OPTIONS (use_remote_estimate 'on');
```

Ancak; EXPLAIN komutu her DMBS'de aynı değerleri döndermediği için sonuçları diğer DBMS fdw uzantıları tarafından planlama için kullanılamaz. Örneğin, mysql'nin EXPLAIN komutu yalnızca tahmini satır sayısını dönderir. Postgreste EXPLAIN komutu hem başlangıç ​​hem de toplam maliyeti dönderdiği için planner'ın ihtiyacı olan değerleri yalnızca postgres_fdw verebilir.

### Deparsing

Planner, plan ağacını oluşturmak için foreign tablonun plan ağacındaki tarama yollarından düz metin SQL ifadesi oluşturur. Örneğin, şekil 4.3 aşağıdaki SELECT ifadesinin plan ağacını göstermektedir.

```sql
localdb=# SELECT * FROM tbl_a AS a WHERE a.id < 10;
```

{% include image.html file="fdw-3.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-3.png" caption="Şekil 3. Foreign tabloyu tarayan plan ağacı örneği. [http://www.interdb.jp/pg/img/fig-4-fdw-3.png]" %}

Şekil 3'te görüldüğü gibi; PlannedStmt'ın plan ağacından bağlanan ForeignScan düğümü düz bir SELECT metni içerir. postgres_fdw, parser ve analyzer tarafından oluşturulan sorgu ağacından yediniden  düz bir SELECT metni oluşturur. Postgres'te bu işleme **`deparsing`** denir.

Aynı şekilde mysql_fdw'de sorgu ağacından MySQL için SELECT metni oluşturur ve [redis_fdw](https://github.com/pg-redis-fdw/redis_fdw) veya rw_redis_fdw kullanımı bir [SELECT komutu](https://redis.io/commands/select) oluşturur.

### SQL İfadeleri Gönderme ve Sonuç Alma

Deparsing işleminden sonra, executor sonuçları almak için deparse edilmiş SQL ifadelerini uzak sunucuya gönderir. SQL ifadelerinin uzak sunucuya gönderilmesi her uzantı için aynı şekilde gerçekleşmez. Örneğin, mysql_fdw SQL ifadelerini transaction kullanmadan gönderir. mysql_fdw içinde bir SELECT sorgusu çalışırken sırayla gerçekleşen SQL komutları: (Şekil 4)

{% include image.html file="fdw-4.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-4.png" caption="Şekil 4. [http://www.interdb.jp/pg/img/fig-4-fdw-4.png]" %}

- (5–1) SQL_MODE'u 'ANSI_QUOTES' olarak ayarla.
- (5–2) Uzak sunucuya bir SELECT ifadesi gönder.
- (5–3) Uzak sunucudan sonuçları al. Burada, mysql_fdw sonuçları PostgreSQL tarafından okunabilir verilere dönüştürür. Sonuçları PostgreSQL tarafından okunabilir verilere dönüştürme özelliği tüm FDW uzantıları için ortaktır.

postgres_fdw'de ise SQL komutlarının sırası biraz daha  karmaşıktır. Postgres_fdw içinde bir SELECT sorgusu çalışırken sırayla gerçekleşen SQL komutları: (Şekil 5)

{% include image.html file="fdw-5.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-5.png" caption="Şekil 5. [http://www.interdb.jp/pg/img/fig-4-fdw-5.png]" %}

- (5–1) Uzak transaction başlat. Varsayılan uzak transaction  izolasyon düzeyi REPEATABLE READ.
- (5–2) - (5–4) İmleci bildir. SQL ifadesi temelde bir cursor(imleç) olarak çalıştırılır.
- (5–5) Sonucu almak için FETCH komutunu çalıştır. Varsayılan olarak 100 satır FETCH komutu tarafından getirilir.
- (5–6) Uzak sunucudan sonuçları al.
- (5–7) İmleci kapat.
- (5–8) transaction'ı commit et.

Uzak sunucuda log dosyası:

```sql
LOG:  statement: START TRANSACTION ISOLATION LEVEL REPEATABLE READ
LOG:  parse : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  bind : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  execute : DECLARE c1 CURSOR FOR SELECT id, data FROM public.tbl_a WHERE ((id < 10))
LOG:  statement: FETCH 100 FROM c1
LOG:  statement: CLOSE c1
LOG:  statement: COMMIT TRANSACTION
```

### Postgres_fdw Extension

`postgres_fdw` eklentisi PostgreSQL Global Development Group tarafından resmi olarak geliştirilen ve kaynak kodu PostgreSQL kaynak kodu ağacında bulunan özel bir modüldür. Postgres 9.3 sürümü ile yayınlanan bu modül gelişimini yeni sürümlerle devam ettirmektedir.

Yukarıda postgres_fdw'nin tek tablolu sorguları nasıl çalıştığını açıklanmıştır. Bu bölümde ise postgres_fdw'de çok tablolu sorguların, sort operasyonlarının ve aggregate fonksiyonlarının nasıl çalıştığını açıklanmaktadır. Örnekler SELECT ifadeleriyle verilmiş olsada postgres_fdw'le diğer DML (INSERT, UPDATE ,DELETE) ifadeleride işlenebilir.

### Çok Tablolu Sorgular

Çok tablolu bir sorgu çalıştığında postgres_fdw, her bir foreign tabloyu tek tablolu SELECT ifadesi gibi yerel sunucuya getirir ve join işlemlerini burada yapar. 9.5 ve öncesi sürümlerde foreign tablolar aynı uzak sunucuda olsa bile bu işlem her biri için ayrı ayrı gerçekleşirdi. 9.6 sürümü ve sonrasında gelişen postgres_fdw remote join işlemleri yapabilmektedir. Bu özelliği kullanabilmek için foreign tablolar aynı sunucuda ve [use_remote_estimate](https://www.postgresql.org/docs/current/postgres-fdw.html) seçeneği açık olmalıdır çünkü maliyet hasabı yapılırken varsıyalan olarak kullanılan değerler genelde büyük maliyet dönderir ve sonuç olarak remote join işlemi postgres tarafından nadiren tercih edilir.

PostgreSQL'in version 9.6 ve sonrasıda iki foreign tablo (tbl_a ve tbl_b) üzerinde join işlemi yaparken nasıl çalıştığını açıklayacak olursak:

```sql
localdb=# SELECT * FROM tbl_a AS a, tbl_b AS b WHERE a.id = b.id AND a.id < 200;
```

Öncelikle postgres_fdw foreign tablolarla ilgili tüm planların maliyetlerini almak için uzak sunucuya bir dizi EXPLAIN komutu gönderir ve planner tarafından en uygun plan seçilir.

```sql
(1) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((id < 200))
(2) EXPLAIN SELECT id, data FROM public.tbl_b
(3) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST
(4) EXPLAIN SELECT id, data FROM public.tbl_a WHERE ((((SELECT null::integer)::integer) = id)) AND ((id < 200))
(5) EXPLAIN SELECT id, data FROM public.tbl_b ORDER BY id ASC NULLS LAST
(6) EXPLAIN SELECT id, data FROM public.tbl_b WHERE ((((SELECT null::integer)::integer) = id))
(7) EXPLAIN SELECT r1.id, r1.data, r2.id, r2.data FROM (public.tbl_a r1 INNER JOIN public.tbl_b r2 ON (((r1.id = r2.id)) AND ((r1.id < 200))))
```

Planner tarafından hangi planın seçildiğini görmek için EXPLAIN komutunu çalıştırırsak planlayıcının uzak sunucuda işlenen en verimli sorgu olan inner join'i seçtiğini görülür.

<script src="https://gist.github.com/berkanyiildirim/012b459b087df23561bf816e3ace6545.js"></script>

postgres_fdw'nin bunu nasıl yaptığına bakacak olursak:

{% include image.html file="fdw-7.png" alt="http://www.interdb.jp/pg/img/fig-4-fdw-7.png" caption="Şekil 6. [http://www.interdb.jp/pg/img/fig-4-fdw-7.png]" %}

- (3–1) Uzak transaction'ı başlat.
- (3–2) Her bir plan yolunun maliyetini tahmin etmek için EXPLAIN komutlarını göder. Bu örnekte, yedi EXPLAIN komutu gönderilmiş ve planner, yürütülen EXPLAIN komutlarının sonuçlarını kullanarak en verimli olanı seçmiştir.
- (5–1) şekildeki  SELECT ifadesini ifade eden c1 imlecini tanımla.
- (5–2) Dönen sonuçları uzak sunucudan al.
- (5–3) c1 imlecini kapat.
- (5–4) transaction'ı commit et.

### Sort Operasyonları

Postgres sürüm 9.5 öncesinde `ORDER BY` gibi sıralama işlemlerinde uzak sunucudan tüm hedef satırlar alınarak yerel sunucuda sıralama işlemi yapılırdı. Bunu ORDER BY ifadesi içeren basit bir sorgu üzerinde EXPLAIN komutuyla gösterecek olursak:

<script src="https://gist.github.com/berkanyiildirim/2ade90ccc7ac375365745c9bec5c9f66.js"></script>

{% include callout.html content=" 6. satırda Executor `SELECT id, data FROM public.tbl_a WHERE ((id < 200))` sorgusunu uzak sunucuya gönderir ve dönen sonucu alır. 4. satırda ise yerel sunucuda executor getirilen tbl_a satırlarını sıralar." type="primary" %}

9.6 ve sonraki sürümlerle birlikte postgres_fdw uzak sunucudaki `SELECT` ve `ORDER BY` ifadelerini birlikte yürütebilir.

<script src="https://gist.github.com/berkanyiildirim/949e21fb90e7c9a45d4ad44d91bc5478.js"></script>

{% include callout.html content=" 4. satırda Executor uzak sunucuya `SELECT id, data FROM public.tbl_a WHERE ((id < 200)) ORDER BY id ASC NULLS LAST` sorgusunu gönderir ve sıralanmış sonuçları alır. Böylece yerel sunucuda iş yükü azaltılır." type="primary" %}

### Aggregate Functions

Sürüm 9.6 ve öncesinde, sıralama işlemine benzer şekilde `AVG()` ve `COUNT()` gibi aggregate fonsiyonları yerel sunucuda yapılırdı.

<script src="https://gist.github.com/berkanyiildirim/3d6d32cb20df4bbd935f37306d5be470.js"></script>

{% include callout.html content=" 5. satırda Executor `SELECT id, data FROM public.tbl_a WHERE ((id < 200))` sorgusunu gönderir ve dönen sonucu alır. 4. satırda Executor getirilen tbl_a satırlarının ortalamasını yerel sunucuda hesaplar. Çok sayıda satır göndermek yoğun ağ trafiği tüketiği ve uzun zaman aldığı için bu işlem maliyetlidir." type="primary" %}

Sürüm 10 ve sonrasında postgres_fdw uzak sunucudaki SELECT deyimini ile aggregate fonksununu birlikte yürütür.

<script src="https://gist.github.com/berkanyiildirim/e5d1755b6352b164ed1e7abb2ff6c3d3.js"></script>

{% include callout.html content=" 4. satırda Executor uzak sunucuya bir AVG() fonksiyonu içeren `SELECT avg(data) FROM public.tbl_a WHERE ((id < 200))`  sorgusunu gönderir ve dönen sonucu alır. Uzak sunucu ortalamayı hesapladığı ve sonuç olarak yalnızca bir satır gönderdiği için bu işlem daha verimlidir." type="primary" %}

**Kaynak**:

[The Internals of PostgreSQL](http://www.interdb.jp/pg/pgsql04.html)

{% include links.html %}
