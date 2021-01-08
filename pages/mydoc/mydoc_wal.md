---
title: "Write Ahead Log"
tags: [PostgreSQL]
keywords: postgres, wal_level, fsync, synchronous_commit, wal_sync_method, full_page_writes, wal_log_hints, wal_compression,wal_init_zero,wal_recycle, wal_buffers, wal_writer_delay, wal_writer_flush_after, wal_skip_threshold, commit_delay, commit_siblings, checkpoint_timeout, checkpoint_completion_target, checkpoint_flush_after, checkpoint_warning, checkpoint_warning, archive_mode,archive_command, archive_timeout, restore_command, archive_cleanup_command, archive_cleanup_command, recovery_target_name, recovery_target, recovery_target_time, recovery_target_xid, recovery_target_timeline, recovery_target_timeline
last_updated: January 7, 2021
sidebar: mydoc_sidebar
permalink: mydoc_wal.html
folder: mydoc
---

## Write Ahead Log

### Parametreler

{% include callout.html content="**`wal_level (enum)`**: Bu parametre WAL'a ne kadar bilgi yazılacağını belirler. Varsayılan değer `replica`'dır.<br/><br/>

- `replica` değeri bir standby sunucu üzerinde çalışan read-only sorgular dahil, WAL archiving ve replikasyon işlemleri için yeterli veriyi sağlar.<br/><br/>

- `minimal`, bir çökme veya ani kapanma durumunda recover işlemleri için gereken bilgileri garantiye alır. minimal değeri toplu veri operaslarıyla ilgili işlemleri (CREATE TABLE AS SELECT, CREATE INDEX) WAL günlüğüne kaydetmeyerek depolama avantajı sağlar.<br/><br/>

- `logical`, logical decoding ve logical replication için gerekli bilgileri sağlar. logical seviyesi, replica seviyesinde kaydedilen bilgiler ek, WAL'dan mantıksal değişikliklerin ayıklanmasına olanak sağlamak için gerekli bilgileri günlüğe kaydeder. logical seviyesini kullanmak, özellikle birçok tablo `REPLICA IDENTITY FULL` olarak yapılandırılmışsa ve fazla `UPDATE` ve `DELETE` ifadesi yürütülüyorsa WAL hacmini artıracaktır.<br/><br/>

Her düzey, alt düzeylerde kaydedilen tüm bilgileri içerir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" minimal WAL, verileri base backup ve WAL günlüklerinden yeniden yapılandırmak için yeterli bilgi içermez, bu nedenle WAL archiving ve streaming replication işlemleri için `replica` veya üstü ayar kullanılmalıdır." type="warning" %}

