---
title: "Sorgu Planlama"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 10, 2020
summary: "Sorgu Planlama"
sidebar: mydoc_sidebar
permalink: mydoc_sorgu_planlama.html
folder: mydoc
---

## Sorgu Planlama

### Planlayıcı Yöntemi Yapılandırması

Bu yapılandırma parametreleri, sorgu iyileştirici (query optimizer) tarafından seçilen sorgu planlarını etkilemek sağlanmıştır. Optimizer tarafından bir sorgu için seçilen varsayılan plan en uygun değilse, geçici bir çözüm olarak optimizer'ı farklı bir plan seçmeye zorlamak için bu konfigürasyon parametrelerini kullanmaktır. Optimizer tarafından seçilen planların kalitesini iyileştirmenin diğer yolları; [planlayıcı maliyet sabitlerinin](mydoc_sorgu_planlama.html) ayarlanması, ANALYZE'ın manuel olarak çalıştırılması, `default_statistics_target` konfigürasyon parametresinin değerinin artırılması ve belirli sütunlar için toplanan istatistik miktarının `ALTER TABLE SET STATISTICS`' komutu ile artırılmasıdır.

{% include callout.html content="**`enable_bitmapscan (boolean)`**: Sorgu planlayıcısının bitmap-scan plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın." type="primary" %}

{% include callout.html content="**`enable_gathermerge (boolean)`**: Sorgu planlayıcısının gather merge plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_hashagg (boolean)`**: Sorgu planlayıcısının hashed aggregation plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın." type="primary" %}

{% include callout.html content="**`enable_hashjoin (boolean)`**: Sorgu planlayıcısının hash-join plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın." type="primary" %}

{% include callout.html content="**`enable_incremental_sort (boolean)`**: Sorgu planlayıcısının artımlı sıralama (incremental sort) adımlarını kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_indexscan (boolean)`**: Sorgu planlayıcısının index-scan plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın." type="primary" %}

{% include callout.html content="**`enable_indexonlyscan (boolean)`**: Sorgu planlayıcısının  index-only-scan plan türlerini kullanmasını etkinleştirir / devre dışı bırakır (bkz. [](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)). Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_material (boolean)`**: Sorgu planlayıcısının materyalizasyonu kullanmasını etkinleştirir / devre dışı bırakır. Materyalizasyonu tamamen durdurmak imkansızdır, ancak bu değişkeni kapatmak planlayıcının uygunluk için gerekli olduğu durumlar dışında materyalize düğümler eklemesini engeller. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_mergejoin (boolean)`**: Sorgu planlayıcısının merge-join plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_nestloop (boolean)`**: Sorgu planlayıcısının nested-loop join planlarını kullanmasını etkinleştirir / devre dışı bırakır. nested-loop joins'i tamamen durdurmak mümkün değildir, ancak bu değişkeni kapatmak, mevcut başka yöntemler varsa planlayıcıyı bunları kullanmaya iter. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_parallel_append (boolean)`**: Sorgu planlayıcısının parallel append plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_parallel_hash (boolean)`**: Sorgu planlayıcısının paralel hash ile hash-join plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. hash-join planları da etkin değilse hiçbir etkisi yoktur. Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`enable_partition_pruning (boolean)`**: Sorgu planlayıcısının bölümlenmiş bir tablonun bölümlerini, sorgu planlarından kaldırma yeteneğini etkinleştirir / devre dışı bırakır. Bu aynı zamanda planlayıcının, sorgu çalıştırıcısının (executor) sorgu yürütme sırasında bölümleri kaldırmasına (yok saymasına) izin veren sorgu planları oluşturma yeteneğini de kontrol eder. Varsayılan açıktır. Ayrıntılar için [buraya](https://www.postgresql.org/docs/current/ddl-partitioning.html#DDL-PARTITION-PRUNING) bakabilirsiniz." type="primary" %}

{% include callout.html content="**`enable_partitionwise_join (boolean)`**: Sorgu planlayıcısının partition-wise join kullanmasını etkinleştirir / devre dışı bırakır; bu, bölümlenmiş tablolar arasında eşleşen bölümlerin birleştirilmesiyle gerçekleşecek join'lere izin verir. Partition-wise join, şu anda yalnızca birleştirme koşullarının aynı veri tipinde partition key içerdiğinde ve alt bölüm kümelerinin bire bir eşleştiğinde yapılabilmektedir. Partition-wise join planlaması, planlama sırasında önemli ölçüde CPU zamanı ve bellek kullandığı için varsayılan olarak kapalıdır." type="primary" %}

