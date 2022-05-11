---
Ftitle: "Replikasyon"
layout: default
parent: Veritabanı Yapılandırması
nav_order: 6
---

## Replikasyon

Bu bölümde verilen ayarlar dahili [streaming replikasyon](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION) özelliğinin davranışını kontrol eder. Sunucular, primary ya da standby bir sunucu olabilir. Primary sunucular veri gönderebilir, standby ise replike verilerin alıcılarıdır. Standby sunucular [Cascading replikasyon](https://www.postgresql.org/docs/current/warm-standby.html#CASCADING-REPLICATION) kullanıldığında alıcının (receiver) yanı sıra gönderici (senders) de olabilir.

### Sending Sunucular

Bu başlıkta verilen parametreler, replike verileri bir veya daha fazla standby sunucuya gönderen sunucuda ayarlanır. Primary, her durumda gönderen bir sunucu olduğundan dolayı bu parametreler her zaman primary üzerinde ayarlanmalıdır. Bu parametrelerin rolü ve anlamı bir standby primary olduktan sonra değişmez.

#### `max_wal_senders`

{% include parameter_info.html parametre="max_wal_senders" %}

{% include callout.html content=" **Standby sunuculardan veya streaming tabanlı yedekleme istemcilerinden gelen maksimum senkron bağlantı sayısını (örneğin, aynı anda çalışan maksimum WAL sender süreci sayısı) belirtir.** Öntanımlı değeri 10'dur. 0 değeri replikasyonun devre dışı bırakıldığı anlamına gelir. Bir streaming istemcisinin bağlantısının aniden kesilmesi, zaman aşımına ulaşılıncaya kadar orphan bir bağlantı slotuna sebep olabilir. Bu sebepten dolayı parametre, bağlantısı kesilen istemcilerin yeniden bağlanabilmesi için beklenen maksimum istemci sayısından fazla ayarlanmalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Standby sunucu bağlantılarına izin vermek için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır.<br/><br/>

Bir standby sunucuda bu parametreyi primary sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde standby sunucusunda sorgulara izin verilmeyecektir." type="primary" %}

#### `max_replication_slots`

{% include parameter_info.html parametre="max_replication_slots" %}

{% include callout.html content=" **Sunucunun destekleyebileceği maksimum [replikasyon slotu](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) sayısını belirtir.** Öntanımlı değeri 10'dur. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Mevcut replikasyon slotlarının sayısından daha düşük bir değere ayarlamak, sunucunun başlamasını engeller. Ayrıca, replikasyon slotlarının kullanılmasına izin vermek için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır" type="primary" %}

#### `wal_keep_size`

{% include parameter_info.html parametre="wal_keep_size" %}

{% include callout.html content=" **`pg_wal` dizininde tutulan geçmiş log dosyası segmentlerinin minimum miktarını belirtir.** Standby sunucu bunları streaming replikasyon için kullanır. Standby, sending sunucunundan `wal_keep_size` megabayttan daha fazla geride kalırsa, sending sunucu standby'ın ihtiyacı olan bir WAL segmentini silebilir. Bu durumda replikasyon bağlantısı sonlandırılır. (WAL arşivleme aktifse, standby sunucu segmentleri arşivden alarak kurtarabilir.)<br/><br/>

Bu parametre yalnızca `pg_wal`'da tutulan minimum segment boyutunu ayarlar. Sistemin WAL arşivleme veya bir checkpoint'ten kurtarmak için daha fazla segment tutması gerekebilir. `wal_keep_size` 0 ise (varsayılan), sistem standby amaçları için fazladan segment tutmaz. Bu değer birimsiz belirtilirse megabayt olarak alınır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `max_slot_wal_keep_size`

{% include parameter_info.html parametre="max_slot_wal_keep_size" %}

{% include callout.html content=" **[Replikasyon slotlarının](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) checkpoint zamanında `pg_wal` dizininde tutmasına izin verilen maksimum WAL dosyası boyutunu belirtir.**`max_slot_wal_keep_size = -1` ayarında (varsayılan), replikasyon slotları sınırsız miktarda WAL dosyası tutabilir. Bir replikasyon slotunun `restart_lsn` değeri geçerli LSN'nin verilen boyuttan daha fazla gerisinde kalırsa slotu kullanan standby gerekli WAL dosyalarının silinmesinden dolayı replikasyona devam edemez. Replikasyon slotlarının WAL kullanabilirliğini [`pg_replication_slots`](https://www.postgresql.org/docs/current/view-pg-replication-slots.html) ile görebilirsiniz." type="primary" %}

#### `wal_sender_timeout`

{% include parameter_info.html parametre="wal_sender_timeout" %}

{% include callout.html content=" **Bu süreden daha uzun süre etkin olmayan replikasyon bağlantılarını sonlandırılır.** Gönderen sunucunun standby'da bir çökmeyi veya ağ kesintisini algılaması için kullanışlıdır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 60 saniyedir. 0 değeri, zaman aşımı mekanizmasını devre dışı bırakır.<br/><br/>

Birden çok coğrafi konuma dağıtılmış bir kümede, her bir konum için farklı değerler kullanmak küme yönetiminde esneklik sağlar. Daha küçük bir değer, düşük gecikmeli ağ bağlantısına sahip bir standby'da hızlı arıza tespiti için kullanışlıdır. Daha büyük bir değer, yüksek gecikmeli ağ bağlantısıyla uzak bir konumda bulunan standby'ın sağlığını daha iyi değerlendirmeye yardımcı olur." type="primary" %}

#### `track_commit_timestamp`

{% include parameter_info.html parametre="track_commit_timestamp" %}

{% include callout.html content=" **Transaction'ların commit süresini kaydeder.** Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri `off`." type="primary" %}

### Primary Sunucu

Bu başlıkta verilen parametreler, replikasyon verilerini bir veya daha fazla standby sunucuya gönderecek olan master/primary sunucuda ayarlanır. Bu parametrelere ek olarak, `wal_level`'ın primary sunucuda uygun şekilde ayarlanması gerekir. İsteğe bağlı olarak WAL arşivleme de etkinleştirilebilir bkz. [Archiving](mydoc_wal.html#archiving). Standby sunucularda bu parametrelerin değerleri önemsizdir, ancak bir standby'ın primary sunucu olma senaryosuna hazırlık için standby sunucuda da ayarlanabilir.

#### `synchronous_standby_names`

{% include parameter_info.html parametre="synchronous_standby_names" %}

{% include callout.html content=" **[Senkron replikasyon]((https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION)) destekleyen standby sunucu listesini belirtir.** Herhangi bir anda bir veya daha fazla aktif senkron standby olabilir. Bu standby sunucular verilerin alındığını onayladıktan sonra commit için bekleyen transaction'lara izin verir. Mevcut bağlı ve gerçek zamanlı streaming verisi olan senkron standby'lar listede ismi verilenlerden olacaktır. (`pg_stat_replication` view'ı streaming durumunu gösterir). Birden fazla senkron standby'in belirtilmesi, yüksek erişebilirliği (HA) ve veri kaybına karşı koruma sağlar. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `vacuum_defer_cleanup_age`

{% include parameter_info.html parametre="vacuum_defer_cleanup_age" %}

{% include callout.html content=" **Varsa, `VACUUM` ve HOT temizlemesinin ertelenmesi gereken transaction sayısı.** Varsayılan sıfır transaction'dır, dead row sürümleri mümkün olan en kısa sürede, yani açık transaction'lar tarafından visible olmaktan çıkar çıkmaz kaldırılır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Standby Sunucular

Bu başlıkta verilen ayarlar replikasyon verilerini alan standby sunucunun davranışını kontrol eder.

#### `primary_conninfo`

{% include parameter_info.html parametre="primary_conninfo" %}

{% include callout.html content=" **Standby sunucunun gönderen sunucuya bağlanması için kullanılan bağlantı dizesini belirtir.** Bu dize [burada](https://www.postgresql.org/docs/current/libpq-envars.html) açıklanan formattadır. Bu dizede herhangi bir seçenek belirtilmediğinde ilgili ortam değişkeni kontrol edilir. Ortam değişkeni de ayarlanmadıysa varsayılanlar kullanılır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `primary_slot_name`

{% include parameter_info.html parametre="primary_slot_name" %}

{% include callout.html content=" **Gönderen sunucuda kullanılacak replikasyon slotu adını ayarlar.** Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. WAL receiver süreci çalışırken bu parametre değiştirilirse, bu sürecin kapanması için sinyal gönderilir ve yeni ayarla tekrar başlatılması beklenir. `primary_conninfo` ayarlanmamışsa veya sunucu standby modunda değilse bu ayarın hiçbir etkisi yoktur." type="primary" %}

#### `promote_trigger_file`

{% include parameter_info.html parametre="promote_trigger_file" %}

{% include callout.html content=" **Standby modunda recovery'i sona erdiren bir trigger dosyası belirtir.** Bu değer ayarlandan standby `pg_ctl promote` ve `pg_promote ()` ile promote edilebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `hot_standby`

{% include parameter_info.html parametre="hot_standby" %}

{% include callout.html content=" **Recovery sırasında bağlantılara ve sorgulara izin verir.** [Hot Standby](https://www.postgresql.org/docs/current/hot-standby.html)'de açıklandığı gibi recovery sırasında bağlanıp sorgu çalıştırıp çalıştırmayacağınızı ayarlar. Öntanımlı değeri `on`'dur. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Archive recovery sırasında ve standby modunda etkilidir." type="primary" %}

#### `max_standby_archive_delay`

{% include parameter_info.html parametre="max_standby_archive_delay" %}

{% include callout.html content=" **Bu parametre Hot standby etkin olduğunda işlenen WAL verileriyle çakışan standby sorgularını iptal etmeden önce standby sunucunun maksimum gecikmesini ayarlar.** WAL verileri WAL arşivinden okunduğunda uygulanır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı 30 saniyedir. -1 değeri, standby'ın çakışan sorguların tamamlanması için sonsuza kadar beklemesine izin verir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include tip.html content=" Failover amaçlı replikasyon yapıyorsanız, standby'ı olabildiğince güncel tutmak için bunu çok düşük bir değere (0 gibi) ayarlayın."%}

#### `max_standby_streaming_delay`

{% include parameter_info.html parametre="max_standby_streaming_delay" %}

{% include callout.html content=" **Bu parametre Hot Standby etkin olduğunda işlenen WAL verileriyle çakışan standby sorgularını iptal etmeden önce standby sunucunun maksimum gecikmesini ayarlar.** `max_standby_streaming_delay`, WAL verileri streaming replikasyon yoluyla alındığında uygulanır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 30 saniyedir. -1 değeri, çakışan sorguların tamamlanması için standby'ın sonsuza kadar beklemesine izin verir. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include tip.html content=" Failover amaçlı replikasyon yapıyorsanız, standby'ı olabildiğince güncel tutmak için bunu çok düşük bir değere (0 gibi) ayarlayın."%}

#### `wal_receiver_create_temp_slot`

{% include parameter_info.html parametre="wal_receiver_create_temp_slot" %}

{% include callout.html content=" **Kalıcı bir slot yapılandırılmamışsa (`primary_slot_name` kullanarak) bir WAL alıcısının geçici bir replikasyon slotu oluşturup oluşturmayacağını belirler.** Varsayılan olarak kapalıdır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. WAL receiver süreci çalışırken bu parametre değiştirilirse, bu sürecin kapanması için sinyal gönderilerek yeni ayarla tekrar başlatılması beklenir." type="primary" %}

#### `wal_receiver_status_interval`

{% include parameter_info.html parametre="wal_receiver_status_interval" %}

{% include callout.html content=" **Primary ve upstream standby'a replikasyon ilerlemesi hakkında bilgi gönderen standby üzerindeki WAL receiver sürecinin durum raporları arasındaki maksimum aralığı ayarlar.** Bu bilgiler [pg_stat_replication](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-VIEW) view'ı kullanılarak görüntülenir. Standby, yazdığı son WAL kaydının konumunu, diske flush edilen yer ve uyguladığı son konumu raporlar. Parametrenin değeri bu raporlar arasındaki maksimum süredir. Değişiklikler, yazma veya flush konumları her değiştiğinde veya bu parametre ile belirtilen sıklıkta gönderilir. Bu nedenle, uygulama konumu gerçek konumun biraz gerisinde kalabilir. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı değeri 10 saniyedir. Bu parametrenin 0 olarak ayarlanması, durum değişikliklerini tamamen devre dışı bırakır. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `hot_standby_feedback`

{% include parameter_info.html parametre="hot_standby_feedback" %}

{% include callout.html content=" **Bir hot standby'ın yürütülen sorgular hakkında primary veya upstream standby'a geri bildirim gönderip göndermeyeceğini ayarlar.** Bu parametre, temizleme kayıtlarından kaynaklı sorgu iptallerini ortadan kaldırmak için kullanılır. Bazı iş yüklerinde primary veritabanı şişmesine neden olabilir. Geri bildirim mesajları, her bir `wal_receiver_status_interval`'da bir defadan fazla gönderilmez. Varsayılan değeri `off`'dur. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Cascaded replikasyon kullanımında, geri bildirim primary'e ulaşana kadar upstream'e iletilir. Standby'lar aldıkları geri bildirimi, upstream'e göndermek dışında hiçbir şekilde kullanmaz." type="primary" %}

#### `wal_receiver_timeout`

{% include parameter_info.html parametre="wal_receiver_timeout" %}

{% include callout.html content=" **Bu parametrede belirtilen süreden daha uzun süre etkin olmayan replikasyon bağlantılarını sonlandırır.** Alıcı standby sunucunun primary düğüm çökmesi veya ağ kesintisini algılaması için kullanışlıdır. Birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 60 saniyedir. 0 değeri, zaman aşımı mekanizmasını devre dışı bırakır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `wal_retrieve_retry_interval`

{% include parameter_info.html parametre="wal_retrieve_retry_interval" %}

{% include callout.html content=" **Standby sunucunun, herhangi bir kaynakta (streaming replikasyon, yerel 'pg_wal' veya WAL arşivi) WAL verileri hazır olmadığında beklenecek süreyi belirtir.** Bu değer birimsiz belirtilirse milisaniye olarak alınır. Ötanımlı değeri 5 saniyedir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Archive recovery'de, bu parametre değerini azaltarak yeni WAL log dosyası tespitinde recovery'i daha duyarlı hale getirmek mümkündür. Düşük WAL etkinliğine sahip bir sistemde bu değeri artırmak, WAL arşivine erişim istek miktarını azaltır. Bu, altyapıya erişme miktarının hesaba katıldığı bulut ortamlarında faydalıdır." type="primary" %}

#### `recovery_min_apply_delay`

{% include parameter_info.html parametre="recovery_min_apply_delay" %}

{% include callout.html content=" **Bir standby sunucu varsayılan olarak WAL kayıtlarını gönderen sunucudan mümkün olan en kısa sürede yeniler.** Verilerin gecikmeli bir kopyasına sahip olmak veri kaybı hatalarını düzeltmek için faydalı olabilir. Bu parametre recovery'i belirli bir süre geciktirmenizi sağlar. Örneğin, bu parametreyi `5min` olarak ayarlarsanız standby, her transaction commit'ini yalnızca standby'daki sistem süresi primary tarafından bildirilen commit süresinden en az beş dakika geçtiğinde replay eder. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan sıfırdır, gecikme yoktur.<br/><br/>

Sunucular arasındaki replikasyon gecikmesinin bu parametrenin değerini aşması durumunda herhangi bir gecikme eklenmez. Gecikmenin, primary'de yazılan WAL zaman damgası ile standby'daki saat arasında hesaplandığını unutmayın. Ağ gecikmesi veya cascading replikasyon yapılandırmaları sebepli aktarımdaki gecikmeler mevcut bekleme süresini önemli ölçüde azaltabilir. Primary ve standby'daki sistem saatleri senkronize değilse kayıtlar beklenenden daha erken uygulanabilir.<br/><br/>

Gecikme, yalnızca transaction commit'leri için WAL kayıtlarında meydana gelir. Diğer kayıtlar olabildiğince çabuk replay edilir. Bu bir problem değildir çünkü MVCC visibility kuralları, ilgili commit kaydı uygulanana kadar etkilerinin görünür olmamasını sağlar.<br/><br/>

Gecikme, recovery sırasında veritabanı tutarlı bir duruma ulaştığında, standby promote edilene veya tetiklenene kadar gerçekleşir ve sonrasında standby daha fazla beklemeden recovery'i sonlandırır.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasında veya sunucu komut satırında ayarlanabilir." type="primary" %}

### Subscribers

Bu başlıkta verilen ayarlar logical replikasyon subscriber davranışını kontrol eder. publisher değerleri konu dışıdır.

#### `max_logical_replication_workers`

{% include parameter_info.html parametre="max_logical_replication_workers" %}

{% include callout.html content=" **Maksimum logical replikasyon worker sayısını belirtir.** Bu, hem apply hem de tablo senkronizasyon worker'ları kapsar. Logical replikasyon worker, `max_worker_processes` tarafından tanımlanan havuzdan alınır. Öntanımlı değer 4'tür." type="primary" %}

#### `max_sync_workers_per_subscription`

{% include parameter_info.html parametre="max_sync_workers_per_subscription" %}

{% include callout.html content=" **Her bir subscription için maksimum senkronizasyon worker'ı sayısı belirtir.** Bu parametre, subscription başlatma sırasında veya yeni tablolar eklendiğinde ilk veri kopyasının parallelism miktarını kontrol eder. Şu an için her tablo başına yalnızca bir senkronizasyon worker'ı olabilir. Senkronizasyon worker'ları `max_logical_replication_workers` tarafından tanımlanan havuzdan alınır. Öntanımlı değeri 2'dir." type="primary" %}

{% include note.html content=" **wal_receiver_timeout**, **wal_receiver_status_interval** ve **wal_retrieve_retry_interval** yapılandırma parametreleri de logical replication worker'larını etkiler."%}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-replication.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
