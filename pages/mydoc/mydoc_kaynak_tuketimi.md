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

### Arkaplan Yazarı ( Background Writer )

### Eşzamansız Davranış ( Asynchronous Behavior )

{% include links.html %}