{% include callout.html content="**`enable_partitionwise_aggregate (boolean)`**: Sorgu planlayıcısının partition-wise grouping ve aggregation kullanımını etkinleştirir / devre dışı bırakır. GROUP BY yan tümcesi bölüm anahtarlarını içermiyorsa, her bölümde yalnızca partial aggregation gerçekleştirilebilir ve finalization daha sonra gerçekleştirilmelidir. Partition-wise grouping ve aggregation, planlama sırasında önemli ölçüde CPU zamanı ve bellek tüketeceğinden, varsayılan olarak kapalıdır." type="primary" %}

{% include callout.html content="**`enable_seqscan (boolean)`**: Sorgu planlayıcısının sequential scan (sıralı tarama) plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Sıralı taramaları tamamen kapatmak mümkün değildir, ancak bu değişkeni kapatmak planlayıcının başka yöntemler varsa bunları kullanmaya iter. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın." type="primary" %}

{% include callout.html content="**`enable_sort (boolean)`**: Sorgu planlayıcısının explicit sort (açık sıralama) adımlarını kullanmasını etkinleştirir / devre dışı bırakır. Explicit sort'u tamamen kapamak mümkün değildir, ancak bu değişkeni kapatmak planlayıcının başka yöntemler varsa bunları kullanmaya iter. Varsayılan açıktır. Etkileşimli oturum için yalnızca sorun giderme sorgularında kullanın" type="primary" %}

{% include callout.html content="**`enable_tidscan (boolean)`**: Sorgu planlayıcısının TID scan plan türlerini kullanmasını etkinleştirir / devre dışı bırakır. Varsayılan açıktır." type="primary" %}

### Planlayıcı Maliyet Sabitleri

Bu bölümde açıklanan cost (maliyet) değişkenleri keyfi bir skalada ölçülür ve yalnızca bağıl değerleri önemlidir. Bu nedenle hepsini aynı çarpanla yukarı veya aşağı ölçeklendirmek, planlayıcının seçimlerinde hiçbir değişikliğe neden olmaz. Bu maliyet değişkenleri, varsayılan olarak sıralı sayfa getirmelerinin maliyeti temellidir. `seq_page_cost` geleneksel olarak `1.0`'a ayarlanır ve diğer maliyet değişkenleri bu referansa göre ayarlanır. Ancak, dilerseniz belirli bir makine üzerinde milisaniye cinsinden gerçek yürütme süreleri gibi farklı bir skala kullanabilirsiniz.

{% include note.html content=" Ne yazık ki, maliyet değişkenleri için ideal değerler belinirken mükemmel tanımlanmış bir yöntem yoktur. En iyi yöntem, belirli bir kurulumun alacağı tüm sorguların karışımının ortalamalarıdır. Bu, değerlerin birkaç deney temelinde değiştirilmesi çok riskli olduğu anlamına gelir."%}

{% include callout.html content="**`seq_page_cost (floating point)`**: Planlayıcının, bir disk sayfasının sıralı getirilme (sequentially fetch) maliyetine ilişkin tahminini ayarlar. Varsayılan değer 1.0'dır. Bu değer, belirli bir tablespace'deki tablolar ve indeks için ilgili tablespace parametresi kullanılarak geçersiz kılınabilir (bkz. [ALTER TABLESPACE](https://www.postgresql.org/docs/current/sql-altertablespace.html))." type="primary" %}

{% include tip.html content=" **`seq_page_cost`**'u değiştirmenin ana nedeni, planlayıcı maliyetlerini milisaniye cinsinden daha fazla / az yürütme süresi belirtmeye çalışmaktır. Diğer tüm maliyetler bu maliyete göre otomatik olarak değişir."%}

