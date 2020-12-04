---
title: "Write Ahead Log"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 03, 2020
summary: "Write Ahead Log"
sidebar: mydoc_sidebar
permalink: mydoc_wal.html
folder: mydoc
---

## Write Ahead Log

Bahsedilecek ayarların yapılmasıyla ilgili ek bilgi için [WAL Yapılandırması](https://www.postgresql.org/docs/current/wal-configuration.html) bölümüne bakın.

### Ayarlar

{% include callout.html content="**`wal_level (enum)`**: `wal_level`, WAL'a ne kadar bilgi yazılacağını belirler. Varsayılan değer `replica`'dır. replica değeri bir standby sunucu üzerinde çalışan read-only sorgular dahil, WAL archiving ve replication işlemleri için yeterli veriyi sağlar. `minimal`, bir çökme veya ani kapanma durumunda kurtarma işlemleri için gereken bilgileri garantiye alır, toplu veri operaslarıyla ilgili işlemleri (`CREATE TABLE AS SELECT`, `CREATE INDEX`) WAL günlüğüne kaydetmeyerek depolama avantajı sağlar. Son olarak `logical`, logical decoding ve logical replication için gerekli bilgileri sağlar. Her düzey, tüm alt düzeylerde kaydedilen bilgileri içerir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

`minimal` seviyede, kalıcı ilişkiler için onları oluşturan veya yeniden yazan transaction'ın geri kalan bilgileri günlüğe yazılmaz. Bu, operasyonları çok daha hızlı hale getirir. Bu optimizasyonu başlatan operasyonlar şunlardır:" type="primary" %}

```sql
ALTER ... SET TABLESPACE
CLUSTER
CREATE TABLE
REFRESH MATERIALIZED VIEW (without CONCURRENTLY)
REINDEX
TRUNCATE
```

{% include callout.html content=" minimal WAL, verileri base backup ve WAL günlüklerinden yeniden yapılandırmak için yeterli bilgi içermez, bu nedenle WAL archiving ve streaming replication işlemleri için `replica` veya üstü kullanılmalıdır.<br/><br/>

`logical` seviyesi, replica seviyesinde kaydedilen bilgiler ek, WAL'dan mantıksal değişikliklerin ayıklanmasına olanak sağlamak için gereken bilgileri günlüğe kaydeder. logical seviyesini kullanmak, özellikle birçok tablo `REPLICA IDENTITY FULL` olarak yapılandırılmışsa ve fazla `UPDATE` ve `DELETE` ifadesi yürütülüyorsa WAL hacmini artıracaktır." type="info" %}

{% include callout.html content="**`fsync (boolean)`**: PostgreSQL sunucusu bu parametre açıksa, `fsync ()` sistem çağrıları veya çeşitli eşdeğer yöntemler yayınlayarak değişikliklerin fiziksel olarak diske yazıldığından emin olmaya çalışır. Bu, veritabanı kümesinin bir işletim sistemi veya donanım çökmesinden sonra tutarlı bir duruma geri yüklenebilmesini sağlar.<br/><br/>

`fsync`'i kapatmak genellikle bir performans avantajı olsamasına rağmen bir elektrik kesintisi veya sistem çökmesi durumunda kurtarılamaz veri bozulmasına neden olabilir. Bu nedenle, yanlızca tüm veritabanınızı dış verilerden kolayca yeniden oluşturabiliyorsanız `fsync`'i kapatmanız önerilir.<br/><br/>

fsync'i kapatmak için güvenli koşullara, yeni veritabanı kümesinin bir yedekleme dosyasından ilk yüklenmesi, sık sık yeniden oluşturulan ve failover için kullanılmayan read-only bir veritabanı klonu örnek olarak verilebilir. Yüksek kaliteli donanım tek başına fsync'i kapatmak için yeterli bir gerekçe değildir.<br/><br/>

`fsync` kapılıdan açık duruma getirildiğinde güvenilir kurtarma sağlamak için, çekirdekteki tüm değiştirilmiş buffer'ları dayanıklı depolamaya zorlamak gerekir. Bu, küme kapalıyken veya `fsync` açıkken `initdb --sync-only` komutuyla, `sync` çalıştırılarak, dosya sisteminin bağlantısını keserek veya sunucuyu yeniden başlatarak yapılabilir.<br/><br/>

Çoğu durumda, kritik olmayan transactionlar için `synchronous_commit`'i kapatmak, `fsync`'i kapatmanın getireceği potansiyel performans avantajının fazlasını veri bozulmasına bağlı riskler olmadan sağlayabilir.<br/><br/>

fsync, yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. Bu parametreyi kapatırsanız, `full_page_writes`'i de kapatmak düşünülebilir." type="primary" %}

{% include callout.html content="**`synchronous_commit (enum)`**: Veritabanı sunucusu istemciye bir 'success' işareti döndürmeden önce ne kadar WAL işlemenin tamamlanması gerektiğini belirtir. Geçerli değerler `remote_apply`, `on` (varsayılan), `remote_write`, `local` ve `off` şeklindedir.<br/><br/>

`synchronous_standby_names` boşsa, anlamlı ayarlar yalnızca `on` ve `off`'dur; `remote_apply`, `remote_write` ve `local`, `on` aynı yerel senkronizasyon seviyesini sağlar. `off` olmayan tüm modların yerel davranışı, WAL'ın diske yerel olarak flush edilmesini beklemektir. `off` modunda bekleme yoktur, bu nedenle istemciye başarının bildirilmesi ile transaction'ın sonrasında bir sunucu çökmesine karşı güvenli olmasının garanti edilmesi arasında bir gecikme olabilir. (Maksimum gecikme `wal_writer_delay`'in üç katıdır.) fsync'in aksine, bu parametrenin `off` olarak ayarlanması herhangi bir veritabanı tutarsızlığı riski oluşturmaz. İşletim sistemi veya veritabanı çökmesi, yakın zamanda taahhüt edildiği iddia edilen bazı transaction'ların kaybolmasına neden olabilir, ancak veritabanı durumu bu transaction'lar temiz bir şekilde iptal edilmiş gibi olacaktır. Bu nedenle, işlemin dayanıklılığı konusunda kesinlikten yerine performansın daha önemli olduğu durumlarda `synchronous_commit`'i kapatmak faydalı olabilir. Daha fazlası için [Asynchronous Commit](https://www.postgresql.org/docs/current/wal-async-commit.html) bölümüne bakın.<br/><br/>

`synchronous_standby_names` boş değilse, `synchronous_commit` aynı zamanda transaction commit WAL kayıtlarının standby sunucularda işlenmesini bekleyip beklemeyeceğini de kontrol eder.<br/><br/>

`remote_apply` olarak ayarlandığında commit'ler, mevcut senkron standby'lar gelen transaction commit kaydını aldıklarını ve uyguladıklarını gösterene kadar bekleyecektir, böylece standby'lardaki sorgularda görünür hale gelecek ve standby sunucudaki dayanıklı depolamaya yazılacaktır. Bu, WAL replay için beklendiğinde önceki ayarlara göre çok daha büyük commit gecikmelerine neden olacaktır. Açık olarak ayarlandığında commit'ler, mevcut senkron standby gelen yanıtlar transaction commit kayıtlarını aldıklarını ve dayanıklı depolamaya yazdıklarını belirtene kadar bekler. Bu, hem primary hem de tüm senkron standby veritabanı depolamalarının bozulmasına maruz kalmadıkça transaction kaybolmamasını sağlar. `remote_write` olarak ayarlandığında commit'ler, mevcut senkron standby'lar transaction'ın commit kaydını aldıklarını ve dosya sistemlerine yazdıklarını bildirene kadar bekleyecektir. Bu ayar, bir standby örneğinin beklenmedik şekilde çökmesi durumunda verilerin korunmasını sağlar. Ancak işletim sistemi düzeyinde bir çökme yaşanırsa bu durum geçerli değildir, çünkü standby modundayken verilerin kalıcı bir depolamaya ulaşması gerekmez. `local` ayarı, commit'lerin diske yerel flush edilmesi için beklemesine neden olurken replication için bu geçerli değildir. Bu, synchronous replication kullanımdayken genellikle istenilmez ancak bütünlük için sağlanır. <br/><br/>

Bu parametre herhangi bir zamanda değiştirilebilir. Herhangi bir transaction'ın davranışı, commit edildiğinde geçerli olan ayar tarafından belirlenir. Bu nedenle, bazı transaction'ların senkron, bazılarının asenkron olarak commit edilmesi mümkün ve yararlıdır. Örneğin, tek bir multistatement transaction'ı asenkron şekilde commit etmek için transaction içinde `SET LOCAL synchronous_commit TO OFF` verilebilir." type="primary" %}

Aşağıdaki tabloda `synchronous_commit` ayarlarının yeteneklerini özetlemektedir.

| synchronous_commit setting | local durable commit | standby durable commit after PG crash | standby durable commit after OS crash | standby query consistency |
|-------|--------|-------|--------|-------|
| remote_apply | + | + | + | + |
| on | + | + | + | - |
| remote_write | + | + | - | - |
| local | + | - | - | - |
| off | - | - | - | - |

{% include callout.html content="**`wal_sync_method (enum)`**: WAL değişikliklerini diske göndermeye zorlamak için kullanılan yöntem. `fsync` kapalıysa bu ayar geçersizdir. Olası değerler şunlardır: `open_datasync`, `fdatasync`, `fsync`, `fsync_writethrough`, `open_sync`. <br/><br/>

Verilen seçenekler tüm platformlarda mevcut değildir. `fdatasync` Linux'ta varsayılan değerdir. Diğerleri için varsayılan, yukarıdaki listede platform tarafından desteklenen ilk yöntemdir. Varsayılan mutlaka en iyisi değildir; Kilitlenmeye karşı korumalı bir yapılandırma oluşturmak veya optimum performans elde etmek için bu ayarı veya sistem yapılandırmanızın diğer yönlerini değiştirmeniz gerekebilir. Bu parametre yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. Bu yönler [Reliability](https://www.postgresql.org/docs/current/wal-reliability.html) bölümünde ele alınmıştır." type="primary" %}

{% include callout.html content="**`full_page_writes (boolean)`**: PostgreSQL sunucusu bu parametre açık olduğunda, herbir disk page'inin tüm içeriğini ilgili page'in checkpoint'den sonraki ilk değişikliği sırasında WAL'a yazar. İşletim sistemi çökmesi sırasında işlemde olan bir page yazma işlemi kısmen tamamlanarak disk üzerinde eski ve yeni verilerin karışımını içeren bir page'ye yol açacağı için bu gereklidir. Böyle bir page için WAL'da depolanan satır düzeyinde değişiklik verileri çökme sonrası kurtarma işleminde tamamen geri yüklemek için yeterli olmayacaktır. Tüm page'in saklanması, page'in doğru bir şekilde geri yüklenmesini garanti ederken beraberinde WAL'a yazılması gereken veri miktarını artırır. (WAL replay her zaman bir checkpoint'den başlar, bunu bir checkpoint'den sonra her sayfanın ilk değişikliği sırasında yapması yeterlidir. Bu nedenle, Tüm page'i yazma maliyetini azaltmanın bir yolu, checkpoint aralığı parametrelerini artırmaktır.)<br/><br/>

Bu parametrenin kapatılması çalışmayı hızlandırır ancak bir sistem arızasından sonra kurtarılamayan veri bozulmalarına neden olabilir. Riskler, daha küçük olsa da fsync'i kapatmaya benzer ve yalnızca bu parametre için önerilen koşullara göre kapatılmalıdır.<br/><br/>

Bu parametrenin kapatılması, point-in-time recovery (PITR) için WAL arşivleme kullanımını etkilemez.<br/><br/>

Bu parametre yalnızca postgresql.conf dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan açıktır. " type="primary" %}

{% include callout.html content="**`wal_log_hints (boolean)`**:

Bu parametre `on` olduğunda, PostgreSQL sunucusu, hint bits olarak bilinen kritik olmayan değişiklikler için bile bir checkpoint'den sonra ilgili page'in ilk değişikliği sırasında her disk page'inin tüm içeriğini WAL'a yazar.<br/><br/>

data checksums etkinleştirilirse, hint bit değişiklikleri her zaman WAL'a yazılır ve bu ayar yok sayılır. Veritabanınızda data checksums etkinleştirilmişse, fazladan ne kadar WAL-logging yapılacağını test etmek için bu ayarı kullanabilirsiniz.<br/><br/>

Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Varsayılan değer `off`." type="primary" %}

{% include callout.html content=" **`wal_compression (boolean)`**: Bu parametre açık olduğunda PostgreSQL sunucusu, `full_page_writes` açıkken ve base backup sırasında WAL'a yazılan bir full page görüntüyü sıkıştırır. WAL replay sırasında sıkıştırılmış bir page görüntüsü açılacaktır. Varsayılan değer `off`'dur. Bu ayarı yalnızca süper kullanıcılar değiştirebilir. <br/><br/>

Bu parametrenin açılması kurtarılamaz veri bozulması riskini artırmaksızın WAL boyutunu azaltır, ancak WAL logging sırasında sıkıştırma ve WAL replay sırasında sıkıştırmanın açılmasından kaynaklı fazladan CPU harcanmasına neden olur." type="primary" %}

{% include callout.html content="**`wal_init_zero (boolean)`**: `on` olarak ayarlanırsa (varsayılan), bu seçenek yeni WAL dosyalarını sıfırlarla doldurulur. Bununla bazı dosya sistemlerinde WAL kayıtlarını yazmadan önce alanın tahsis edilmesi sağlanır. Ancak, *Copy-On-Write* (COW) dosya sistemleri bu teknikten yararlanamayabilir, bu nedenle gereksiz çalışmayı atlama seçeneği verilir. `off` olarak ayarlanırsa, dosya oluşturulduğunda beklenen boyuta sahip olması için yalnızca son bayt yazılır." type="primary" %}

{% include callout.html content="**`wal_recycle (boolean)`**: `on` olarak ayarlanırsa (varsayılan), WAL dosyalarını yeniden adlandırarak geri kullanılmalarını sağlar. Böylece yeni dosya oluşturma yükünden kurtarabilir. COW dosya sistemlerinde yenilerini oluşturmak daha hızlı olabildiğinden bu davranışı devre dışı bırakma seçeneği verilmiştir." type="primary" %}

{% include callout.html content="**`wal_buffers (integer)`**: Henüz diske yazılmamış WAL verileri için kullanılan paylaşılan bellek miktarıdır. Öntanımlı -1 ayarı, shared_buffers'ın 1 / 32'ine eşit boyutu seçer, ancak 64kB'den az veya bir WAL segmentinin boyutundan (tipik boyut 16MB) daha fazla olmamalıdır. Otomatik seçim çok büyük veya çok küçükse bu değer elle ayarlanabilir, ancak 32kB'den küçük herhangi bir pozitif değer 32kB olarak değerlendirilecektir. Bu değer birim olmadan belirtilirse, WAL blokları olarak alınır, yani XLOG_BLCKSZ bayt tipik olarak 8kB. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.

WAL buffer'ların içeriği her transaction commit'inde diske yazıldığından çok büyük değerlerin önemli bir fayda sağlayacağı söylenmeyebilir. Ancak, bu değeri en az birkaç megabayta ayarlamak, birçok istemcinin aynı anda commit'te bulunduğu yoğun bir sunucuda yazma performansını artırabilir. Öntanımlı -1 ayarı ile seçilen otomatik ayarlama çoğu durumda makul sonuçlar verir." type="primary" %}

{% include callout.html content="**`wal_writer_delay (integer)`**: WAL writer'ın WAL'ı zaman cinsinden ne sıklıkla temizleyeceğini belirtir. WAL temizledikten sonra, asenkron commit edilen bir transaction ile daha erken uyanmadıkça, WAL writer `wal_writer_delay` süresince uyur. Son temizleme, `wal_writer_delay` öncesinde gerçekleştiyse ve bu zamandan beri `wal_writer_flush_after` değerinden daha az WAL üretildiyse, WAL kayıtları yalnızca işletim sistemine yazılır, diske temizlenmez. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan değer 200 milisaniyedir (200 ms). Birçok sistemde uyku gecikmelerinin etkili çözümünün 10 milisaniyedir. `wal_writer_delay` parametresini 10'un katı olmayan bir değere ayarlamak 10'un bir sonraki katına ayarlamakla aynı sonuçları verebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_writer_flush_after (integer)`**: WAL writer'ın volume cinsinden WAL'ı ne sıklıkla temizlediğini belirtir. Son temizleme, `wal_writer_delay` öncesinden gerçekleştiyse ve o zamandan beri `wal_writer_flush_after` değerinden daha az WAL üretildiyse, WAL yalnızca işletim sistemine yazılır, diske temizlenmez. `wal_writer_flush_after` 0 olarak ayarlanmışsa, WAL verileri anında temizlenir. Bu değer birim olmadan belirtilirse WAL blokları olarak alınır. (XLOG_BLCKSZ bayt, tipik olarak 8kB). Öntanımlı 1MB'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_skip_threshold (integer)`**: `wal_level = minimal` olduğunda ve bir transaction kalıcı bir ilişki oluşturduktan veya yeniden yazıldıktan sonra commit edildiğinde, bu ayar yeni verilerin nasıl kalıcı hale getirileceğini belirler. Veriler bu ayardan küçükse WAL log'larına yazınlır değilse fsync özelliği kullanılır. Depolamanızın özelliklerine bağlı olarak bu tür commitler senkron transaction'ları yavaşlatıyorsa, bu değeri yükseltmek veya düşürmek yardımcı faydalı olabilir. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Varsayılan, iki megabayttır (2MB)." type="primary" %}

{% include callout.html content="**`commit_delay (integer)`**: Bu parametrenin ayarlanması bir WAL temizliği başlatılmadan önce gecikme süresi ekler. Daha fazla sayıda transaction'ın tek bir WAL temizleme yoluyla commit edilmesine imkan vererek grup commit verimini artırır. Bu, sistem yükünün verilen aralıkta ek transaction'ları commit etmeye hazır hale gelmesine yetecek kadar yüksek olduğu durumda gerçekleşir. Ancak bu, her WAL temizleme için `commit_delay`'e kadar gecikmeyi de artırır. Gecikme, hiçbir transaction'ın commit edilmeye hazır olmadığında boşa gittiğinden, gecikme yalnızca en az `commit_siblings` kadar transaction bir temizleme başlatılmak üzereyken aktifse gerçekleştirilir. Ayrıca, fsync devre dışı bırakılırsa gecikme yapılmaz. Bu değer birimsiz belirtilirse, mikrosaniye olarak alınır. commit_delay öntanımlı 0 (gecikme yok). Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`commit_siblings (integer)`**: commit_delay gecikmeyi gerçekleştirmeden önce ihtiyaç duyulacak minimum eşzamanlı açık işlem sayısını belirtir. Daha büyük değerler gecikme aralığı sırasında en az bir tane daha transaction'ın commit edilmeye hazır hale gelme olasılığını arttırır. Varsayılan, 5 transaction'dır." type="primary" %}

### Checkpoints

{% include callout.html content="**`checkpoint_timeout (integer)`**: Otomatik WAL checkpoint'leri arasında maksimum süreyi belirtir. Bu değer birimleri olmadan belirtilirse, saniye olarak alınır. Geçerli aralık 30 saniye ile 1 gün arasındadır. Varsayılan değer 5 dakikadır (5min). Bu parametrenin artırılması, crash recovery için gerekli süreyi artırabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_completion_target (floating point)`**: checkpoint'ler arasındaki toplam sürenin bir bölümü olarak checkpoint tamamlama hedefini belirtir. Öntanımlı 0,5'tir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_flush_after (integer)`**: Bir checkpoint gerçekleştirilirken bu miktardan daha fazla veri yazıldığında, bunları işletim sistemi depolamasına yazmaya zorlar. Bunu yapmak, çekirdeğin page önbelleğindeki kirli veri miktarını sınırlayarak, checkpoint sonunda fsync yayınlandığında ve işletim sistemi verileri arka planda daha büyük gruplar halinde geri yazdığı durumlarda durma olasılığını azaltır. Çoğu zaman bu, büyük ölçüde azalmış transaction gecikmesiyle sonuçlanır, ancak işletim sisteminin page önbelleğinden daha küçük, `shared_buffers`'dan büyük iş yüklerinde performansın düşebileceği durumlar da vardır. Bazı platformlarda bu ayarın hiçbir etkisi olmayabilir. Bu değer birimsiz belirtilirse bloklar olarak alınır,yani BLCKSZ bayt (tipik olarak 8kB'dir). Geçerli aralık zorunlu geri yazmayı devre dışı bırakan 0 ile 2MB arasıdır. Öntanımlı Linux'ta 256kB, diğer sistemlerde 0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_warning (integer)`**: WAL segment dosyalarının doldurulmasından kaynaklanan checkpoint'ler birbirine bu süreden daha yakın olursa sunucu günlüğüne bir mesaj yazar. Bu, `max_wal_size` değerinin yükseltilmesi gerektiğine işaret eder. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı 30 saniyedir (30s). 0 uyarıyı devre dışı bırakır. `checkpoint_timeout`, `checkpoint_warning` değerinden azsa hiçbir uyarı üretilmez. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`min_wal_size (integer)`**: WAL disk kullanımı bu ayarın altında kaldığı sürece, eski WAL dosyaları silmek yerine bir checkpoint'de tekrar kullanmak için geri dönüştürülür. Bu, örneğin büyük iş yüklerinde WAL kullanımındaki ani artışlarını karşılamak için yeterli WAL alanının ayrıldığından emin olmak için kullanılabilir. Bu değer birimsiz belirtilirse megabayt olarak alınır. Öntanımlı 80 MB'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Archiving

{% include callout.html content="**`archive_mode (enum)`**: Bu parametre etkinleştirildiğinde, tamamlanan WAL segmentleri `archive_command` ayarlanarak arşiv depolamaya gönderilir. `off`, `on` ve `always` modları vardır. Normal çalışmada `on` ile `always` modu arasında fark yoktur, ancak `always` olarak ayarlandığında archive recovery ve standby modunda da WAL arşivleyici etkinleştirilir. `always` modunda, arşivden geri yüklenen veya streaming replication ile akışa alınan tüm dosyalar arşivlenecektir. Ayrıntılar için Bölüm [Log-Shipping Standby Servers]('') bölümüne bakın.<br/><br/>

`archive_mode` ve `archive_command` ayrı değişkenlerdir. archive_command arşivleme modundan çıkmadan değiştirilebilir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. `wal_level=minimal` olarak ayarlandığında `archive_mode` etkinleştirilemez." type="primary" %}

{% include callout.html content="**`archive_command (string)`**: Tamamlanmış bir WAL dosyası segmentini arşivlemek için çalıştırılacak kabuk komutudur. Değerdeki her `%p` arşivlenecek dosyanın yoluyla, `%f` ise yalnızca dosya adıyla değiştirilir. (Yol adı sunucunun çalışma dizinine, yani kümenin veri diziniyle ilişkilidir.) Komuta gerçek `%` karakteri eklemek için `%%` kullanın. Daha fazla bilgi için [Continuous Archiving and Point-in-Time Recovery (PITR)]('') bölümüne bakın.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu parametre server başlangıcında `archive_mode` etkinleştirilmediği sürece yok sayılır. `archive_command`, `archive_mode` etkinken boş bir string ise (varsayılan), WAL arşivleme geçici olarak devre dışı bırakılır, ancak sunucu yakında bir komutun sağlanacağı beklentisiyle WAL segment dosyalarını biriktirmeye devam eder. `archive_command` komutunun bir şey yapmayan ama true döndüren bir komuta ayarlanması, örneğin /bin/true (Windows'ta REM) arşivlemeyi devre dışı bırakır, ancak aynı zamanda archive recovery için gereken WAL dosyaları zincirini de kırar. Bu nedenle yalnızca olağandışı durumlarda kullanılmalıdır ." type="primary" %}

{% include callout.html content="**`archive_timeout (integer)`**: archive_command yalnızca tamamlanmış WAL segmentleri için çağrılır. Bu nedenle, sunucunuz çok az WAL trafiği oluşturuyorsa bir transaction tamamlanması ile arşiv depolamasına güvenli kaydı arasında gecikme olabilir. Arşivlenmemiş verilerin ne kadar eski olabileceğini sınırlamak için `archive_timeout` ayarını sunucuyu periyodik olarak yeni bir WAL segment dosyasına geçmeye zorlayacak şekilde ayarlayabilirsiniz. Bu parametre 0 büyük olduğunda sunucu son segment dosyası değişiminden bu değer kadar süre geçtiğinde ve tek bir checkpoint gerçekleşmesi dahil herhangi bir veritabanı etkinliği olduğunda yeni segment dosyasına geçecektir (Veritabanı etkinliği yoksa checkpoint atlanır). Zorunlu geçiş nedeniyle erken kapatılan arşivlenmiş dosyaların hala tamamen dolu dosyalarla aynı uzunlukta olduğunu unutmayın. Bu nedenle, çok kısa `archive_timeout` kullanmak akıllıca değildir. Bu arşiv depolama alanınızı şişirecektir. 1 dakikalık `archive_timeout` ayarları genellikle makuldur. Verilerin ana sunucudan daha hızlı kopyalanması isteniyorsa arşivleme yerine streaming replication kullanılabilir. Bu değer birimsiz belirtilirse saniye olarak alınır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Archive Recovery

Bu kısımda yalnızca recovery süresi için uygulanan ayarlar açıklanmaktadır. Gerçekleştirmek istediğiniz sonraki herhangi bir recovery için sıfırlanmaları gerekir.

'Recovery', sunucunun standby olarak kullanılmasını veya hedeflenen bir kurtarmanın ( targeted recovery ) gerçekleştirilmesini kapsar. Standby modu yüksek kullanılabilirlik ve okuma ölçeklenebilirliğini sağlamak için kullanılırken, targeted recovery veri kaybından önlemek için kullanılır.

Sunucuyu standby modunda başlatmak için veri dizininde `standby.signal` adlı bir dosya oluşturun. Sunucu kurtarma işlemine girecek ve arşivlenen WAL'nin sonuna ulaşıldığında kurtarmayı durdurmayacaktır, ancak `primary_conninfo` ayarında belirtildiği gibi gönderen sunucuya bağlanarak ve / veya `restore_command` kullanarak yeni WAL segmentlerini alıp kurtarmaya devam edecektir. Bu mod ile ilgili parametler bu ve üstteki başlıklarda ele alınmıştır

Sunucuyu targeted recovery modunda başlatmak için veri dizininde `recovery.signal` adlı bir dosya oluşturun. Hem *standby.signal* hem de *recovery.signal* dosyaları oluşturulursa standby modu önceliklidir. targeted recovery modu, arşivlenen WAL tamamen replay edildiğinde veya `recovery_target`'e ulaşıldığında sona erer. Bu mod ile ilgili parametler bu ve üstteki başlıklarda ele alınmıştır

{% include callout.html content="**`restore_command (string)`**: WAL dosya serisinin arşivlenmiş bir bölümünü almak için yürütülecek kabuk komutudur. Bu parametre arşiv kurtarma için gereklidir, ancak streaming replication için isteğe bağlıdır. Verilen string'teki herbir `%f` arşivden alınacak dosyanın adıyla, `%p` ise sunucudaki kopya hedef path adı ile değiştirilir. Path adı geçerli çalışma diziniyle, yani kümenin veri diziniyle ilişkilidir. Herbir `%r` geçerli son yeniden başlatma noktasını içeren dosyanın adıyla değiştirilir. Bu, bir geri yüklemenin yeniden başlatılabilir olmasına izin vermek için saklanması gereken en eski dosyadır, bu nedenle bu bilgiler arşivi yalnızca geçerli geri yüklemeden yeniden başlatmayı sağlarken gereken minimum düzeye indirmek için kullanılabilir. `%r` genellikle yalnızca warm-standby yapılandırmaları tarafından kullanılır (bkz. [](https://www.postgresql.org/docs/current/warm-standby.html)). Gerçek bir `%` karakteri kullanmak için `%%` kullanın.

Komut yalnızca başarılı olduğunda 0 exit status döndermesi önemlidir. Komut, arşivde bulunmayan dosyaları istediğinde 0 farklı bir değer döndürmelidir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

Örnekler:

```bash
restore_command = 'cp /mnt/server/archivedir/%f "%p"'
restore_command = 'copy "C:\\server\\archivedir\\%f" "%p"'  # Windows

```

{% include important.html content="komutun veritabanı sunucusu kapatmanın bir parçası olarak kullanılan SIGTERM dışında bir sinyalle veya kabuktan kaynaklanan bir hatayla (komut bulunamadı gibi) sonlandırılması durumunda recovery işlemi durdurulur ve sunucu başlamaz."%}

{% include callout.html content="**`archive_cleanup_command (string)`**: İsteğe bağlı olan bu parametre, her restartpoint'de yürütülecek kabuk komutunu belirtir. `archive_cleanup_command`'ın amacı artık standby sunucu tarafından ihtiyaç duyulmayan eski arşivlenmiş WAL dosyalarını temizlemek için bir mekanizma sağlamaktır. Herbir `%r` son geçerli yeniden başlatma noktasını içeren dosyanın adıyla değiştirilir. Bu, bir restore'un yeniden başlatılabilir olmasını sağlamak için saklanması gereken en eski dosyadır. Bu nedenle `%r`'den önceki tüm dosyalar güvenli bir şekilde kaldırılabilir. Bu bilgiler, arşivi mevcut geri yüklemeden yeniden başlatmayı sağlmasında gereken minimum düzeye indirmek için kullanılır. [pg_archivecleanup](https://www.postgresql.org/docs/current/pgarchivecleanup.html) modülü genellikle `archive_cleanup_command`'da single-standby konfigürasyonlar için kullanılır, örneğin:" type="primary" %}

```bash
archive_cleanup_command = 'pg_archivecleanup /mnt/server/archivedir %r'
```

{% include callout.html content=" Aynı arşiv dizininden birden fazla strandby sunucu restore ediliyorsa, sunuculardan herhangi birinin ihtiyaç duymadığı WAL dosyalarının silinmediğinden emin olun. `archive_cleanup_command` komutu genellikle bir warm-standby konfigürasyonunda kullanılır (bkz. [](https://www.postgresql.org/docs/current/warm-standby.html)). Komuta gerçek bir `%` karakteri eklemek için `%%` kullanılır.<br/><br/>

Komut 0'dan farklı bir exit status döndürüldüğünde log dosyasına bir uyarı mesajı yazılır. Bunun bir istisnası, komutun bir sinyal veya kabuk tarafından bir hatayla sonlandırılması durumunda (komut bulunamadı gibi) fatal error gerçekleşir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. " type="info" %}

{% include callout.html content="**`recovery_end_command (string)`**: Bu parametre, recovery işleminin sonunda yalnızca bir kez yürütülecek bir kabuk komutunu belirtir. Bu parametre isteğe bağlıdır. `recovery_end_command` komutunun amacı, replication ve recovery sonrasında temizleme için bir mekanizma sağlamaktır. Herbir `%r`, `archive_cleanup_command`'da olduğu gibi geçerli enson yeniden başlatma noktasını içeren dosyanın adıyla değiştirilir.<br/><br/>

Komut 0'dan farklı bir exit status döndürüldüğünde log dosyasına bir uyarı mesajı yazılır ve veritabanı yine de başlamaya devam eder. Bunun bir istisnası, komutun bir sinyal veya kabuk tarafından bir hatayla sonlandırılması durumunda (komut bulunamadı gibi) veritabanı başlatmaya devam etmeyecektir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Recovery Target

Varsayılan olarak recovery WAL log'larının sonuna kadar devam edecektir. Daha öncesinde bir durma noktasını belirtmek için aşağıdaki parametreler kullanılabilir. `recovery_target`, `recovery_target_lsn`, `recovery_target_name`, `recovery_target_time` ve `recovery_target_xid`'den en fazla biri kullanılabilir. Yapılandırma dosyasında bunların birden fazlası belirtilirse hata ortaya çıkar. Bu parametreler yalnızca sunucu başlangıcında ayarlanabilir.

{% include callout.html content="**`recovery_target = 'immediate'`**: Bu parametre, recovery işleminin tutarlı bir duruma ulaşılır ulaşılmaz, yani mümkün olan en kısa sürede bitmesi gerektiğini belirtir. Bu, çevrimiçi bir yedeklemeden geri yükleme yaparken yedeklemenin bittiği nokta anlamına gelir.<br/><br/>

Teknik olarak bir string parametresi olmasına rağmen 'immediate' şu anda izin verilen tek değerdir." type="primary" %}

{% include callout.html content="**`recovery_target_name (string)`**: Bu parametre, recovery işleminin devam edeceği adlandırılmış geri yükleme noktasını (`pg_create_restore_point ()` ile oluşturulan) belirtir." type="primary" %}

{% include callout.html content="**`recovery_target_time (timestamp)`**: Bu parametre, recovery işleminin devam edeceği zaman damgasını belirtir. Kesin durma noktası [recovery_target_inclusive](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-RECOVERY-TARGET-INCLUSIVE)'e de bağlıdır.<br/><br/>

Bu parametrenin değeri, `timestamp with time zone` veri tiple ile aynı formatta bir zaman damgasıdır, tek farkı bir saat dilimi kısaltması kullanamamanızdır (`timezone_abbreviations` değişkeni yapılandırma dosyasında ayarlanmadıkça). Tercih edilen stil UTC'den sayısal bir uzaklık kullanmaktır veya tam bir saat dilimi adı yazalabilir, örneğin, `Europe/Helsinki`." type="primary" %}

{% include callout.html content="**`recovery_target_xid (string)`**: Bu parametre, recovery işleminin devam edeceği transaction ID'sini belirtir. Transaction ID'leri transaction başlangıcında sıralı olarak atanmasına rağmen transaction'ların farklı sayısal sırada tamamlanabileceğini unutmayın. Geri kazanılacak transaction'lar, belirtilen transaction'dan önce yapılan commmit'dir. Kesin durma noktası [recovery_target_inclusive](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-RECOVERY-TARGET-INCLUSIVE)'e de bağlıdır." type="primary" %}

{% include callout.html content="**`recovery_target_lsn (pg_lsn)`**: Bu parametre, recovery işleminin devam edeceği write-ahead log konumunun LSN'sini belirtir. Kesin durma noktası ayrıca recovery_target_inclusive'de bağlıdır. Bu parametre, [`pg_lsn`](https://www.postgresql.org/docs/current/datatype-pg-lsn.html) sistem veri tipi kullanılarak parse edilir." type="primary" %}

Aşağıdaki parametreler, recovery target'i daha ayrıntılı olarak belirtir ve hedefe ulaşıldığında ne olacağını etkiler:

{% include callout.html content="**`recovery_target_timeline (string)`**: Belirli bir zaman çizelgesindeki recovery işlemini belirtir. Değer, numeric bir timeline ID veya özel bir değer olabilir. `current` değeri, base backup alındığında geçerli olan aynı zaman çizelgesi boyunca kurtarılır. `latest` değeri, arşivde bulunan en son zaman çizelgesine geri döner ve bu bir standby sunucusunda yararlıdır.`latest` değeri varsayılandır.<br/><br/>

Bu parametreyi genellikle point-in-time recovery işleminden sonra elde edilen bir duruma geri dönmeniz gerektiği karmaşık re-recovery işlemlerinde ayarlamanız gerekir. Ayrıntı için bkz [](https://www.postgresql.org/docs/current/continuous-archiving.html#BACKUP-TIMELINES) ." type="primary" %}

{% include links.html %}
