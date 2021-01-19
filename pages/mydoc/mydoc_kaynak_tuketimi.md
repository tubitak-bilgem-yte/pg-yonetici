---
title: "Kaynak Tüketimi"
tags: [PostgreSQL]
keywords: postgresql, shared_buffers, huge_pages, temp_buffers, max_prepared_transactions, work_mem, max_stack_depth, maintenance_work_mem, autovacuum_max_workers, autovacuum_work_mem, max_parallel_workers_per_gather
last_updated: January 5, 2021
sidebar: mydoc_sidebar
permalink: mydoc_kaynak_tuketimi.html
folder: mydoc
---

## Kaynak Tüketimi

### Memory

#### `shared_buffers`

{% include parameter_info.html parametre="shared_buffers" %}

{% include callout.html content=" **Veritabanı sunucusunun shared memory buffers için kullandığı bellek miktarını ayarlar.** Öntanımlı değeri 128 megabayttır (128MB). Bu ayar en az 128 kilobayt olabilir. İyi performans için minimumdan değerden daha yüksek ayarlar gereklidir. Bu değer birimsiz verilirse bloklar (BLCKSZ = 8kb) olarak alınır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

1 GB veya fazlası RAM'e sahip veritabanı sunucuları için sisteminizdeki belleğin %25'i makul bir `shared_buffers` başlangıç ​​değeridir. Daha büyük shared_buffers ayarlarının etkili olduğu bazı iş yükleri olabilir, ancak PostgreSQL aynı zamanda işletim sistemi önbelleğide kullandığı için RAM'in %40'ından fazlasının shared_buffers'a tahsis edilmesi daha küçük bir miktara kıyasla iyi bir performans göstermeyecektir." type="primary" %}

#### `huge_pages`

{% include parameter_info.html parametre="huge_pages" %}

{% include callout.html content=" **Huge page kullanımını kontrol eder.** Geçerli değerler, `try` (varsayılan), `on` ve `off` şeklindedir. huge_pages `try` ayarında sunucu huge page'ler istemeye çalışır, ancak başarısız olursa varsayılana geri döner. `on` ayarında huge page'lerin istenmemesi sunucunun başlamasını engelleyecektir. `off` ayarında huge page'ler istenmez." type="primary" %}

#### `temp_buffers`

{% include parameter_info.html parametre="temp_buffers" %}

{% include callout.html content=" **Her veritabanı oturumunda temporary buffers için kullanılan maksimum bellek miktarını ayarlar.** Bu alanlar, yalnızca geçici tablolara erişim için kullanılan session-local arabellekleridir. Öntanımlı değeri 8 megabayttır (8MB). Bu ayar, bireysel oturumlar içinde yalnızca oturumdaki geçici tabloların ilk kullanımından önce değiştirilebilir. Değeri değiştirmeye yönelik sonraki girişimlerin o oturum üzerinde hiçbir etkisi olmayacaktır." type="primary" %}

#### `max_prepared_transactions`

{% include parameter_info.html parametre="max_prepared_transactions" %}

{% include callout.html content=" **Aynı anda `prepared` statede olabilecek maksimum transaction sayısını ayarlar.** Parametrenin sıfıra (varsayılan değerdir) ayarlanması prepared-transaction özelliğini devre dışı bırakır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

Prepared transaction kullanmayı planlamıyorsanız yanlışlıkla oluşturulmasını önlemek için bu parametre sıfıra ayarlayın. Prepared transaction kullanırken `max_prepared_transactions` değerinin en az `max_connections` kadar olması önerilir. Böylece her oturum sırada bir prepared transactiona sahip olur.<br/><br/>

Standby sunucuda bu parametre primary sunucudakiyle aynı veya daha yüksek bir değere ayarlanmalıdır. Aksi takdirde standby sunucuda sorgulara izin verilmez." type="primary" %}

#### `work_mem`

{% include parameter_info.html parametre="work_mem" %}

{% include callout.html content=" **Sorgu operasyonları (ORDER BY, DISTINCT, merge joins, hash joins) tarafından kullanılacak maksimum bellek miktarını ayarlar.** Bu değer birimsiz olarak verilirse kilobayt olarak alınır. Öntanımlı değeri 4 megabayttır (4MB). Her operasyon, verileri geçici dosyalara yazmaya başlamadan önce bu değer kadar bellek kullanabilir. Bu operasyonlar çalışan birkaç oturum aynı anda olabileceğinden, total kullanılan bellek `work_mem` değerinin birçok katı olabilir. Değeri seçerken bu gerçeği göz önünde bulundurun. <br/><br/>

Hash-based operasyonlar kullanılabilir belleğe sort-based operasyonlara göre daha duyarlıdır. hash table'lar için kullanılabilir bellek `work_mem` ile `hash_mem_multiplier` çarpımından hesaplanır." type="primary" %}

#### `hash_mem_multiplier`

{% include parameter_info.html parametre="hash_mem_multiplier" %}

{% include callout.html content=" **Hash-based operasyonların kullanabileceği maksimum bellek miktarını hesaplamak için kullanılır.** Nihai sınır `work_mem`'in `hash_mem_multiplier` ile çarpılmasıyla belirlenir. Öntanımlı değer 1.0'dır." type="primary" %}

#### `maintenance_work_mem`

{% include parameter_info.html parametre="maintenance_work_mem" %}

{% include callout.html content=" **`VACUUM`, `CREATE INDEX` ve `ALTER TABLE ADD FOREIGN KEY` gibi bakım operasyonlarında kullanılacak maksimum bellek miktarını belirtir.** Bu değer birimsiz verildiğinde kilobayt olarak alınır. Öntanımlı değeri 64 megabayttır (64MB). Bir veritabanı oturumu tarafından aynı anda bu operasyonlardan yalnızca biri yürütülebilir ve normal kurulumda çoğu eşzamanlı olarak çalışmaz. Bu yüzden `work_mem`'den daha büyük ayar değeri güvenlidir. Daha büyük ayarlar vakumlama ve veritabanı dumplarından geri yükleme performansını artırır." type="primary" %}

#### `autovacuum_work_mem`

{% include parameter_info.html parametre="autovacuum_work_mem" %}

{% include callout.html content=" **Her bir autovacuum worker process tarafından kullanılacak maksimum bellek miktarını belirtir.** Bu değer birimsiz verildiğinde kilobayt olarak alınır. Öntanımlı değeri -1'dir ve bunun yerine `maintenance_work_mem` değerinin kullanılması gerektiğini belirtir." type="primary" %}

#### `logical_decoding_work_mem`

{% include parameter_info.html parametre="logical_decoding_work_mem" %}

{% include callout.html content=" **Logical decoding tarafından kullanılacak maksimum bellek miktarını belirtir.** Bu, logical streaming replication bağlantıları tarafından kullanılan bellek miktarını sınırlar. Öntanımlı olarak 64 megabayttır (64MB). Bu değeri `work_mem`'den önemli ölçüde daha yüksek değerde ayarlamak diske yazılan decoded değişikliklerin miktarı azaltır." type="primary" %}

#### `max_stack_depth`

{% include parameter_info.html parametre="max_stack_depth" %}

{% include callout.html content=" **Sunucunun kilobayt cinsinden maksimum execution stack derinliğini ayarlar.** Bu parametre için ideal ayar çekirdek tarafından zorlanan mevcut stack boyutu sınırıdır. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı ayar 2 megabayttır (2MB). Bu değer karmaşık fonksiyonların yürütülmesinde yeterli olmayabilir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `shared_memory_type`

{% include parameter_info.html parametre="shared_memory_type" %}

{% include callout.html content=" **Ana shared memory alanı için sunucunun kullanması gereken shared memory implementasyonunu belirtir.** Alabileceği değerler: `mmap` (mmap kullanılarak ayrılan anonymous shared memory için), `sysv` (shmget ile ayrılan System V shared memory için) ve `windows` (Windows shared memory için)." type="primary" %}

#### `dynamic_shared_memory_type`

{% include parameter_info.html parametre="dynamic_shared_memory_type" %}

{% include callout.html content=" **Sunucunun kullanacağı dynamic shared memory implementasyonunu belirtir.** Alabileceği değerler: `posix` (shm_open kullanılarak ayrılan POSIX shared memory için), `sysv` (shmget aracılığıyla ayrılan System V shared memory için), `windows` (Windows shared memory için) ve `mmap` (veri dizinindeki memory-mapped dosyaları kullanarak shared memory'i simüle etmek için). `mmap` seçeneğinin kullanılması genellikle tavsiye edilmez." type="primary" %}

### Disk

#### `temp_file_limit`

{% include parameter_info.html parametre="temp_file_limit" %}

{% include callout.html content=" **Her işlem tarafından kullanılan total geçici dosyaların ( sort ve hash temporary files gibi) toplam boyutunu sınırlar.** Bu sınırı aşmaya çalışan bir transaction iptal edilir. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı -1 değeri sınır yok demektir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu ayarın, belirli bir PostgreSQL sürecinin kullandığı tüm geçici dosyaların herhangi bir anda kullanılan toplam alanı sınırladığına dikkat edin." type="primary" %}

### Çekirdek Kaynak Kullanımı

#### `max_files_per_process`

{% include parameter_info.html parametre="max_files_per_process" %}

{% include callout.html content=" **Her sunucu süreci için eşzamanlı olarak açılabilecek maksimum dosya sayısını ayarlar.** Varsayılan 1000 dosyadır. 'Too many open files' hatası ile karşılaşırsanız bu ayarı düşürmeyi deneyin. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

### Cost-based Vacuum Delay

`VACUUM` ve `ANALYZE` komutları yürütülürken sistem, gerçekleşen çeşitli I / O operasyonlarının tahmini maliyetini izleyen dahili bir sayaç tutar. Birikmiş maliyet `vacuum_cost_limit` ile belirtilen limite ulaştığında operasyonu gerçekleştiren süreç `vacuum_cost_delay` ile belirtilen süre boyunca uyutulur. Ardından sayacı sıfırlanarak çalışmaya devam eder.

Bu özelliğin amacı yöneticilere, bu komutların eşzamanlı veritabanı etkinliği üzerindeki I / O etkisini azaltmalarına olanak sağlamaktır. VACUUM ve ANALYZE gibi bakım komutlarının sistemin diğer veritabanı operasyonlarını gerçekleştirme becerisine fazla müdahale etmemesi çok önemlidir. Maliyete dayalı vakum gecikmesi, yöneticilerin bunu başarması için bir yol sunar.

Bu özellik, manuel olarak verilen VACUUM komutları için öntanımlı olarak devre dışı bırakılmıştır. Bunu etkinleştirmek isterseniz `vacum_cost_delay` değişkenini sıfır olmayan bir değere ayarlayın.

#### `vacuum_cost_delay`

{% include parameter_info.html parametre="vacuum_cost_delay" %}

{% include callout.html content=" **Maliyet sınırı aşıldığında sürecin uyuyacağı süre (Vacuum cost delay).** Bu değer birimsiz verilirse milisaniye olarak alınır. Öntanımlı değer sıfırdır ve maliyete dayalı vakum gecikmesi özelliğini devre dışı bırakır. Pozitif değerler maliyete dayalı vakumlamaya izin verir.Maliyete dayalı vacuum kullanılırken `vacuum_cost_delay` için uygun değerler genellikle 1 milisaniye ile ifade edecek kadar çok küçüktür." type="primary" %}

#### `vacuum_cost_page_hit`

{% include parameter_info.html parametre="vacuum_cost_page_hit" %}

{% include callout.html content=" **Shared buffer cache'de bulunan bir buffer'ı vakumlamanın tahmini maliyeti.** Öntanımlı değer 1'dir. " type="primary" %}

#### `vacuum_cost_page_miss`

{% include parameter_info.html parametre="vacuum_cost_page_miss" %}

{% include callout.html content=" **Diskten okunması gereken bir buffer'ı vakumlamanın tahmini maliyeti.** Öntanımlı değer 10'dur." type="primary" %}

#### `vacuum_cost_page_dirty`

{% include parameter_info.html parametre="vacuum_cost_page_dirty" %}

{% include callout.html content=" **Vakum önceden temiz olan bir bloğu değiştirdiğinde alınan tahmini maliyet.** Dirty bloğu tekrar diske geçirmek için gereken ekstra I / O'u temsil eder. Öntanımlı değer 20'dir." type="primary" %}

#### `vacuum_cost_limit`

{% include parameter_info.html parametre="vacuum_cost_limit" %}

{% include callout.html content=" **Vakum sürecinin uyumasına neden olacak birikmiş maliyet.** Öntanımlı değer 200'dür." type="primary" %}

{% include note.html content=" Kritik kilitleri tutan ve olabildiğince çabuk tamamlanması gereken operasyonlarda maliyete dayalı vakum gecikmeleri yaşanmaz. Bu nedenle maliyet belirlenen limitin çok üzerinde birikebilir. Bu gibi durumlarda gereksiz derecede uzun gecikmelerden kaçınmak için mevcut gecikme **vacuum_cost_delay * accumulated_balance / vacuum_cost_limit with a maximum of vacuum_cost_delay * 4** formülü ile hesaplanır." %}

### Background Writer

Background Writer ayrı bir PostgreSQL sunucu sürecidir. Background Writer "dirty", yani yeni veya değiştirilmiş shared buffer'ları diske yazmakla görevlidir. Böylece, sunucu süreçlerinin kullanıcı sorgularını yazması nadiren beklenir veya hiçbir zaman beklenmez. Background Writer I / O yükünde net bir şekilde artışa neden olur, çünkü tekrar tekrar dirty olan bir page her checkpoint aralığında yalnızca bir kez yazılırken Background Writer tarafından birkaç kez yazabilir. Verilen parametreleri davranışları ihtiyaçlarınıza göre ayarlamak için kullanabilirsiniz.

#### `bgwriter_delay`

{% include parameter_info.html parametre="bgwriter_delay" %}

{% include callout.html content=" **Background Writer'ın aktif periyodları arasındaki süreyi belirtir.** Background Writer her turda aşağıdaki parametreler ile kontrol edilen sayıdaki dirty buffer'ları diske işlerek `bgwriter_delay` boyunca uyur. Buffer pool'da dirty buffer olmadığında bgwriter_delay'e bakmaksızın daha uzun süre uykuda kalabilir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Öntanımlı değeri 200 milisaniyedir (200 ms). Birçok sistemde etkili değer 10 milisaniye olmakla bereaber bgwriter_delay'i 10'un katı olmayan bir değere ayarlamak, 10'un bir sonraki katına ayarlamakla aynı sonuçları verebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `bgwriter_lru_maxpages`

{% include parameter_info.html parametre="bgwriter_lru_maxpages" %}

{% include callout.html content=" **Her turda, Background Writer tarafından `bgwriter_lru_maxpages` değerinde buffer'dan fazlası yazılmaz.** Bunu sıfıra ayarlamak, background writing'i devre dışı bırakır (checkpoints etkilenmez). Öntanımlı değeri 100 buffer'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `bgwriter_lru_multiplier`

{% include parameter_info.html parametre="bgwriter_lru_multiplier" %}

{% include callout.html content=" Her turda yazılan dirty buffer sayısı, son turlarda sunucu işlemleri tarafından ihtiyaç duyulan yeni buffer sayısına bağlıdır. Bir sonraki turda ihtiyaç duyulacak tahmini buffer sayısı, yakın zamandaki ortalama ihtiyacın `bgwriter_lru_multiplier` değeri ile çarpımından bulunur. dirty buffer'lar çok sayıda temiz, yeniden kullanılabilir buffer bulunana kadar yazılır. 1.0 ayarı 'just in time' ilkesini yani tam olarak ihtiyaç duyulan tahmini buffer sayısını yazmayı temsil eder. Daha büyük değerler talepteki ani artışlara karşı bir miktar buffer sağlar. Küçük değerler kasıtlı olarak yazma işlemini sunucu süreçlerine bırakır. Öntanımlı değeri 2.0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `bgwriter_flush_after`

{% include parameter_info.html parametre="bgwriter_flush_after" %}

{% include callout.html content=" **Background Writer tarafından bu miktardan daha fazla veri yazıldığında, işletim sistemini bu verileri esas depolama alınan yazmaya zorlar.** Bununla, çekirdeğin page cache'indeki dirty veri miktarını sınırlandırılır ve bir checkpoint sonunda `fsync` yayınlandığında veya işletim sistemi verileri daha büyük gruplar halinde arka planda geri yazdığında durma olasılığını düşürülür. Bu ayar bazı platformlarda hiç bir etkiye sahip olmayabilir. Bu değer birimsiz belirtilirse, blok yani `BLCKSZ` bayt (tipik olarak 8kB'dir) olarak alınır. Geçerli aralık, zorunlu geri yazmayı devre dışı bırakan 0 ile 2MB arasındadır. Linux'ta öntanımlı değer 512kB, diğer sistemlerde 0'dır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### Asynchronous Davranış

#### `effective_io_concurrency`

{% include parameter_info.html parametre="effective_io_concurrency" %}

{% include callout.html content=" **Disk alt sistemi tarafından verimli bir şekilde işlenebilen eşzamanlı istek sayısı.** Bu değerin yükseltilmesi, herhangi bir PostgreSQL sessionının paralel olarak başlatmaya çalıştığı I / O operasyonlarının sayısını artırır. İzin verilen aralık 1 ile 1000 arasıdır. 0 asynchronous I / O isteklerini devre dışı bırakır. Şu an için bu ayar sadece bitmap heap taramalarını etkilemektedir. Varsayılan, desteklenen sistemlerde 1, diğerlerinde 0'dır." type="primary" %}

#### `maintenance_io_concurrency`

{% include parameter_info.html parametre="maintenance_io_concurrency" %}

{% include callout.html content=" **İstemci oturumları adına yapılan bakım çalışmaları için kullanılır.** Desteklenen sistemlerde öntanımlı değeri 10'dur, diğerlerinde 0'dır." type="primary" %}

#### `max_worker_processes`

{% include parameter_info.html parametre="max_worker_processes" %}

{% include callout.html content="**Sistemin destekleyebileceği maksimum background süreç sayısını ayarlar.** Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Öntanımlı değeri 8'dir. Standby sunucu çalıştırırken, bu parametre primary sunucudakiyle aynı veya daha yüksek bir değere ayarlanmalıdır. Aksi takdirde, standby sunucuda sorgulara izin verilmeyecektir.<br/><br/>

Bu değer değiştirilirken `max_parallel_workers`, `max_parallel_maintenance_workers` ve `max_parallel_workers_per_gather` parametrelerinin ayarlanması düşünülebilir." type="primary" %}

#### `max_parallel_workers_per_gather`

{% include parameter_info.html parametre="max_parallel_workers_per_gather" %}

{% include callout.html content=" **Yürütme düğümü başına maksimum paralel işlem sayısını ayarlar.** Tek bir `Gather` veya `Gather Merge` düğümü tarafından başlatılabilecek maksimum worker sayısını ayarlar. Paralel worker'lar `max_worker_processes` tarafından oluşturulmuş, `max_parallel_workers` ile sınırlandırılmış süreç havuzundan alınır. İstenen worker sayısı çalışma zamanında mevcut olmadığında, plan beklenenden daha az sayıda worker ile çalışarak verimsiz olabilir. Öntanımlı değeri 2'dir. Bu değerin 0 olarak ayarlanması paralel sorgu yürütmeyi devre dışı bırakır.<br/><br/>

Paralel sorgular, paralel olmayan sorgularadan çok daha fazla kaynak tüketebilir. Çünkü her worker süreci sistem üzerinde ek bir kullanıcı oturumuyla hemen hemen aynı etkiye sahip olan tamamen ayrı bir süreçtir. `work_mem` gibi kaynak limitleri her bir worker için ayrı ayrı uygulanır. Paralel sorgu hakkında daha fazla bilgi için bkz [](https://www.postgresql.org/docs/current/parallel-query.html)." type="primary" %}

#### `max_parallel_maintenance_workers`

{% include parameter_info.html parametre="max_parallel_maintenance_workers" %}

{% include callout.html content=" **Tek bir utility program komutuyla başlatılabilen maksimum paralel worker sayısını ayarlar.** Şu an için paralel worker kullanımını `CREATE INDEX` (B-tree indekslerde) ve `VACUUM` (FULL olmadan) işlemlerinde desteklenmektedir. Paralel workers `max_worker_processes` tarafından oluşturulmuş, `max_parallel_workers` ile sınırlandırılmış süreç havuzundan alınır. İstenen worker sayısı çalışma zamanında mevcut olamdığında utility program operasyonu beklenenden daha az sayıda worker ile çalışacaktır. Öntanımlı değeri 2'dir. Bu değerin 0 olarak ayarlanması utility program komutlarının paralel worker kullanmasını devre dışı bırakır." type="primary" %}

#### `max_parallel_workers`

{% include parameter_info.html parametre="max_parallel_workers" %}

{% include callout.html content=" **Paralel operasyonlar için sistemin destekleyebileceği maksimum worker sayısını ayarlar.** Varsayılan değeri 8'dir. Bu değeri artırırken veya azaltırken `max_parallel_maintenance_workers` ve `max_parallel_workers_per_gather` parametrelerini de ayarlamayı düşünün. Ayrıca, bu parametre değerinin `max_worker_processes`'ten daha yüksek olan bir ayarı, paralel worker'lar bu ayar tarafından oluşturulan havuzundan alındığı için hiçbir etkisi olmayacağını unutmayın." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-resource.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