{% include callout.html content="**`random_page_cost (floating point)`**: Planlayıcının, sıralı olarak getirilmeyen bir disk sayfasının maliyetine ilişkin tahmini ayarlar. Varsayılan 4.0'dır. Bu değer, belirli bir tablespace'deki tablolar ve indeks için ilgili tablespace parametresi kullanılarak geçersiz kılınabilir (bkz. [ALTER TABLESPACE](https://www.postgresql.org/docs/current/sql-altertablespace.html)).<br/><br/>

Bu değerin `seq_page_cost` değerine göre azaltılması, sistemin indeks taramalarını tercih etmesine; yükseltmek ise, indeks taramalarının daha maliyetli görünmesine sebep olacaktır. Aşağıdaki parametrelerde açıklanan CPU maliyetlerine göre disk I / O maliyetlerinin önemini değiştirmek için her iki değeri birlikte artırabilir veya azaltabilirsiniz.<br/><br/>

Mekanik disk depolamaya rastgele erişim, sıralı erişimden genellikle 4 kattan daha fazla maliyetlidir. Ancak, diske rastgele erişimlerin çoğunun önbellekte olduğu durumlarda ( indekslenmiş okumalar gibi ) daha düşük bir varsayılan kullanılır. Varsayılan değer, rastgele erişimi sıralı erişimden 40 kat daha yavaş modellemek olarak düşünülebilirken rastgele okumaların %90'ının önbelleğe alınmasını bekler.<br/><br/>

İş yükünüz için %90 önbellek oranının hatalı bir varsayım olduğuna inanıyorsanız, rastgele depolama okumalarının gerçek maliyetini daha iyi yansıtmak için `random_page_cost`'u artırabilirsiniz. Buna karşılık olarak, verilerinizin tamamen önbellekte olma olasılığı varsa, örneğin veritabanı toplam sunucu belleğinden küçükse, `random_page_cost`'u düşürmek uygun olabilir. Solid-state sürücüler gibi, sıralı okumaya göre düşük rastgele okuma maliyetine sahip depolamalar daha düşük değerlerle daha verimli modellenebilir, (ör. 1.1)" type="primary" %}

{% include tip.html content=" Sistem **random_page_cost**'u **seq_page_cost**'tan daha düşük bir değere ayarlamanıza izin verse de, bunu yapmak fiziksel olarak mantıklı değildir. Fakat, bu parametreleri eşit olarak ayarlamak, veritabanı tamamen RAM'de önbelleğe alınmışsa anlamlıdır. Çünkü bu durumda, sıralı olmayan sayfalara dokunmanın cezası yoktur. Ayrıca, yoğun şekilde önbelleğe alınmış bir veritabanında, zaten RAM'de bulunan bir sayfayı getirmenin maliyeti normalden çok daha düşük olduğu için CPU parametrelerine göre her iki değeri de düşürmelisiniz."%}

{% include tip.html content=" Bu parametre veritabanı depolamanız için seek/scan süresinin oranını ayarlağı için bunların farklı olduğu özel depolama (SSD'ler, ileri teknoloji SAN'lar vb.) kullanmıyorsanız değiştirilmemelidir. Veritabanının indeskleri daha fazla tercih etmesine ihtiyacınız varsa, bunun yerine `effective_cache_size` ve `cpu_ *cost` ayarlarına bakın."%}

{% include callout.html content="**`cpu_tuple_cost (floating point)`**: Sorgu sırasında planlayıcının her satırı işleme maliyetine ilişkin tahminini ayarlar. Öntanımlı değeri 0.01'dir." type="primary" %}

{% include callout.html content="**`cpu_index_tuple_cost (floating point)`**: Planlayıcının, bir indesk taraması sırasında her bir indeks girişini işleme maliyetine ilişkin tahminini ayarlar. Öntanımlı değeri 0,005'tir." type="primary" %}

{% include callout.html content="**`cpu_operator_cost (floating point)`**: Planlayıcının bir sorgu sırasında yürütülen her operatör veya fonksiyonu işleme maliyetine ilişkin tahminini ayarlar. Öntanımlı değeri 0,0025'tir." type="primary" %}

{% include callout.html content="**`parallel_setup_cost (floating point)`**: Planlayıcının paralel worker süreçlerini başlatma maliyetine ilişkin tahminini ayarlar. Öntanımlı değeri 1000'dir." type="primary" %}

{% include callout.html content="**`parallel_tuple_cost (floating point)`**: Planlayıcının bir tuple'ı paralel worker sürecinde başka bir sürece aktarmanın maliyetine ilişkin tahminini ayarlar. Öntanımlı değeri 0.1'dir." type="primary" %}

{% include callout.html content="**`min_parallel_table_scan_size (integer)`**: Paralel scan'in dikkate alınması için taranması gereken minimum tablo verisi boyutunu ayarlar. Paralel sequential scan'de taranan tablo verisi miktarı her zaman tablo boyutuna eşittir, ancak indeks kullanıldığında bu boyut normalde daha az olacaktır. Bu değer birim olmadan belirtilirse bloklar ( BLCKSZ bayt, genellikle 8kB'dir ) olarak alınır. Öntanımlı değeri 8 megabayttır (8MB)." type="primary" %}

{% include callout.html content="**`min_parallel_index_scan_size (integer)`**: Paralel scan'in dikkate alınması için taranması gereken minimum indeks verisi boyutunu ayarlar. Paralel indeks scan'in genellikle indeksin tamamına bakmaz. Bu, planlayıcının taramada gerçekten dokunulacağına inandığı ilgili sayfaların sayısıdır. Bu parametre aynı zamanda belirli bir indeksin paralel vacuum katılıp katılamayacağına karar vermek için de kullanılır. ( bkz. [VAKUM](https://www.postgresql.org/docs/current/sql-vacuum.html)). Bu değer birim olmadan belirtilirse bloklar ( BLCKSZ bayt, genellikle 8kB'dir ) olarak alınır. Öntanımlı değeri 512 kilobayttır (512kB)." type="primary" %}

{% include callout.html content="**`effective_cache_size (integer)`**: Planlayıcının, bir sorgu için kullanılabilir etkin disk önbelleği boyutu hakkındaki varsayımını belirtir. Bu değer indeks kullanmanın maliyet tahminlerinde hesaba katılır. Daha yüksek bir değer, index scan kullanılma olasılığını artırır; daha düşük bir değer, sequential scan kullanılma olasılığını artırır. Bu parametreyi ayarlarken, PostgreSQL shared buffer ve veri dosyalarını içeren işletim sistemi önbelleğini dikkate almalısınız. Bazı veriler her ikisinde de mevcut olabilir. Ayrıca, kullanılabilir alanı paylaşmak zorunda kalacakları için farklı tablolardaki eşzamanlı sorgu sayısını dikkate alın. Bu parametrenin PostgreSQL tarafından ayrılan shared memory boyutu ve çekirdek disk önbelleği üzerinde bir etkisi yoktur. Sadece tahmin amaçlı kullanılır. Sistem ayrıca sorgular arasında disk önbelleğinde veri kaldığını varsaymaz. Bu değer birisiz belirtildiğinde bloklar olarak alınır (BLCKSZ bayt, genellikle 8kB'dir). Öntanımlı değeri 4 gigabayttır (4 GB). (BLCKSZ 8kB değilse, varsayılan değer bununla orantılı olarak ölçeklenir.)" type="primary" %}

{% include callout.html content="**`jit_above_cost (floating point)`**: Etkinleştirilmişse, JIT ( Just-in-Time ) derlemesinin aktif olduğu sorgu maliyetini belirtir (bkz. [](https://www.postgresql.org/docs/current/jit.html)). JIT gerçekleştirmek, planlama süresi maliyeti getirir ancak sorgu yürütmeyi hızlandırabilir. Bu parametreyi -1 olarak ayarlamak JIT derlemesini devre dışı bırakır. Öntanımlı değeri 100000'dir." type="primary" %}

{% include callout.html content="**`jit_inline_above_cost (floating point)`**: JIT derlemesinin, inline işlevler ve operatörler yapmaya çalıştığı sorgu maliyetini belirtir. Inlining, planlama süresini beraberinde getirir ancak yürütme hızını artırabilir. Bu paremetreyi `jit_above_cost`'tan daha küçük bir değere ayarlamak anlamlı değildir. Bu parametre için -1 değeri Inlining'i devre dışı bırakır. Öntanımlı değeri 500000'dir." type="primary" %}

{% include callout.html content="**`jit_optimize_above_cost (floating point)`**: JIT derlemesinin maliyetli optimizasyonları uyguladığı sorgu maliyetini ayarlar. Bu tür bir optimizasyon planlama süresine mal olur, ancak yürütme hızını artırabilir. Bu parametreyi, `jit_above_cost`'tan daha küçük bir değere ayarlamak anlamlı değildir ve `jit_inline_above_cost`'tan daha fazlasına ayarlamak pek faydalı olmayacaktır. Bu parametre için -1 değeri maliyetli optimizasyonları devre dışı bırakır. Öntanımlı değeri 500000'dir." type="primary" %}

### Genetic Query Optimizer

[Genetic query optimizer (GEQO)](https://www.postgresql.org/docs/current/geqo.html), sezgisel arama (heuristic searching) kullanarak sorgu planlaması yapan bir algoritmadır. GEQO karmaşık sorgular için planlama süresini azaltır.

{% include callout.html content="**`geqo (boolean)`**: Genetik sorgu optimizasyonunu etkinleştirir / devre dışı bırakır. Varsayılan olarak açıktır. Production'da kapatmamak tavsiye edilir. `geqo_threshold` parametresi, GEQO'nun daha ayrıntılı denetimini sağlar." type="primary" %}

{% include callout.html content="**`geqo_threshold (integer)`**: En az bu parametre değeri kadar FROM öğesi içeren sorguları planlamak için genetik sorgu optimizasyonunu kullanır. Bir `FULL OUTER JOIN` yapısı yalnızca bir FROM öğesi olarak sayılır. Öntanımlı değeri 12'dir. Daha basit sorgular için normal (exhaustive-search) planlayıcı yeterlidir. Birçok tablo içeren sorgular için exhaustive-search genellikle yetersiz bir planı uygulama cezasından daha uzun sürer. Bu nedenle, sorgunun boyutuna ilişkin bir eşik, GEQO kullanımını yönetmenin uygun bir yoludur." type="primary" %}

{% include callout.html content="**`geqo_effort (integer)`**: GEQO'da planlama süresi ve sorgu planı kalitesi arasındaki dengeyi kontrol eder. Bu değişken, 1 ile 10 arasında değerler alır. Öntanımlı değeri 5'tir. Daha büyük değerler sorgu planlaması için harcanan zamanı artırırken verimli bir sorgu planının seçilme olasılığını artırır.<br/><br/>

`geqo_effort` aslında doğrudan hiçbir şey yapmaz, yalnızca GEQO davranışını etkileyen diğer değişkenlerin varsayılan değerlerini hesaplamak için kullanılır. Diğer parametreleri isterseniz elle de ayarlayabilirsiniz." type="primary" %}

{% include callout.html content="**`geqo_pool_size (integer)`**: GEQO tarafından kullanılan havuz boyutunu, yani genetik popülasyondaki birey sayısını kontrol eder. Bu değer en az 2 olmalıdır, önerilen değerler 100 ile 1000 arasıdır. 0 olarak ayarlanırsa (öntanımlı ayar), `geqo_effort` ve sorgudaki tabloların sayısına göre uygun bir değer seçilir." type="primary" %}

{% include callout.html content="**`geqo_generations (integer)`**: GEQO tarafından kullanılan nesil sayısını, yani algoritmanın yineleme sayısını kontrol eder. Bu değer en az 1 olmalıdır ve önerilen değerler havuz boyutuyla aynı aralıktadır. 0 değerine (varsayılan) ayarlandığında `geqo_pool_size`'a dayalı uygun bir değer seçilir." type="primary" %}

{% include callout.html content="**`geqo_selection_bias (floating point)`**: GEQO tarafından kullanılan seçim eğilimini denetler. Seçim eğilimi, popülasyon içindeki seçici baskıdır. 1,50 ile 2,00 arasında değerler alabilir. 2.0 varsayılandır." type="primary" %}

{% include callout.html content="**`geqo_seed (floating point)`**: GEQO tarafından rastgele yollar seçmek için kullanılan rastgele sayı üretecinin başlangıç ​​değerini kontrol eder. Değer 0 (varsayılan) ile 1 arasında değerler alır. Değerin değiştirilmesi, keşfedilen birleştirme yolları kümesini değiştirir ve daha iyi / kötü bir en iyi yolun bulunmasıyla sonuçlanabilir." type="primary" %}

### Diğer Planlayıcı Seçenekleri

{% include callout.html content="**`default_statistics_target (integer)`**: `ALTER TABLE SET STATISTICS` ile sütuna özgü bir hedef ayarlanmamış tablo sütunları için varsayılan istatistik hedefini ayarlar. Daha büyük değerler ANALYZE yapmak için gereken maliyeti artırmakla birlikte planlayıcının tahmin kalitesini artırabilir. Öntanımlı değeri 100'dür. PostgreSQL sorgu planlayıcı tarafından istatistiklerin kullanımı hakkında daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/planner-stats.html)." type="primary" %}

{% include callout.html content="**`constraint_exclusion (enum)`**: Sorgu planlayıcısının sorguları optimize etmek için tablo kısıtlamalarını kullanmasını kontrol eder. `constraint_exclusion` parametresinin alabileceği değerler `on` (tüm tablolar için kısıtlamaları incele), `off` (kısıtlamaları asla inceleme) ve `partition` (kısıtlamaları yalnızca miras alt tabloları ve UNION ALL alt sorguları için incele). `partition` varsayılan ayardır. Genellikle performansı artırmak için geleneksel miras ağaçlarıyla birlikte kullanılır.<br/><br/>

Bu parametre belirli bir tablo için izin verdiğinde, planlayıcı sorgu koşullarını tablonun CHECK kısıtlamalarıyla karşılaştırır ve koşulların kısıtlamalarla çeliştiği tarama tablolarını atlar. Örneğin:" type="primary" %}

```sql
CREATE TABLE parent(key integer, ...);
CREATE TABLE child1000(check (key between 1000 and 1999)) INHERITS(parent);
CREATE TABLE child2000(check (key between 2000 and 2999)) INHERITS(parent);
...
SELECT * FROM parent WHERE key = 2400;
```

{% include callout.html content=" constraint exclusion etkinleştirildiğinde, verilen SELECT, child1000'i hiç taramayacak ve performansı artıracaktır.<br/><br/>
Şu anda constraint exclusion, yalnızca kalıtım ağaçları (inheritance tree) aracılığıyla tablo bölümlendirme (partitioning) yapmak için varsayılan olarak etkinleştirilmiştir. Bunu tüm tablolar için açmak, basit sorgularda kayda değer derecede ekstra planlama yükü getirir ve çoğu durumda basit sorgular için hiçbir fayda sağlamaz. Geleneksel kalıtım kullanılarak bölümlenmiş tablonuz yoksa tamamen kapatılabilir.<br/><br/>

Partitioning için constraint exclusion kullanımı hakkında daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/ddl-partitioning.html#DDL-PARTITIONING-CONSTRAINT-EXCLUSION)." type="primary" %}

{% include callout.html content="**`cursor_tuple_fraction (floating point)`**: Planlayıcının, imleç satırlarının alınacak kısmına ilişkin tahminini ayarlar. Öntanımlı değeri 0.1'dir. Bu parametre için daha küçük değerler planlayıcıyı imleçler için 'fast start' planlarını kullanmaya yönlendirir. Bu şekilde, ilk birkaç satırı hızlı bir şekilde alırnırken tüm satırları getirmesi uzun zaman alabilir. Daha büyük değerler, toplam tahmini süreye daha fazla önem verir. Maksimum 1.0 ayarında, imleçler tam olarak normal sorgular gibi planlanarak ilk satırların ne kadar çabuk teslim edilebileceğini değil, yalnızca toplam tahmini süreyi göz önünde bulundurur." type="primary" %}

{% include callout.html content="**`from_collapse_limit (integer)`**: Planlayıcı, sonuçta ortaya çıkan FROM listesi bu değerden çok öğe içermiyorsa, alt sorguları üst sorgularda birleştirir. Daha küçük değerler planlama süresini azaltır, ancak daha kalitesiz sorgu planlarına neden olabilir. Öntanımlı değeri 8'dir. Daha fazla bilgi için bkz [](https://www.postgresql.org/docs/current/explicit-joins.html).<br/><br/>

Bu değerin `geqo_threshold`'a veya daha fazlasına ayarlanması, GEQO planlayıcısının kullanımını tetikleyerek optimal olmayan planlarla sonuçlanabilir. Bölüm 19.7.3'e bakınız." type="primary" %}

{% include callout.html content="**`jit (boolean)`**: JIT derlemesinin PostgreSQL tarafından kullanılıp kullanılamayacağını belirler (bkz. [](https://www.postgresql.org/docs/current/jit.html)). Varsayılan açıktır." type="primary" %}

{% include callout.html content="**`join_collapse_limit (integer)`**: Bu parametre değerinden az öğe içeren bir liste ortaya çıktığında planlayıcı explicit JOIN yapılarını (FULL JOIN'ler hariç) FROM-list'de yeniden yazar. Daha küçük değerler planlama süresini azaltır, ancak daha kalitesiz sorgu planlarına neden olabilir.<br/><br/>

Bu değişken, varsayılan olarak çoğu kullanım için uygun olan `from_collapse_limit` parametresi ile aynı şekilde ayarlanır. 1 olarak ayarlamak, explicit JOIN'lerin yeniden sıralanmasını önler. Bu nedenle, sorguda belirtilen explicit join sırası, ilişkiler join edildikten sonra değişmez. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/explicit-joins.html).<br/><br/>

Bu değerin `geqo_threshold` veya daha fazlasına ayarlanması, GEQO planlayıcısının kullanımını tetikleyerek optimal olmayan planlar getirebilir. bkz. [Genetic Query Optimizer](mydoc_sorgu_planlama.html#genetic-query-optimizer)." type="primary" %}

{% include callout.html content="**`parallel_leader_participation (boolean)`**: Lider sürecin, worker süreçleri beklemek yerine `Gather` ve `Gather Merge` düğümleri altında sorgu planı yürütmesini kontrol eder. Varsayılan `on`'dur. Bu parametrenin `off` olarak ayarlanması, liderin yeterince hızlı tuple'ları okumaması nedeniyle worker'ların bloke olma ihtimalini azaltır. Ancak lider süreç ilk tuple'lar üretilmeden önce worker süreçlerin başlamasını beklemelidir. Liderin performansa ne ölçüde yardımcı olabileceği veya performansı engelleyebileceği, plan türüne, worker sayısına ve sorgu süresine bağlıdır." type="primary" %}

{% include callout.html content="**`force_parallel_mode (enum)`**: Paralel sorgu imkanlarını test amaçlarınız için performans katkısı olmasa dahi kullanmaya zorlar. `force_parallel_mode` için geçerli değerler; `off` (paralel modu, yalnızca performansı iyileştirmesi beklendiğinde kullanır), `on` (güvenli olduğu düşünülen tüm sorgular için paralel sorguyu zorlar) ve `regress` (aşağıda açıklanan ek davranışlar ile birlikte `on` gibidir).<br/><br/>

`on` ayarı, herhangi bir sorgu planının üstüne bir `Gather` düğümü ekler. Böylece sorgu, bir paralel worker'ın içinde çalıştırılır. Bu seçenek ayarlandığında hatalar veya beklenmeyen sonuçlar ortaya çıktığında, sorgu tarafından kullanılan bazı işlevlerin `PARALLEL UNSAFE` veya `PARALLEL RESTRICTED` olarak işaretlenmesi gerekebilir.<br/><br/>

`regress` ayarı, `on` ayarının yaptığı etkiler ile beraber otomatik regression testini kolaylaştırmayı amaçlayan bazı ek etkilere sahiptir. Normalde, paralel bir worker'dan gelen mesajlar bunu belirten bir bağlam satırı içerir, ancak `regress` ayarı bu satırı bastırır ve çıktı paralel olmayan yürütmeyle aynı olur. Ayrıca, bu ayar ile planlara eklenen `Gather` düğümleri `EXPLAIN` çıktısında gizlenir. Böylece çıktı, `off` durumunda elde edilecek olanla eşleşir." type="primary" %}

{% include callout.html content="**`plan_cache_mode (enum)`**: Prepared statement'lar custom ve generic planlar kullanılarak yürütülebilir. Her bir yürütme için custom planlar, kendine özgü parametre değerleri kümesi kullanılarak baştan yapılırken; generic planlar, parametre değerlerine dayanmaz ve yürütmelerde tekrar kullanılabilir. Bu sebeple, generic bir planın kullanılması planlama süresinden tasarruf sağlar, ancak ideal plan güçlü bir şekilde parametre değerlerine bağlıysa generic bir plan verimsiz olabilir. Bu seçenekler arasındaki tercih normalde otomatik olarak yapılır, ancak `plan_cache_mode` ile geçersiz kılınabilir. Alabileceği değerler; `auto` (varsayılan), `force_custom_plan` ve `force_generic_plan`'dır. Daha fazla bilgi için bkz. [PREPARE](https://www.postgresql.org/docs/current/sql-prepare.html)." type="primary" %}

{% include links.html %}
