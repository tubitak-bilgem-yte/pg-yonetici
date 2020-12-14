---
Ftitle: "Replikasyon"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 04, 2020
summary: "Replikasyon"
sidebar: mydoc_sidebar
permalink: mydoc_replication.html
folder: mydoc
---

## Replikasyon

Bu bölümde verilen ayarlar yerleşik [streaming replikasyon](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION) özelliğinin davranışını kontrol eder. Sunucular, primary ya da standby bir sunucu olabilir. Primary sunucular veri gönderebilir, standby ise her zaman replike verilerin alıcılarıdır. Standby sunucular [Cascading replikasyon](https://www.postgresql.org/docs/current/warm-standby.html#CASCADING-REPLICATION) kullanıldığında alıcının (receiver) yanı sıra gönderici (senders) de olabilir. Parametreler çoğunlukla sending ve standby sunucular için olmakla birlikte bazı parametreler yalnızca primary sunucuda anlamlıdır.

### Sending Servers

Bu başlıkta verilen parametreler, replike verileri bir veya daha fazla standby sunucuya gönderen sunucuda ayarlanır. Primary, her durumda gönderen bir sunucu olduğundan dolayı bu parametreler her zaman primary üzerinde ayarlanmalıdır. Bu parametrelerin rolü ve anlamı bir standby primary olduktan sonra değişmez.

{% include callout.html content="**`max_wal_senders (integer)`**: Standby sunuculardan veya streaming tabanlı yedekleme istemcilerinden gelen maksimum senkron bağlantı sayısını (örneğin, aynı anda çalışan maksimum WAL sender süreci sayısı) belirtir. Öntanımlı değeri 10'dur. 0 değeri replikasyonun devre dışı bırakıldığı anlamına gelir. Bir streaming istemcisinin bağlantısının aniden kesilmesi, zaman aşımına ulaşılıncaya kadar orphan bir bağlantı slotuna sebep olabilir. Bu nedenle bu parametre, bağlantısı kesilen istemcilerin yeniden bağlanabilmesi için beklenen maksimum istemci sayısından fazla ayarlanmalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Ayrıca, standby sunucu bağlantılarına izin vermek için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır.<br/><br/>

Bir standby sunucu çalıştırırken, bu parametreyi primary sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde standby sunucusunda sorgulara izin verilmeyecektir." type="primary" %}

{% include callout.html content="**`max_replication_slots (integer)`**: Sunucunun destekleyebileceği maksimum [replikasyon slotu](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) sayısını belirtir. Öntanımlı değeri 10'dur. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Mevcut replikasyon slotlarının sayısından daha düşük bir değere ayarlamak, sunucunun başlamasını engeller. Ayrıca, replikasyon slotlarının kullanılmasına izin vermek  için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır" type="primary" %}

{% include callout.html content="**`wal_keep_size (integer)`**: `pg_wal` dizininde tutulan geçmiş log dosyası segmentlerinin minimum miktarını belirtir. Standby sunucu bunları streaming replikasyon için kullanır. Standby, gönderen sunucunundan `wal_keep_size` megabayttan daha fazla geride kalırsa, gönderen sunucu standby'ın ihtiyacı olan bir WAL segmentini silebilir ve bu durumda replikasyon bağlantısı sonlandırılır. (WAL arşivleme kullanımdaysa, standby sunucu segmentleri arşivden alarak kurtarabilir.)<br/><br/>

Bu parametre yalnızca `pg_wal`'da tutulan minimum segment boyutunu ayarlar. Sistemin WAL arşivleme veya bir checkpoint'ten kurtarmak için daha fazla segment tutması gerekebilir. `wal_keep_size` 0 ise (varsayılan), sistem standby amaçları için fazladan segment tutmaz. Bu nedenle standby sunucuların kullanabileceği eski WAL segmentlerinin sayısı, önceki checkpoint konumu ve WAL arşivleme durumunun bir fonksiyonur. Bu değer birimsiz belirtilirse megabayt olarak alınır. Bu parametre yalnızca *postgresql.conf* dosyasından vey sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`max_slot_wal_keep_size (integer)`**: [Replikasyon slotlarının](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) checkpoint zamanında `pg_wal` dizininde tutmasına izin verilen maksimum WAL dosyası boyutunu belirtir. `max_slot_wal_keep_size = -1` ise (varsayılan), replikasyon slotları sınırsız miktarda WAL dosyası tutabilir. Diğer durumlarda, bir replikasyon slotunun `restart_lsn` değeri, geçerli LSN'nin verilen boyuttan daha fazla gerisinde kalırsa slotu kullanan standby gerekli WAL dosyalarının silinmesinden dolayı replikasyona devam edemez. Replikasyon slotlarının WAL kullanabilirliğini [`pg_replication_slots`](https://www.postgresql.org/docs/current/view-pg-replication-slots.html) ile görebilirsiniz." type="primary" %}

{% include callout.html content="**`wal_sender_timeout (integer)`**: Bu süreden daha uzun süre etkin olmayan replikasyon bağlantılarını sonlandırılır. Bu, gönderen sunucunun standby'da bir çökmeyi veya ağ kesintisini algılaması için kullanışlıdır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 60 saniyedir. 0 değeri, zaman aşımı mekanizmasını devre dışı bırakır.<br/><br/>

Birden çok coğrafi konuma dağıtılmış bir küme de, herbir konum için farklı değerler kullanmak küme yönetiminde daha fazla esneklik sağlar. Daha küçük bir değer, düşük gecikmeli ağ bağlantısına sahip bir standby ile daha hızlı arıza tespiti için kullanışlıdır. Daha büyük bir değer, yüksek gecikmeli ağ bağlantısıyla uzak bir konumda bulununan standby'ın sağlığını daha iyi değerlendirmeye yardımcı olur." type="primary" %}

{% include callout.html content="**`track_commit_timestamp (boolean)`**: Transaction'ların commit süresini kaydeder. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri `off`." type="primary" %}

### Primary Server

Bu başlıkta verilen parametreler, replikasyon verilerini bir veya daha fazla standby sunucuya gönderecek olan master/primary sunucuda ayarlanır. Bu parametrelere ek olarak, `wal_level`'ın primary sunucuda uygun şekilde ayarlanması gerekir. İsteğe bağlı olarak WAL arşivleme de etkinleştirilebilir (bkz. [Archiving](mydoc_wal.html#archiving)). Standby sunucularda bu parametrelerin değerleri önemsizdir, ancak bir yedeklemenin primary sunucu olma senaryosuna hazırlık olarak standby sunucuda da ayarlanabilir.

{% include callout.html content="**`synchronous_standby_names (string)`**: Bu parametre senkron standby sayısını ve potansiyel senkron standby'ların isim listesini belirtir. (Senkron replikasyon için bkz. [](https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION)) Herhangi bir an da en fazla bir veya daha fazla aktif senkron standby olabilir ve bu standby sunucusular verileri alındığını onayladıktan sonra commit için bekleyen transaction'lara izin verilir. Mevcut bağlı ve gerçek zamanlı streaming verisi olan senkron standby'lar listede ismi verilenlerden olacaktır. (`pg_stat_replication` view'ı streaming durumunu gösterir). Birden fazla senkron standby'in belirtilmesi, yüksek erişebilirliği (HA) ve veri kaybına karşı koruma sağlar.<br/><br/>

Bir standby sunucusunun ismi standby'ın `application_name` ayarıdır. Physical replication standby olması durumunda, bu `primary_conninfo` ayarında ayarlanmalıdır. varsayılan, ayarlanmışsa `cluster_name`'dir aksi takdirde `walreceiver`'dır. Bu, logical replikasyon için subscription bağlantı bilgilerinde ayarlanabilir ve varsayılan olarak subscription adı olur. Diğer replikasyon stream consumer'ları için ilgili belgelere bakın.<br/><br/>

Bu parametre için aşağıdaki sözdizimlerinden biri kullanılarak standby sunucularının bir listesini belirtilir:" type="primary" %}

```sql
[FIRST] num_sync ( standby_name [, ...] )
ANY num_sync ( standby_name [, ...] )
standby_name [, ...]
```

{% include callout.html content="**num_sync**, transaction'ların yanıtlarını beklemesi gereken senkron standby sayısıdır. **standby_name**, standby sunucusunun adıdır. `FIRST` ve `ANY` listelenen sunuculardan senkron standby'ları seçme yöntemini belirtir.<br/><br/>

`FIRST` anahtar kelimesi, **num_sync** ile verildiğinde, önceliğe dayalı senkron replikasyonu belirtir ve transaction commit'lerinı WAL kayıtları önceliklerine göre seçilen **num_sync** senkron standby'larda replike edilene kadar bekletir. Örneğin, `FIRST` 3 (s1, s2, s3, s4) ayarı, her commit'in s1, s2, s3 ve s4 standby sunucularından seçilen üç yüksek öncelikli standby'ın yanıtlarını bekler. Listede daha önce adı geçen standby'lere daha yüksek öncelik verilir ve senkron olarak kabul edilir. Listede bulunan sonraki standby sunucuları potansiyel senkron standby'ları temsil eder. Mevcut senkron standby'lardan herhangi birinin bağlantısı kesildiğinde bir sonraki en yüksek öncelikli standby ile değiştirilir. `FIRST` anahtar kelimesi isteğe bağlıdır.<br/><br/>

`ANY` anahtar kelimesi **num_sync** ile birleştiğinde, quorum tabanlı senkron bir replikasyonu belirtir ve transaction commit'lerini, WAL kayıtlarını listelenen en az **num_sync** standby'a replike edene kadar bekletir. Örneğin, `ANY 3 (s1, s2, s3, s4)` ayarı s1, s2, s3 ve s4 standby'larından en az herhangi üçü yanıt verir vermez commit'lerin devam etmesine neden olur.<br/><br/>

`FIRST` ve `ANY` büyük / küçük harf duyarlı değildir. Bu anahtar kelimeler bir standby sunucusu adı olarak kullanılıyorsa, **standby_name** çift tırnak içine alınmalıdır.<br/><br/>

PostgreSQL 9.6 sürümünden önce kullanılan üçüncü sözdizimi hala desteklenmektedir. `FIRST` ve **num_sync**'in 1'e eşit olduğu ilk sözdizimi kullanımı ile aynıdır. Örneğin, `FIRST 1 (s1, s2)` ile `s1, s2` aynı anlama gelir: s1 veya s2 senkron standby olarak seçilir.<br/><br/>

Özel giriş `*`, herhangi bir standby adıyla eşleşir.<br/><br/>

Standby adlarının benzersiz olmasını zorlayan bir mekanizma yoktur. Yineleme durumunda, eşleşen standby'lardan biri daha yüksek öncelik olarak kabul edilir ancak bunun tam olarak hangisi olacağı belirsizdir." type="primary" %}

{% include note.html content=" Her **standby_name**, * olmadığı sürece geçerli bir SQL tanımlayıcısı formatında olmalıdır. Gerekirse çift tırnak kullanılabilir. **standby_name**'lerinin, çift tırnaklı olsun ya da olmasın standby durumundaki uygulama adlarıyla büyük / küçük harfe duyarlı olmadan karşılaştırılır."%}

{% include callout.html content="Senkron standby adı belirtilmezse, senkron replikasyon etkinleştirilmez ve transaction commit'ler replikasyonu beklemez. Bu, varsayılan yapılandırmadır. Senkron replikasyon etkinleştirildiğinde bile, bireysel transaction'lar `synchronous_commit` parametresini `local` veya `off` olarak ayarlanarak replikasyonu beklemeyecek şekilde yapılandırılabilir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`vacuum_defer_cleanup_age (integer)`**: `VACUUM` ve HOT ölü satır (dead row) sürümlerinin temizlenmesini erteleyeceği transaction sayısını belirtir. Varsayılan sıfır transaction'dır, ölü satır sürümleri mümkün olan en kısa sürede, yani açık transaction'lar tarafından visible olmaktan çıkar çıkmaz kaldırılır. [Hot Standby](-) bölümünde açıklandığı gibi, etkin standby sunucularını destekleyen bir primary sunucuda bu sıfır olmayan bir değere ayarlanabilir. Bu sayede, standby sorguların satırların erken temizlenmesinden kaynaklanacak çakışmalar olmaksızın tamamlanması için daha fazla zaman sağlanır. Ancak değer primary sunucuda gerçekleşen yazma transaction'larının sayısı olarak ölçüldüğünden, standby sorguları için ne kadar ek yetkisiz kullanım süresi sağlanacağını tahmin etmek zordur. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.

Ayrıca bu parametreyi kullanmaya alternatif olarak standby sunucuları üzerinde `hot_standby_feedback` ayarlamayı da düşünmelisiniz.

Bu, `old_snapshot_threshold` ile belirtilen yaşa ulaşan ölü satırların temizlenmesini engellemez." type="primary" %}

### Standby Servers

Bu başlıkta verilen ayarlar replikasyon verilerini alacak standby sunucusunun davranışını kontrol eder.

{% include callout.html content="**`primary_conninfo (string)`**: Standby sunucusunun gönderen sunucuya bağlanması için kullanılacak bağlantı dizesini belirtir. Bu dize [burada](https://www.postgresql.org/docs/current/libpq-envars.html) açıklanan formattadır. Bu dizede herhangi bir seçenek belirtilmemişse, ilgili ortam değişkeni kontrol edilir. Ortam değişkeni de ayarlanmadıysa varsayılanlar kullanılır.<br/><br/>

Bağlantı dizesi, gönderen sunucunun host adını / adresini ve standby sunucusunun varsayılanı ile aynı değilse port'u belirtmelidir. Ayrıca, gönderen sunucuda uygun ayrıcalıklı bir role karşılık gelen bir kullanıcı adı belirtin (bkz. [](https://www.postgresql.org/docs/13/warm-standby.html#STREAMING-REPLICATION-AUTHENTICATION)). Gönderen, parola doğrulaması talep etmesi durumunda bir parolanın da sağlanması gerekir. `primary_conninfo` dizesinde veya standby sunucusundaki ayrı bir `~/.pgpass` dosyasında sağlanabilir (veritabanı adı olarak **replication** kullanın). `primary_conninfo` dizesinde bir veritabanı adı belirtmeyin.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. WAL receiver süreci çalışırken bu parametre değiştirilirse, bu sürecin kapanması için sinyal gönderilir ve yeni ayarla yeniden başlatılması beklenir (`primary_conninfo`'nun boş dizesi olması dışında). Sunucu standby modunda değilse bu ayarın hiçbir etkisi yoktur." type="primary" %}

{% include callout.html content="**`primary_slot_name (string)`**: Gönderen sunucuda kullanılacak replikasyon slotunun adını ayarlar. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. WAL receiver süreci çalışırken bu parametre değiştirilirse, bu sürecin kapanması için sinyal gönderilir ve yeni ayarla tekrar başlatılması beklenir. `primary_conninfo` ayarlanmamışsa veya sunucu standby modunda değilse bu ayarın hiçbir etkisi yoktur." type="primary" %}

{% include callout.html content="**`promote_trigger_file (string)`**: Standby modunda recovery'i sona erdiren bir tetikleyici (trigger) dosyası belirtir. Bu değer ayarlanmasa bile, `pg_ctl promote` veya `pg_promote ()` ile standby'ı yine de promote edebilirsiniz. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`hot_standby (boolean)`**: Recovery sırasında bağlantılara ve sorgulara izin verir. [Hot Standby](https://www.postgresql.org/docs/current/hot-standby.html)'de açıklandığı gibi recovery sırasında bağlanıp sorgu çalıştırıp çalıştırmayacağınızı belirtir. Öntanımlı değeri `on`. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Yalnızca arşiv recovery sırasında ve standby modunda etkilidir." type="primary" %}

{% include callout.html content="**`max_standby_archive_delay (integer)`**: Hot standby etkin olduğunda, bu parametre işlenen WAL verileriyle çakışan standby sorgularını iptal etmeden önce standby sunucunun maksimum gecikmesini ayarlar. `max_standby_archive_delay`, WAL verileri WAL arşivinden okunduğunda uygulanır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı 30 saniyedir. -1 değeri, standby'ın çakışan sorguların tamamlanması için sonsuza kadar beklemesine izin verir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

`max_standby_archive_delay`, bir sorgunun iptal edilmeden önce çalışabileceği maksimum süreyle aynı değildir, herhangi bir WAL segmentinin verilerini uygulamak için izin verilen toplam maksimum süredir. Bu nedenle, WAL segmentinde daha önce bir sorgu önemli gecikmeye neden olduysa, sonraki çakışan sorgular çok daha az yetkisiz kullanım süresine sahip olacaktır." type="primary" %}

{% include tip.html content=" Yük devretme amaçlı replikasyon yapıyorsanız, standby'ı olabildiğince güncel tutmak için bunu çok düşük bir değere (0 gibi) ayarlayın. Bu standby birincil rolü sorgu çalıştırmaksa, izin vermek istediğiniz en uzun süre çalışan sorgunun süresini ayarlayın."%}

{% include callout.html content="**`max_standby_streaming_delay (integer)`**: Hot Standby etkin olduğunda, bu parametre işlenen WAL verileriyle çakışan standby sorgularını iptal etmeden önce standby sunucunun maksimum gecikmesini ayarlar. `max_standby_streaming_delay`, WAL verileri streaming replikasyon yoluyla alındığında uygulanır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 30 saniyedir. -1 değeri, standby'ın çakışan sorguların tamamlanması için sonsuza kadar beklemesine izin verir. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir.<br/><br/>

`max_standby_streaming_delay`, bir sorgunun iptal edilmeden önce çalışabileceği maksimum süre ile aynı değildir, primary sunucudan alındıktan sonra WAL verilerinin uygulanması için izin verilen maksimum süredir. Bu nedenle, bir sorgu önemli bir gecikmeye yol açtıysa sonraki çakışan sorgular standby sunucu tekrar yakalanıncaya kadar çok daha az yetkisiz kullanım süresine sahip olacaktır." type="primary" %}

{% include tip.html content=" Yük devretme amaçlı replikasyon yapıyorsanız, standby'ı olabildiğince güncel tutmak için bunu çok düşük bir değere (0 gibi) ayarlayın. Bu standby birincil rol olarak sorgu çalıştırıyorsa, izin vermek istediğiniz en uzun süre çalışan sorgunun süresini ayarlayın."%}

{% include callout.html content="**`wal_receiver_create_temp_slot (boolean)`**: Kalıcı bir slot yapılandırılmamışsa (`primary_slot_name` kullanarak) bir WAL alıcısının geçici bir replikasyon slotu oluşturup oluşturmayacağını belirler. Varsayılan kapalıdır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. WAL receiver süreci çalışırken bu parametre değiştirilirse, bu sürecin kapanması için sinyal gönderilerek yeni ayarla tekrar başlatılması beklenir." type="primary" %}

{% include callout.html content="**`wal_receiver_status_interval (integer)`**: Primary ve upstream standby'a replikasyon ilerlemesi hakkında bilgi göndermek için standby üzerindeki WAL receiver süreci için minimum frekansı belirtir. Bu bilgiler [pg_stat_replication](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-VIEW) view'ı kullanılarak görüntülenir. Standby, yazdığı son WAL kaydının konumunu, diske flush edilen yer ve uyguladığı son konumu roparlar. Parametrenin değeri bu raporlar arasındaki maksimum süredir. Değişiklikler, yazma veya flush konumları her değiştiğinde veya bu parametre ile belirtilen sıklıkta gönderilir. Bu nedenle, uygulama konumu gerçek konumun biraz gerisinde kalabilir. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı değeri 10 saniyedir. Bu parametrenin 0 olarak ayarlanması, durum değişikliklerini tamamen devre dışı bırakır. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`hot_standby_feedback (boolean)`**: Bir hot standby'ın yürütülen sorgular hakkında primary veya upstream standby'a geri bildirim gönderip göndermeyeceğini belirtir. Bu parametre, temizleme kayıtlarından kaynaklı sorgu iptallerini ortadan kaldırmak için kullanılır, ancak bazı iş yüklerinde primary veritabanı şişmesine neden olabilir. Geri bildirim mesajları, herbir `wal_receiver_status_interval`'da bir defadan fazla gönderilmeyecektir. Varsayılan değeri `off`'dur. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Cascaded replikasyon kullanımdaysa, geri bildirim primary'e ulaşana kadar upstream'e iletilir. Standby'lar aldıkları geri bildirimi, upstream'e göndermek dışında hiçbir şekilde kullanmaz.<br/><br/>

Bu ayar, primary üzerindeki `old_snapshot_threshold` davranışını geçersiz kılmaz, standby'da primary'nin yaş eşiğini aşan bir snapshot geçersiz hale gelebilir ve bu, standby'da transaction'ların iptal edilmesiyle sonuçlanır. Bunun nedeni, `old_snapshot_threshold`'un ölü satırlardan kaynaklacak şişme için mutlak bir zaman sınırı sağlamayı amaçlamasıdır. Aksi takdirde bir standby konfigürasyonu nedeniyle geçersiz kılanabilir." type="primary" %}

{% include tip.html content=" Çoğu durumda replikasyonlarda sorgu iptalini önlemeye yardımcı olur. Uzun süreli raporlar veren ve gecikmesine izin verilen bir replika için bunu kapatın."%}

{% include callout.html content="**`wal_receiver_timeout (integer)`**: Bu süreden daha uzun süre etkin olmayan replikasyon bağlantılarını sonlandırır. Bu parametre, alıcı standby sunucunun primary düğüm çökmesi veya ağ kesintisini algılaması için kullanışlıdır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 60 saniyedir. 0 değeri, zaman aşımı mekanizmasını devre dışı bırakır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`wal_retrieve_retry_interval (integer)`**: Standby sunucusunun, herhangi bir kaynakta (streaming replikasyon, yerel `pg_wal` veya WAL arşivi) WAL verileri hazır olmadığında beklenecek süreyi belirtir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Ötanımlı değeri 5 saniyedir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Bu parametre, recovery'deki bir düğümün yeni WAL verilerinin hazır olması için bekleme süresini kontrol etmesi gereken yapılandırmalarda yararlıdır. Örneğin, arşiv recovery'de, bu parametre değerini azaltarak yeni WAL log dosyası tespitinde recovery'i daha duyarlı hale getirmek mümkündür. Düşük WAL etkinliğine sahip bir sistemde bu değeri arttırmak, WAL arşivine erişim istek miktarını azaltır. Bu, altyapıya erişme miktarının hesaba katıldığı bulut ortamlarında faydalıdır." type="primary" %}

{% include callout.html content="**`recovery_min_apply_delay (integer)`**: Bir standby sunucusu varsayılan olarak WAL kayıtlarını gönderen sunucudan mümkün olan en kısa sürede yeniler. Verilerin gecikmeli bir kopyasına sahip olmak veri kaybı hatalarını düzeltmek için faydalı olabilir. Bu parametre recovery'i belirli bir süre geciktirmenizi sağlar. Örneğin, bu parametreyi `5min` olarak ayarlarsanız standby, her transaction commit'ini yalnızca standby'daki sistem süresi primary tarafından bildirilen commit süresinden en az beş dakika geçtiğinde replay eder. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan sıfırdır, gecikme yoktur.<br/><br/>

Sunucular arasındaki replikasyon gecikmesinin bu parametrenin değerini aşması durumunda herhangi bir gecikme eklenmez. Gecikmenin, primary'de yazılan WAL zaman damgası ile standby'daki saat arasında hesaplandığını unutmayın. Ağ gecikmesi veya cascading replikasyon yapılandırmaları sebepli aktarımdaki gecikmeler mevcut bekleme süresini önemli ölçüde azaltabilir. Primary ve standby'daki sistem saatleri senkronize değilse kayıtlar beklenenden daha erken uygulanabilir. Ancak bu önemli bir sorun değildir çünkü bu parametrenin kullanışlı ayarları sunucular arasındaki tipik zaman sapmalarından çok daha büyüktür.<br/><br/>

Gecikme, yalnızca transaction commit'leri için WAL kayıtlarında meydana gelir. Diğer kayıtlar olabildiğince çabuk replay edilir. Bu bir problem değildir çünkü MVCC görünürlük (visibility) kuralları, ilgili commit kaydı uygulanana kadar etkilerinin görünür olmamasını sağlar.<br/><br/>

Gecikme, recovery sırasında veritabanı tutarlı bir duruma ulaştığında, standby promote edilene veya tetiklenene kadar gerçekleşir ve sonrasında standby daha fazla beklemeden recovery'i sonlandırır.<br/><br/>

Bu parametre, streaming replikasyon deploymentları ile kullanmak üzere tasarlanmış olmasına rağmen parametre belirtildiğinde crash recovery dışındaki tüm durumlarda dikkate alınacaktır. `hot_standby_feedback` bu özelliğin kullanılmasıyla ertelenir ve bu da master'da şişmeye neden olabilir. İkisinin birlikte kullanırken dikkatli olunması önerilir.<br/><br/>

Bu parametre yalnızca postgresql.conf dosyasında veya sunucu komut satırında ayarlanabilir." type="primary" %}

{% include warning.html content="Senkron replikasyon, `synchronous_commit = remote_apply` olarak ayarlandığında bu ayardan etkilenir. Her COMMIT'in uygulanmak için beklemesi gerekecektir."%}

### Subscribers

Bu başlıkta verilen ayarlar logical replikasyon subscriber davranışını kontrol eder. publisher'daki değerleri konu dışıdır.

{% include callout.html content="**`max_logical_replication_workers (int)`**: Maksimum logical replikasyon worker sayısını belirtir. Bu, hem uygulayıcı hem de tablo senkronizasyon worker'ları kapsar. Logical replikasyon worker, `max_worker_processes` tarafından tanımlanan havuzdan alınır. Öntanımlı değer 4'tür." type="primary" %}

{% include callout.html content="**`max_sync_workers_per_subscription (integer)`**: Herbir subscription için maksimum senkronizasyon worker'ı sayısı belirtir. Bu parametre, subscription başlatma sırasında veya yeni tablolar eklendiğinde ilk veri kopyasının paralellik miktarını kontrol eder. Şu an için her tablo başına yalnızca bir senkronizasyon worker'ı olabilir. Senkronizasyon worker'ları `max_logical_replication_workers` tarafından tanımlanan havuzdan alınır. Öntanımlı değeri 2'dir." type="primary" %}

{% include note.html content=" **wal_receiver_timeout**, **wal_receiver_status_interval** ve **wal_retrieve_retry_interval** yapılandırma parametreleri de logical replication worker'larını etkiler."%}

{% include links.html %}