{% include callout.html content="**`fsync (boolean)`**: Güncellemelerin diske senkronizasyonunu zorlar. PostgreSQL sunucusu bu parametre açıksa, `fsync ()` sistem çağrıları veya eşdeğer yöntemler ile değişikliklerin fiziksel olarak diske yazıldığından emin olmaya çalışır. Bu, veritabanı kümesinin bir işletim sistemi veya donanım çökmesinden sonra tutarlı bir duruma geri yüklenebilmesini sağlar.<br/><br/>

`fsync`'i kapatmak genellikle bir performans avantajı olsamasına rağmen bir elektrik kesintisi veya sistem çökmesi durumunda kurtarılamaz veri bozulmasına neden olabilir.<br/><br/>

fsync, yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. Bu parametreyi kapatdığınızda `full_page_writes`'i de kapatmak düşünülebilir." type="primary" %}

{% include callout.html content="**`synchronous_commit (enum)`**: Mevcut transaction'ın senkronizasyon seviyesini ayarlar. Veritabanı sunucusu istemciye bir 'success' işareti döndürmeden önce ne kadar WAL işlemenin tamamlanması gerektiğini belirtir. Geçerli değerleri `remote_apply`, `on` (varsayılan), `remote_write`, `local` ve `off`." type="primary" %}

`synchronous_commit` ayarlarının yetenekleri:

| synchronous_commit setting | local durable commit | standby durable commit after PG crash | standby durable commit after OS crash | standby query consistency |
|-------|--------|-------|--------|-------|
| remote_apply | + | + | + | + |
| on | + | + | + | - |
| remote_write | + | + | - | - |
| local | + | - | - | - |
| off | - | - | - | - |

{% include callout.html content="**`wal_sync_method (enum)`**: WAL değişikliklerini diske göndermeye zorlamak için kullanılan yöntemi belirtir. `fsync` kapalıysa bu ayar geçersizdir. Olası değerler şunlardır: `open_datasync`, `fdatasync`, `fsync`, `fsync_writethrough`, `open_sync`. Verilen seçenekler tüm platformlarda mevcut değildir. `fdatasync` Linux'ta varsayılan değerdir. Bu parametre yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. " type="primary" %}

{% include callout.html content="**`full_page_writes (boolean)`**: PostgreSQL sunucusu bu parametre açık olduğunda herbir disk page'inin tüm içeriğini, ilgili page'in checkpoint'den sonraki ilk değişikliğnde WAL'a yazar. Tüm page'in saklanması, page'in doğru bir şekilde geri yüklenmesini garanti ederek, ancak WAL'a yazılması gereken veri miktarını artırır. (WAL replay her zaman bir checkpoint'den başlar, bunu bir checkpoint'den sonra her sayfanın ilk değişikliği sırasında yapması yeterlidir. Bu nedenle, Tüm page'i yazma maliyetini azaltmanın bir yolu, checkpoint aralığı parametrelerini artırmaktır.)<br/><br/>

Bu parametrenin kapatılması çalışmayı hızlandırır ancak bir sistem arızasından sonra kurtarılamayan veri bozulmalarına neden olabilir.<br/><br/>

Bu parametrenin kapatılması, point-in-time recovery (PITR) için WAL arşivleme kullanımını etkilemez.<br/><br/>

Bu parametre yalnızca postgresql.conf dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan açıktır. " type="primary" %}

{% include callout.html content="**`wal_log_hints (boolean)`**: Bu parametre `on` değerinde PostgreSQL sunucusu, hint bits olarak bilinen kritik olmayan değişiklikler için bile bir checkpoint'den sonra ilgili page'in ilk değişikliğinde her disk page'inin tüm içeriğini WAL'a yazar. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Varsayılan değeri `off`'dur." type="primary" %}

{% include callout.html content=" **`wal_compression (boolean)`**: Bu parametre `on` değerinde PostgreSQL sunucusu WAL'a yazılan bir full page görüntüyü sıkıştırır. WAL replay sırasında sıkıştırılmış bir page görüntüsü açılacaktır. Varsayılan değer `off`'dur. Bu ayarı yalnızca süper kullanıcılar değiştirebilir. <br/><br/>

Bu parametrenin açılması kurtarılamaz veri bozulması riskini artırmaz, WAL boyutunu azaltır. Fakat, sıkıştırma ve WAL replay sırasında sıkıştırmanın açılmasından kaynaklı fazladan CPU harcanmasına neden olur." type="primary" %}

{% include callout.html content="**`wal_init_zero (boolean)`**: `on` ayarında (varsayılan), yeni WAL dosyaları ilk kullanımdan önce sıfır ile doldurulur. Böylece, bazı dosya sistemlerinde WAL kayıtları yazılmadan önce alanın tahsis edilmesi sağlanır." type="primary" %}

{% include callout.html content="**`wal_recycle (boolean)`**: `on` ayarında (varsayılan), WAL dosyalarını yeniden adlandırarak geri kullanımını sağlar. Bu, yeni dosya oluşturma yükünden kurtarır. COW dosya sistemlerinde yenilerini oluşturmak daha hızlı olabildiğinden bu davranışı devre dışı bırakma seçeneği verilmiştir." type="primary" %}

{% include callout.html content="**`wal_buffers (integer)`**: Henüz diske yazılmamış WAL verileri için kullanılan shared memory miktarıdır. Öntanımlı -1 ayarı, shared_buffers'ın 1 / 32'ine eşit boyutu kullanır. Otomatik seçim çok büyük veya küçükse bu değer elle ayarlanabilir. 32kB'den küçük herhangi bir pozitif değer 32kB olarak değerlendirilecektir. Bu değer birim olmadan belirtdiğinde WAL blokları olarak alınır (XLOG_BLCKSZ bayt -- 8kB). Öntanımlı -1 ayarı ile seçilen otomatik ayarlama çoğu durumda makul sonuçlar verir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_writer_delay (integer)`**: WAL writer'ın WAL'ı zaman cinsinden ne sıklıkla temizleyeceğini (flush) belirtir. WAL temizledikten sonra asenkron commit edilen bir transaction ile daha erken uyanmadıkça, WAL writer `wal_writer_delay` süresince uyur. Son temizleme, `wal_writer_delay` öncesinde gerçekleştiyse ve bu zamandan beri `wal_writer_flush_after` değerinden daha az WAL üretildiyse, WAL kayıtları yalnızca işletim sistemine yazılır, diske temizlenmez. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan değer 200 milisaniyedir (200 ms). `wal_writer_delay` parametresini 10'un katı olmayan bir değere ayarlamak 10'un bir sonraki katına ayarlamakla aynı sonuçları verebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_writer_flush_after (integer)`**: WAL writer'ın volume cinsinden WAL'ı ne sıklıkla temizlediğini belirtir. Son temizleme, `wal_writer_delay` öncesinden gerçekleştiyse ve o zamandan beri `wal_writer_flush_after` değerinden daha az WAL üretildiyse, WAL yalnızca işletim sistemine yazılır, diske temizlenmez. `wal_writer_flush_after` 0 olarak ayarlanmışsa, WAL verileri anında temizlenir. Bu değer birim olmadan belirtilirse WAL blokları olarak alınır. (XLOG_BLCKSZ bayt -- 8kB). Öntanımlı değeri 1MB'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_skip_threshold (integer)`**: `wal_level = minimal` olduğunda ve bir transaction kalıcı bir ilişki oluşturduktan yada yeniden yazıldıktan sonra commit edildiğinde, bu ayar yeni verilerin nasıl kalıcı hale getirileceğini belirler. Veriler bu ayardan küçükse WAL log'larına yazınlır değilse fsync özelliği kullanılır. Depolamanızın özelliklerine bağlı olarak bu tür commitler eşzamanlı transaction'ları yavaşlatıyorsa bu değeri değiştirmek faydalı olabilir. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Varsayılan, iki megabayttır (2MB)." type="primary" %}

{% include callout.html content="**`commit_delay (integer)`**: Bu parametrenin ayarlanması bir WAL temizliği başlatılmadan önce gecikme süresi ekler. Daha fazla sayıda transaction'ın tek bir WAL temizleme yoluyla commit edilmesini sağlayarak grup commit verimini artırır. fsync devre dışı bırakılırsa gecikme yapılmaz. Bu değer birimsiz belirtilirse, mikrosaniye olarak alınır. commit_delay öntanımlı değeri 0'dır (gecikme yok). Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`commit_siblings (integer)`**: commit_delay'in gecikmeyi gerçekleştirmesi için gerekli minimum eşzamanlı açık transaction sayısını belirtir. Öntanımlı değeri 5 transaction'dır." type="primary" %}

### Checkpoints

{% include callout.html content="**`checkpoint_timeout (integer)`**: Otomatik WAL checkpoint'leri arasında maksimum süreyi belirtir. Bu değer birimsiz belirtilirse, saniye olarak alınır. Geçerli aralık 30 saniye ile 1 gün arasındadır. Varsayılan değeri 5 dakikadır (5min). Bu parametrenin artırılması, crash recovery için gerekli süreyi artırabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_completion_target (floating point)`**: checkpoint tamamlama hedefini belirtir. Öntanımlı 0,5'tir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_flush_after (integer)`**: Bir checkpoint gerçekleştirilirken bu miktardan daha fazla veri yazıldığında, bunları işletim sistemi depolamasına yazmaya zorlar. Bazı platformlarda bu ayarın hiçbir etkisi olmayabilir. Bu değer birimsiz belirtilirse bloklar olarak alınır (BLCKSZ bayt -- 8kB). Geçerli aralık zorunlu geri yazmayı devre dışı bırakan 0 ile 2MB arasıdır. Öntanımlı değeri Linux'ta 256kB, diğer sistemlerde 0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`checkpoint_warning (integer)`**: checkpoint segmentleri bundan daha sık doldurulursa uyarıları etkinleştirir. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı 30 saniyedir (30s). 0 uyarıyı devre dışı bırakır. `checkpoint_timeout` değeri `checkpoint_warning`'den azsa hiçbir uyarı üretilmez. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`min_wal_size (integer)`**: WAL disk kullanımı bu ayarın altında kaldığı sürece, eski WAL dosyalarını silmek yerine checkpoint'de tekrar kullanmak için geri dönüştürür. Bu değer birimsiz belirtilirse megabayt olarak alınır. Öntanımlı 80 MB'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Archiving

{% include callout.html content="**`archive_mode (enum)`**: Bu parametre etkinleştirildiğinde, tamamlanan WAL segmentleri `archive_command` ile arşiv depolamaya gönderilir. `off`, `on` ve `always` modları vardır. Normal çalışmada `on` ile `always` modu arasında fark yoktur. `always` ayarında archive recovery ve standby modunda da WAL arşivleyici etkinleştirilir. `always` modunda, arşivden geri yüklenen veya streaming replication ile akışa alınan tüm dosyalar arşivlenir." type="primary" %}

{% include callout.html content="**`archive_command (string)`**: Tamamlanmış bir WAL dosyası segmentini arşivlemek için çalıştırılacak kabuk komutudur. Değerdeki her `%p` arşivlenecek dosyanın path'i, her `%f` ise yalnızca dosya adıyla değiştirilir. path kümenin veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`archive_timeout (integer)`**: archive_command yalnızca tamamlanmış WAL segmentleri için çağrılır. Sunucunuz çok az WAL trafiği oluşturuyorsa bir transaction tamamlanması ile arşiv depolamasına gönderilmesi arasında gecikme olabilir. `archive_timeout` parametresi sunucuyu periyodik olarak yeni bir WAL segment dosyasına geçmeye zorlayacak şekilde ayarlanabilir. Bununla arşivlenmemiş verilerin ne kadar eski olabileceği sınırlandırılır. Zorunlu geçiş nedeniyle erken kapatılan arşivlenmiş dosyaların hala tamamen dolu dosyalarla aynı uzunlukta olduğuna dikkat edin. Bu nedenle, çok kısa `archive_timeout` kullanmak pek önerilmez. Bu arşiv depolama alanınızı şişirecektir. 1 dakikalık `archive_timeout` ayarları genellikle makuldur. Verilerin primary sunucudan daha hızlı kopyalanması isteniyorsa arşivleme yerine streaming replication kullanılabilir. Bu değer birimsiz belirtilirse saniye olarak alınır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Archive Recovery

{% include callout.html content="**`restore_command (string)`**: WAL dosya serisinin arşivlenmiş bir bölümünü almak için yürütülecek kabuk komutudur. Bu parametre arşiv kurtarma için gereklidir, streaming replication için isteğe bağlıdır. Verilen string'teki herbir `%f` arşivden alınacak dosyanın adıyla, `%p` ise sunucudaki kopya hedef path adı ile değiştirilir. Path adı kümenin veri diziniyle ilişkilidir. Herbir `%r` geçerli son yeniden başlatma noktasını içeren dosyanın adıyla değiştirilir. `%r` genellikle warm-standby yapılandırmalarında kullanılır bkz. [](https://www.postgresql.org/docs/current/warm-standby.html)

Komut başarılı olduğunda 0 exit status, arşivde bulunmayan dosyaları istediğinde 0 farklı bir değer dönderir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

Örnekler:

```bash
restore_command = 'cp /mnt/server/archivedir/%f "%p"'
restore_command = 'copy "C:\\server\\archivedir\\%f" "%p"'  # Windows

```

{% include important.html content="komutun veritabanı sunucusu kapatmanın bir parçası olarak kullanılan SIGTERM dışında bir sinyalle veya kabuktan kaynaklanan bir hatayla (komut bulunamadı gibi) sonlandırılması durumunda recovery işlemi durdurulur ve sunucu başlatılmaz."%}

{% include callout.html content="**`archive_cleanup_command (string)`**: İsteğe bağlı olan bu parametre, her restartpoint'de yürütülecek kabuk komutunu belirtir. `archive_cleanup_command`'ın amacı artık standby sunucu tarafından ihtiyaç duyulmayan eski arşivlenmiş WAL dosyalarını temizlemek için bir mekanizma sağlamaktır. Herbir `%r` son geçerli restartpoint'i içeren dosyanın adıyla değiştirilir. Bu bilgiler, arşivi mevcut geri yüklemeden yeniden başlatmayı sağlmasında gereken minimum düzeye indirmek için kullanılır. [pg_archivecleanup](https://www.postgresql.org/docs/current/pgarchivecleanup.html) modülü genellikle `archive_cleanup_command`'da single-standby konfigürasyonlar için kullanılır, örneğin:" type="primary" %}

```bash
archive_cleanup_command = 'pg_archivecleanup /mnt/server/archivedir %r'
```

{% include callout.html content=" Aynı arşiv dizininden birden fazla strandby sunucu restore ediliyorsa, sunuculardan herhangi birinin ihtiyaç duymadığı WAL dosyalarının silinmediğinden emin olun. `archive_cleanup_command` komutu genellikle warm-standby konfigürasyonunda kullanılır bkz. [](https://www.postgresql.org/docs/current/warm-standby.html).<br/><br/>

Komut 0'dan farklı bir exit status döndürüldüğünde log dosyasına bir uyarı mesajı yazılır. Komutun bir sinyal veya kabuk tarafından bir hatayla sonlandırılması durumunda (komut bulunamadı gibi) fatal error gerçekleşir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. " type="info" %}

{% include callout.html content="**`recovery_end_command (string)`**: Bu parametre, recovery işleminin sonunda yalnızca bir kez yürütülecek bir kabuk komutunu belirtir. İsteğe bağlı bir parametredir. Amacı, replication ve recovery sonrasında temizleme için bir mekanizma sağlamaktır. Herbir `%r`, `archive_cleanup_command`'da olduğu gibi geçerli enson yeniden başlatma noktasını içeren dosyanın adıyla değiştirilir.<br/><br/>

Komut 0'dan farklı bir exit status dönderdiğinde log dosyasına bir uyarı mesajı yazılır ve veritabanı yine de başlatılır. Komutun bir sinyal veya kabuk tarafından bir hatayla sonlandırılması durumunda (komut bulunamadı gibi) veritabanı başlatmaya devam etmez.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Recovery Target

Recovery varsayılan olarak WAL log'larının sonuna kadar devam edecektir. Daha öncesinde bir durma noktasını belirtmek için aşağıdaki parametreler kullanılabilir. `recovery_target`, `recovery_target_lsn`, `recovery_target_name`, `recovery_target_time` ve `recovery_target_xid`'den en fazla biri kullanılabilir. Yapılandırma dosyasında bunların birden fazlası belirtilirse hata verir. Bu parametreler yalnızca sunucu başlangıcında ayarlanabilir.

{% include callout.html content="**`recovery_target = 'immediate'`**: Bu parametre, recovery işleminin tutarlı bir duruma ulaşılır ulaşılmaz, yani mümkün olan en kısa sürede bitmesi gerektiğini belirtir. 'immediate' şu anda izin verilen tek değerdir." type="primary" %}

{% include callout.html content="**`recovery_target_name (string)`**: Bu parametre, recovery işleminin devam edeceği adlandırılmış geri yükleme noktasını (`pg_create_restore_point ()` ile oluşturulan) belirtir." type="primary" %}

{% include callout.html content="**`recovery_target_time (timestamp)`**: Bu parametre, recovery işleminin devam edeceği zaman damgasını belirtir. Parametrenin değeri, `timestamp with time zone` veri tiple ile aynı formatta bir zaman damgasıdır, tek farkı bir saat dilimi kısaltması kullanılamaz (`timezone_abbreviations` değişkeni yapılandırma dosyasında ayarlanmadıkça). Tercih edilen stil UTC'den sayısal bir uzaklık kullanmaktır veya tam bir saat dilimi adı yazalabilir, örneğin, `Europe/Istanbul`." type="primary" %}

{% include callout.html content="**`recovery_target_xid (string)`**: Bu parametre, recovery işleminin devam edeceği transaction ID'sini belirtir. Transaction ID'leri transaction başlangıcında sıralı olarak atanmasına rağmen transaction'ların farklı sayısal sırada tamamlanabileceğine dikkat edin. Kurtarılacak transaction'lar, belirtilen transaction'dan önce yapılan commmit'dir." type="primary" %}

{% include callout.html content="**`recovery_target_lsn (pg_lsn)`**: Bu parametre, recovery işleminin devam edeceği write-ahead log konumunun LSN'sini belirtir. Kesin durma noktası ayrıca recovery_target_inclusive'de bağlıdır. Bu parametre, [`pg_lsn`](https://www.postgresql.org/docs/current/datatype-pg-lsn.html) sistem veri tipi kullanılarak parse edilir." type="primary" %}

Bu bölümdeki ayarların yapılmasıyla ilgili ek bilgi için [WAL Yapılandırması](https://www.postgresql.org/docs/current/wal-configuration.html) bölümüne bakın.

{% include links.html %}
