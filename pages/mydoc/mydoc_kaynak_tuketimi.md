---
title: "Kaynak Tüketimi"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 27, 2020
summary: "Kaynak Tüketimi"
sidebar: mydoc_sidebar
permalink: mydoc_kaynak_tuketimi.html
folder: mydoc
---

## Kaynak Tüketimi

### Hafıza

{% include callout.html content="`shared_buffers (integer)`: Veritabanı sunucusunun paylaşılan bellek arabellekleri (shared memory buffers) için kullandığı bellek miktarını ayarlar. Öntanımlı değer 128 megabayttır (128MB), ancak çekirdek ayarlarınız bunu desteklemiyorsa daha az olabilir. Bu ayar en az 128 kilobayt olabilir. İyi performans isteniyorsa minimumdan önemli ölçüde daha yüksek ayarlar gereklidir. Bu değer birimsiz verilirse bloklar (BLCKSZ = 8kb) olarak alınır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

1 GB veya fazlası RAM'e sahip veritabanı sunucuları için sisteminizdeki belleğin %25'i makul bir `shared_buffers` başlangıç ​​değeridir. Daha büyük shared_buffers ayarlarının etkili olduğu bazı iş yükleri olabilir, ancak PostgreSQL aynı zamanda işletim sistemi önbelleğide kullandığı için RAM'in %40'ından fazlasının shared_buffers'a tahsis edilmesi daha küçük bir miktara kıyasla iyi bir performans göstermeyecektir. shared_buffers için daha büyük ayarlar, büyük miktarlarda yeni veya değiştirilmiş veri yazma sürecini daha uzun bir periyoda yaymak için genellikle `max_wal_size` değerinde bir artış gerektirir.<br/><br/>

1 GB'den az RAM'e sahip sistemlerde, işletim sistemi için yeterli alan bırakmak amacıyla daha küçük bir RAM yüzdesi uygundur." type="primary" %}

{% include callout.html content="`huge_pages (enum)`: Ana paylaşılan hafıza alanı için büyük sayfaların istenip istenmediğini kontrol eder. Geçerli değerler, `try` (varsayılan), `on` ve `off` şeklindedir. huge_pages `try` olarak ayarlandığında sunucu çok büyük sayfalar istemeye çalışır, ancak başarısız olursa varsayılana geri döner. `on` olduğunda büyük sayfaların istenmemesi sunucunun başlamasını engelleyecektir. `off` olduğunda büyük sayfalar istenmeyecektir.<br/><br/>

Bu ayar şuan için yalnızca Linux ve Windows'ta desteklenmektedir. Diğer sistemlerde `try` olarak ayarlandığında yok sayılır.<br/><br/>

Büyük sayfaların kullanımı, daha küçük sayfa tabloları ve bellek yönetimi için daha az CPU zamanı harcaması sağlayarak performansı arttırır. Linux üzerinde büyük sayfalar kullanma hakkında ayrıntı için, [Çekirdek Kaynaklarını Yönetme](https://www.postgresql.org/docs/current/kernel-resources.html#LINUX-HUGE-PAGES) bölümüne bakın.<br/><br/>

Bu ayar yalnızca ana paylaşılan hafıza alanını (shared memory area) etkiler. Linux, FreeBSD ve Illumos gibi işletim sistemleri PostgreSQL'den açık bir istek olmaksızın normal bellek tahsisi için otomatik olarak büyük sayfaları ('super' sayfalar, 'large' sayfalar olarak da bilinir) kullanabilir. Linux'ta buna şeffaf büyük sayfalar ( transparent huge pages - THP) denir. Bu özelliğin, bazı Linux sürümlerinde PostgreSQL ile performans düşüşüne neden olduğu bilindiğinden kullanımı şu anda tavsiye edilmemekte." type="primary" %}

{% include callout.html content="`temp_buffers (integer)`: Her veritabanı oturumunda geçici tamponlar (temporary buffers) için kullanılan maksimum bellek miktarını ayarlar. Bu alanlar, yalnızca geçici tablolara erişim için kullanılan session-local arabellekleridir. Bu değer birimsiz belirtilirse, `BLCKSZ` ( genellikle 8kB ) olarak alınır . Öntanımlı 8 megabayttır (8MB). BLCKSZ 8kB değilse öntanımlı değer buna orantılı olarak ölçeklenir. Bu ayar, bireysel oturumlar içinde yalnızca oturumdaki geçici tabloların ilk kullanımından önce değiştirilebilir. Değeri değiştirmeye yönelik sonraki girişimlerin o oturum üzerinde hiçbir etkisi olmayacaktır.<br/><br/>

Bir oturum, `temp_buffers` tarafından verilen sınıra kadar gerektiğinde geçici arabellekler tahsis edebilir. Geçici arabelleğe pek ihtiyaç duymayan oturumlarda ayarlanan büyük bir değerin maliyeti arabellek tanımlayıcısı veya `temp_buffers`'daki artış başına yaklaşık 64 bayttır. Arabellek kullanılıyorsa bunun için ek 8192 bayt (genelde BLCKSZ bayt) tüketilecektir." type="primary" %}

{% include callout.html content="`max_prepared_transactions (integer)`: Aynı anda `prepared` durumda olabilecek maksimum işlem sayısını ayarlar. Parametrenin sıfıra (varsayılan değerdir) ayarlanması hazırlanmış işlemler ( prepared-transaction ) özelliğini devre dışı bırakır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

Hazırlanmış işlemler kullanmayı planlamıyorsanız yanlışlıkla oluşturulmasını önlemek için bu parametre sıfıra ayarlayın. Hazırlanmış işlemleri kullanıyorsanız, muhtemelen `max_prepared_transactions` değerinin en az `max_connections` kadar olmasını isteyeceksiniz böylece her oturum bekleyen hazırlanmış bir işleme sahip olabilir.<br/><br/>

Standby sunucuda bu parametreyi ana sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde standby sunucuda sorgulara izin verilmeyecektir." type="primary" %}

{% include callout.html content="`work_mem (integer)`: Sıralama, karma (hash) tablo gibi sorgu işlemlerinde geçici disk dosyalarına yazmadan önce kullanılacak maksimum bellek miktarını ayarlar. Bu değer birimsiz olarak verilirse kilobayt olarak alınır. Öntanımlı değer 4 megabayttır (4MB). Karmaşık sorgularda birkaç sıralama veya karma işlemin paralel olarak çalıştırılabilir. Her operasyonun verileri geçici dosyalara yazmaya başlamadan önce bu değerin belirlediği kadar bellek kullanmasına izin verilir. Ayrıca, bu tür işlemleri çalışan birkaç oturum aynı anda yapıyor olabilir. Bu nedenle tüm kullanılan bellek, `work_mem` değerinin birçok katı olmalıdır. Değeri seçerken bu gerçeği göz önünde bulundurmalısınız. Sıralama operasyonları `ORDER BY`, `DISTINCT` ve merge join için kullanılır. Karma tablolar, karma birleştirmelerde (hash joins), karma tabanlı toplamada (hash-based aggregation) ve `IN` alt sorgularının karma tabanlı işlenmesinde kullanılır.<br/><br/>

Karma tabanlı operasyonlar kullanılabilir belleğe sıralama tabanlı işlemlere göre daha duyarlıdır. Karma tablolar için kullanılabilir bellek `work_mem` ile `hash_mem_multiplier` çarpımından hesaplanır. Bu, karma tabanlı işlemlerin normal work_mem temel miktarını aşan miktarda bellek kullanmasını sağlar." type="primary" %}

{% include callout.html content="`hash_mem_multiplier (floating point)`: Karma tabanlı işlemlerin kullanabileceği maksimum bellek miktarını hesaplamak için kullanılır. Nihai sınır `work_mem`'in `hash_mem_multiplier` ile çarpılmasıyla belirlenir. Öntanımlı değer 1.0'dır ve karma tabanlı işlemleri sıralama tabanlı işlemlerle aynı maksimum work_mem'e tabi kılar.<br/><br/>

Özellikle, sorgu işlemleriyle yayılmanın düzenli bir olay olduğu ve yalnızca work_mem'i artırmanın bellek baskısına sebep olduğu ortamlarda `hash_mem_multiplier` artırılabilir. 1.5 veya 2.0 değeri karışık iş yüklerinde etkili olabilir. work_mem'in halihazırda 40MB veya daha fazlasına yükseltildiği ortamlarda 2.0 - 8.0 aralığındaki veya daha yüksek değerler etkili olabilir." type="primary" %}

{% include callout.html content="`maintenance_work_mem (integer)`: `VACUUM`, `CREATE INDEX` ve `ALTER TABLE ADD FOREIGN KEY` gibi bakım operasyonlarında kullanılacak maksimum bellek miktarını belirtir. Bu değer birimsiz verildiğinde kilobayt olarak alınır. Öntanımlı olarak 64 megabayttır (64MB). Bir veritabanı oturumu tarafından aynı anda bu işlemlerden yalnızca biri yürütülebilir ve normal kurulumda çoğu eşzamanlı olarak çalışmaz. Bu sebeple bu değeri `work_mem`'den önemli ölçüde daha büyük ayarlamak güvenlidir. Daha büyük ayarlar vakumlama ve veritabanı dump'larından geri yükleme performansını artırır.<br/><br/>

Otomatik vakum çalıştığında bu bellek `autovacuum_max_workers` kez tahsis edilebileceğinden varsayılan değeri çok yüksek ayarlamamaya dikkat edin. Bunu, `autovacuum_work_mem`'i ayrı ayrı ayarlayarak kontrol etmek faydalı olabilir." type="primary" %}

{% include callout.html content="`autovacuum_work_mem (integer)`: Her bir autovacuum çalışan süreci ( autovacuum worker process ) tarafından kullanılacak maksimum bellek miktarını belirtir. Bu değer birimsiz verildiğinde kilobayt olarak alınır. Öntanımlı olarak -1'dir ve bunun yerine `maintenance_work_mem` değerinin kullanılması gerektiğini belirtir. Ayarın, diğer bağlamlarda çalıştırıldığında VACUUM davranışı üzerinde hiçbir etkisi yoktur." type="primary" %}

{% include callout.html content="`logical_decoding_work_mem (integer)`: Çözülen değişikliklerin bazılarının yerel diske yazılmadan önce mantıksal kod çözme ( logical decoding ) tarafından kullanılacak maksimum bellek miktarını belirtir. Bu, mantıksal akış çoğaltma ( logical streaming replication ) bağlantıları tarafından kullanılan bellek miktarını sınırlar. Öntanımlı olarak 64 megabayttır (64MB). Her çoğaltma bağlantısı yalnızca bu boyutta tek bir arabellek kullandığından ve bir kurulumda normalde aynı anda bu tür çok sayıda bağlantı bulunmadığından, bu değeri `work_mem`'den önemli ölçüde daha yükseğe ayarlamak güvenlidir. Böylece, diske yazılan kodu çözülmüş değişikliklerin miktarı azaltılır." type="primary" %}

{% include callout.html content="`max_stack_depth (integer)`: Sunucunun yürütme yığınının (execution stack) maksimum güvenli derinliğini belirtir. Bu parametre için ideal ayar çekirdek tarafından zorlanan mevcut yığın boyutu sınırıdır. Yığın derinliği sunucudaki her rutinde değil, yalnızca potansiyel olarak yinelemeli rutinlerde kontrol edilir. Bu sebeple güvenlik marjı gereklidir. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı ayar 2 megabayttır (2MB). Bu değer karmaşık fonksiyonların yürütülmesine izin vermek için yeterli olmayabilir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

`max_stack_depth`'in mevcut çekirdek sınırından daha yüksek ayarlanması kontrolden çıkmış bir özyinelemeli fonksiyonun bir arka uç işlemini çökertebileceği anlamına gelir. PostgreSQL'in çekirdek sınırını belirleyebildiği platformlarda, sunucu bu değişkenin güvenli olmayan bir değere ayarlanmasına izin vermez. Ancak, tüm platformlar bilgi sağlamadığı için bu değer seçilirken dikkatli olunması önerilir." type="primary" %}

{% include callout.html content="`shared_memory_type (enum)`: Ana paylaşılan bellek alanı için sunucunun kullanması gereken paylaşılan bellek uygulamasını belirtir. Alabileceği değerler: `mmap` (mmap kullanılarak ayrılan anonim paylaşılan bellek için), `sysv` (shmget ile ayrılan System V paylaşılan bellek için) ve `windows` (Windows paylaşılan bellek için). Her platformlarda tüm değerler. Desteklenen ilk seçenek, o platform için varsayılandır. Hiçbir platformda varsayılan olmayan `sysv` seçeneğinin kullanılması genellikle tavsiye edilmez, çünkü büyük tahsislere izin vermek için varsayılan olmayan çekirdek ayarları gerektirir (bkz. [](https://www.postgresql.org/docs/13/kernel-resources.html#SYSVIPC))." type="primary" %}

{% include callout.html content="`dynamic_shared_memory_type (enum)`: Sunucunun kullanacağı dinamik paylaşılan bellek uygulamasını belirtir. Alabileceği değerler: `posix` (shm_open kullanılarak ayrılan POSIX paylaşılan bellek için), `sysv` (shmget aracılığıyla ayrılan System V paylaşılan bellek için), `windows` (Windows paylaşılan bellek için) ve `mmap` (verilerde depolanan bellek eşlemeli dosyaları kullanarak paylaşılan belleği simüle etmek için). Her platformlarda tüm değerler desteklenmez. Desteklenen ilk seçenek, o platform için varsayılandır. Herhangi bir platformda varsayılan olmayan `mmap` seçeneğinin kullanılması genellikle tavsiye edilmez, çünkü işletim sistemi değiştirilen sayfaları tekrar tekrar diske yazabilir ve bu da sistem I / O yükünü artırır. Ancak bu, `pg_dynshmem` dizininin bir RAM diskte depolandığı veya diğer paylaşılan bellek olanakları kullanılamadığı durumlarda hata ayıklama için yararlı olabilir." type="primary" %}

### Disk

{% include callout.html content="`temp_file_limit (integer)`: Bir sürecin geçici dosyalar (temporary files) için kullanabileceği maksimum disk alanı miktarını belirtir. Bu sınırı aşmaya çalışan bir transaction iptal edilecektir. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı -1 değeri sınır yok demektir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu ayar, herhangi bir PostgreSQL sürecinin herhangi bir anda kullandığı geçici dosyarın kapladığı toplam alanı sınırlar. Sorgu yürütmede perde arkasında kullanılan geçici dosyaların aksine açık geçici ( explicit temporary ) tablolar için kullanılan disk alanı bu sınıra dahil değildir." type="primary" %}

### Çekirdek Kaynak Kullanımı

{% include callout.html content="`max_files_per_process (integer)`: Sunucu alt süreçlerine (subprocess) izin verilen maksimum eşzamanlı açık dosya sayısını ayarlar. Varsayılan 1000 dosyadır. Çekirdek, süreç başına güvenli bir sınır uyguluyorsa, bu ayar için endişelenmenize gerek yoktur. Ancak bazı platformlarda, özellikle çoğu BSD sisteminde, çekirdek süreçlerin sistemin destekleyebileceğinden çok daha fazla dosya açmasına izin verir. Eğer 'Too many open files' hatası ile karşılaşırsanız bu ayarı düşürmeyi deneyin. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

### Maliyete Dayalı Vakum Gecikmesi ( Cost-based vacuum delay )

`VACUUM` ve `ANALYZE` komutlarının yürütülmesi sırasında, sistem gerçekleştirilen çeşitli I / O operasyonlarının tahmini maliyetini izleyen dahili bir sayaç tutar. Birikmiş maliyet `vacuum_cost_limit` ile belirtilen limite ulaştığında, operasyonu gerçekleştiren süreç `vacuum_cost_delay` ile belirtilen süre boyunca uyuyacaktır. Ardından sayacı sıfırlayarak çalışmaya devam edecektir.

Bu özelliğin amacı yöneticilerin bu komutların eşzamanlı veritabanı etkinliği üzerindeki I / O etkisini azaltmalarına olanak sağlamaktır. VACUUM ve ANALYZE gibi bakım komutlarının hızla bitmesinin önemli olmadığı birçok durum vardır; ancak bu komutların sistemin diğer veritabanı operasyonlarını gerçekleştirme becerisine önemli ölçüde müdahale etmemesi çok önemlidir. Maliyete dayalı vakum gecikmesi, yöneticilerin bunu başarması için bir yol sunar.

Bu özellik, manuel olarak verilen VACUUM komutları için öntanımlı olarak devre dışı bırakılmıştır. Bunu etkinleştirmek isterseniz `vacum_cost_delay` değişkenini sıfır olmayan bir değere ayarlayın.

{% include callout.html content="`vacuum_cost_delay (floating point)`: Maliyet sınırı geçildiğinde sürecin uyuyacağı süre. Bu değer birimsiz verilirse milisaniye olarak alınır. Öntanımlı değer sıfırdır ve maliyete dayalı vakum gecikmesi özelliğini devre dışı bırakır. Pozitif değerler maliyete dayalı vakumlamaya izin verir.<br/><br/>

Maliyete dayalı vakumlama kullanılırken `vacuum_cost_delay` için uygun değerler genellikle 1 milisaniye ile ifade edecek kadar çok küçüktür. vacuum_cost_delay kesirli milisaniye değerlerine ayarlanabilir olsa da bu tür gecikmeler eski platformlarda doğru ölçülemeyebilir. Bu tür platformlarda VACUUM'un daraltılmış kaynak tüketimini 1 ms'de elde ettiğinizin üzerine çıkarmak diğer vakum maliyeti parametrelerini değiştirmeyi gerektirecektir. Yine de vacuum_cost_delay'i platformunuzun tutarlı bir şekilde ölçeceği kadar küçük tutmalısınız, büyük gecikmeler yardımcı olmaz." type="primary" %}

{% include callout.html content="`vacuum_cost_page_hit (integer)`: Paylaşılan arabellek önbelleğinde ( shared buffer cache ) bulunan bir arabelleği vakumlamanın tahmini maliyeti. Tampon havuzunu ( buffer pool ) kilitlemenin, paylaşılan karma tablosunu arama ve sayfanın içeriğini taramanın maliyetini temsil eder. Öntanımlı değer 1'dir.
" type="primary" %}

{% include callout.html content="`vacuum_cost_page_miss (integer)`: Diskten okunması gereken bir arabelleği vakumlamanın tahmini maliyeti. Bu, tampon havuzunu kilitleme, paylaşılan karma tablosunu arama, istenen bloğu diskten okuma ve içeriğini tarama çabasını temsil eder. Öntanımlı değer 10'dur." type="primary" %}

{% include callout.html content="`vacuum_cost_page_dirty (integer)`: Vakum önceden temiz olan bir bloğu değiştirdiğinde alınan tahmini maliyet. Kirli bloğu tekrar diske geçirmek için gereken ekstra I / O'u temsil eder. Öntanımlı değer 20'dir." type="primary" %}

{% include callout.html content="`vacuum_cost_limit (integer)`: Vakumlama sürecinin uyumasına neden olacak birikmiş maliyet. Öntanımlı değer 200'dür." type="primary" %}

{% include note.html content="Kritik kilitleri tutan ve olabildiğince çabuk tamamlanması gereken belirli operasyonlar vardır. Bu tür operasyonlar sırasında maliyete dayalı vakum gecikmeleri yaşanmaz. Bu nedenle, maliyetin belirlenen limitin çok üzerinde birikmesi mümkündür. Bu gibi durumlarda gereksiz derecede uzun gecikmelerden kaçınmak için gerçek gecikme **vacuum_cost_delay * accumulated_balance / vacuum_cost_limit with a maximum of vacuum_cost_delay * 4** formülü ile hesaplanır." %}

### Arka Plan Yazarı ( Background Writer )

Arka plan yazarı ( Background Writer ) ayrı bir PostgreSQL sunucu sürecidir. Arka plan yazarı, "kirli" (dirty) yani yeni veya değiştirilmiş paylaşılan arabellekleri diske yazmakla görevlidir. Böylece, sunucu süreçleri kullanıcı sorgularını nadiren işler veya hiçbir zaman yazma işleminin gerçekleşmesini beklemesi gerekmez. Arka plan yazarı I / O yükünde net bir şekilde artışa neden olur, çünkü tekrar tekrar kirlenen bir sayfa her kontrol noktası ( checkpoint ) aralığında yalnızca bir kez yazılırken arka plan yazıcısı tarafından birkaç kez yazabilir. Bahsedeceğimiz parametreleri yerel ihtiyaçlarınıza yönelik davranışları ayarlamak için kullanabilirsiniz.

{% include callout.html content="`bgwriter_delay (integer)`: Arka planda yazarının aktif turları arasındaki süreyi belirtir. Arka plan yazarı her turda aşağıdaki parametreler ile kontrol edilebilen sayıdaki kirli tamponları diske işler. Daha sonra `bgwriter_delay` boyunca uyur ve bu işlemi tekrar eder. Arabellek havuzunda kirli arabellekler olmadığında bgwriter_delay'e bakmaksızın daha uzun süre uykuda kalabilir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı 200 milisaniyedir (200 ms). Birçok sistemde etkili değer 10 milisaniye olmakla bereaber bgwriter_delay'i 10'un katı olmayan bir değere ayarlamak, 10'un bir sonraki katına ayarlamakla aynı sonuçları verebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`bgwriter_lru_maxpages (integer)`: Her turda, arka plan yazarı tarafından `bgwriter_lru_maxpages` değerinde tampondan fazlası yazılmayacaktır. Bunu sıfıra ayarlamak, arka planda yazmayı devre dışı bırakır. Öntanımlı değeri 100 arabellektir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Ayrı bir yardımcı süreç tarafından yönetilen checkpoints ( kontrol noktaları ) bu ayardan etkilenmez." type="primary" %}

{% include callout.html content="`bgwriter_lru_multiplier (floating point)`: Her turda yazılan kirli arabelleklerin sayısı ile yakın zamandaki raundlar sırasında sunucu süreçlerinin ihtiyaç duyduğu yeni arabellek sayısı arasında bir ilişki vardır. Bir sonraki turda ihtiyaç duyulacak tahmini arabellek sayısı, yakın zamandaki ortalama ihtiyacın `bgwriter_lru_multiplier` değeri ile çarpımından bulunur. Kirli tamponlar, yeterli sayıda temiz, yeniden kullanılabilir uygun olana kadar yazılır. (her turda `bgwriter_lru_maxpages` arabelleklerinden fazlası yazılmaz.) 1.0 ayarı tam olarak ihtiyaç duyulan tahmini tampon sayısını yazmayı yani 'just in time' ilkesini temsil eder. Daha büyük değerler talepteki ani artışlara karşı bir miktar tampon sağlarken, küçük değerler ise kasıtlı olarak yazma işlemini sunucu süreçlerine bırakır. Öntanımlı 2.0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`bgwriter_flush_after (integer)`: Arkaplan yazarı tarafından bu miktardan daha fazla veri yazıldığında, işletim sistemini bu verileri esas depolama alınan yazmaya zorlar. Bununla, çekirdeğin sayfa önbelleğindeki (page cache) kirli veri miktarını sınırlandırılır ve bir denetim noktası (checkpoint) sonunda `fsync` yayınlandığında veya işletim sistemi verileri arka planda daha büyük gruplar halinde geri yazdığında durma olasılığını düşürülür. Bu genellikle büyük ölçüde azaltılmış transaction gecikmesini sağlar, ancak işletim sisteminin sayfa önbelleğinden daha küçük, `shared_buffers`'dan daha büyük iş yüklerinde performans kaybı yaşanabilir. Bu ayar bazı platformlarda hiç bir etkiye sahip olmayabilir. Bu değer birimsiz belirtilirse, blok yani `BLCKSZ` bayt (tipik olarak 8kB'dir) olarak alınır. Geçerli aralık, zorunlu geri yazmayı devre dışı bırakan 0 ile 2MB arasındadır. Linux'ta öntanımlı 512kB diğer sistemlerde 0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Eşzamansız Davranış ( Asynchronous Behavior )

{% include callout.html content="`effective_io_concurrency (integer)`: PostgreSQL'in aynı anda yürütülebilmesini beklediği eşzamanlı disk I / O operasyonlarının sayısını ayarlar. Bu değerin yükseltilmesi, herhangi bir PostgreSQL oturumunun paralel olarak başlatmaya çalıştığı I / O operasyonlarının sayısını artırır. İzin verilen aralık 1 ile 1000 arasıdır. 0 eşzamansız I / O isteklerini devre dışı bırakır. Şu an için bu ayar sadece bitmap heap taramalarını etkilemektedir.<br/><br/>

Manyetik sürücülerde bu ayar için iyi başlangıç ​​noktası kullanılan RAID 0 stripe veya RAID 1 mirror içeren ayrık sürücülerin sayısıdır. (RAID 5'de parity drive sayılmamalıdır). Ancak, veritabanı genellikle eşzamanlı oturumlarda yayınlanan birden çok soruyla meşgulse disk dizisini meşgul tutmak için daha düşük değerler yeterli olabilir. Diskleri meşgul tutmak için gerekenden daha yüksek bir değer beraberinde yalnızca fazladan CPU ek yükü getirir. SSD'ler ve diğer bellek tabanlı depolamalar çoğu zaman eşzamanlı birçok isteği işleyebildiğinden bunlar için en iyi değer yüzlerde olabilir.<br/><br/>

Eşzamansız I / O bazı işletim sistemlerinde bulunmayan [`posix_fadvise`](https://pubs.opengroup.org/onlinepubs/009695399/functions/posix_fadvise.html) fonksiyonuna bağlıdır. Fonksiyon mevcut değilse, bu parametrenin sıfır dışında herhangi bir değere ayarlanması hataya neden olur. Bazı işletim sistemlerinde (ör. Solaris), fonksiyon mevcut olmasına rağmen hiçbir şey yapmaz.<br/><br/>

Desteklenen sistemlerde öntanımlı 1'dir, aksi takdirde 0'dır. Bu değer, belirli bir tablo alanındaki (tablespace) tablolar için aynı isme sahip tablespace parametresi ayarlanarak geçersiz kılınabilir (bkz. [ALTER TABLESPACE](https://www.postgresql.org/docs/current/sql-altertablespace.html))." type="primary" %}

{% include callout.html content="`maintenance_io_concurrency (integer)`: `Effect_io_concurrency`'e benzer, ancak istemci oturumları adına yapılan bakım çalışmaları için kullanılır.<br/><br/>

Desteklenen sistemlerde öntanımlı 10'dur, aksi takdirde 0'dır. Bu değer, belirli bir tablo alanındaki (tablespace) tablolar için aynı isme sahip tablespace parametresi ayarlanarak geçersiz kılınabilir (bkz. [ALTER TABLESPACE](https://www.postgresql.org/docs/current/sql-altertablespace.html))." type="primary" %}

{% include callout.html content="`max_worker_processes (integer)`: Sistemin destekleyebileceği maksimum arka plan süreci sayısını ayarlar. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Öntanımlı değeri 8'dir.<br/><br/>

Standby sunucu çalıştırırken, bu parametreyi ana sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde, standby sunucuda sorgulara izin verilmeyecektir.<br/><br/>

Bu değer değiştirilirken `max_parallel_workers`, `max_parallel_maintenance_workers` ve `max_parallel_workers_per_gather` parametrelerinin ayarlanması düşünülebilir. İlgili paremetreler için [Kaynak tüketimi](mydoc_kaynak_tuketimi.html) bölümüne bakın." type="primary" %}

{% include callout.html content="`max_parallel_workers_per_gather (integer)`: Tek bir `Gather` veya `Gather Merge` düğümü tarafından başlatılabilecek maksimum işçi (worker) sayısını ayarlar. Paralel işçiler (workers), `max_worker_processes` tarafından oluşturulmuş, `max_parallel_workers` ile sınırlandırılmış süreç havuzundan alınır. İstenen içi sayısı çalışma zamanında mevcut olmayabilir. Bu durumda, plan beklenenden daha az sayıda işçi çalışarak verimsiz olabilir. Öntanımlı değer 2'dir. Bu değerin 0 olarak ayarlanması paralel sorgu yürütmeyi devre dışı bırakır.<br/><br/>

Paralel sorgular, paralel olmayan sorgularadan çok daha fazla kaynak tüketebilir. Çünkü her işçi (worker) süreci sistem üzerinde ek bir kullanıcı oturumuyla hemen hemen aynı etkiye sahip olan tamamen ayrı bir süreçtir. Bu ayara değer verilirken, `work_mem` gibi diğer kaynak kullanımını kontrol eden ayarlarda olduğu gibi bu durum dikkate alınmalıdır. `work_mem` gibi kaynak limitleri her bir işçi için ayrı ayrı uygulanır. Bu tüm süreçlerdeki toplam kullanımın, normalde herhangi bir süreç için olandan çok daha yüksek olabileceği anlamına gelir. Örneğin, 4 işçi kullanan bir paralel sorgu, hiç işçi kullanmayan bir sorguya göre 5 kat daha fazla CPU zamanı, bellek, I / O bant genişliği vb. kullanabilir.<br/><br/>

Paralel sorgu hakkında daha fazla bilgi için bkz [](https://www.postgresql.org/docs/current/parallel-query.html)." type="primary" %}

{% include callout.html content="`max_parallel_maintenance_workers (integer)`: Tek bir yardımcı program komutuyla başlatılabilen maksimum paralel işçi sayısını ayarlar. Şu an için paralel işçi kullanımını `CREATE INDEX` (B-tree indekslerde) ve `VACUUM` (FULL olmadan) işlemlerinde desteklenmektedir. Paralel işçiler (workers), `max_worker_processes` tarafından oluşturulmuş, `max_parallel_workers` ile sınırlandırılmış süreç havuzundan alınır. İstenen içi sayısı çalışma zamanında mevcut olmayabilir. Bu durumda, yardımcı program operasyonu beklenenden daha az sayıda işçi ile çalışacaktır. Öntanımlı değer 2'dir. Bu değerin 0 olarak ayarlanması yardımcı program komutlarının paralel işçi kullanmasını devre dışı bırakır." type="primary" %}

{% include links.html %}
