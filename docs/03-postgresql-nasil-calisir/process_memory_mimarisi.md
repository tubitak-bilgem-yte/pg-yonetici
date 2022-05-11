---
title: "Process ve Memory Mimarisi"
layout: default
parent: PostgreSQL Nasıl Çalışır?
nav_order: 3
---


## Process Mimarisi

PostgreSQL, multi-process mimarisine sahip istemci/sunucu tipi ilişkisel veritabanı yönetim sistemidir ve tek bir hostta çalışır.

Bir database cluster'ı, birden çok processle birlikte yöneten yapı `PostgreSQL server` olarak adlandırılır ve aşağıdaki process'leri içerir:

{% include callout.html content=" **`Postgres server process`**: Veritabanı cluster yönetimiyle ilgili tüm process'lerin fork edildiği ana process'dir." type="primary" %}

{% include callout.html content=" **`Backend process`**: Bağlı istemci tarafından gönderilen sorguları işler. Bu process, postgres olarak da isimlendirilir." type="primary" %}

{% include callout.html content=" **`Background process`**: VACUUM, CHECKPOINT gibi veritabanı yönetim özelliklerini  gerçekleştirilir." type="primary" %}

{% include callout.html content=" **`Replication associated processes`**: Streaming replikasyon'u gerçekleştirir." type="primary" %}

{% include callout.html content=" **`Background worker process`**: kullanıcılar tarafından sağlanan herhangi bir process'i gerçekleştirir. Böylece postgres kullanıcı tarafından sağlanan bir kodu çalıştırmak üzere genişletilebilir." type="primary" %}

Aşağıda ilk üç process türü ayrıntılı şekilde ele alınmıştır.

{% include image.html file="process_memory_mimarisi-1.png" alt="https://www.interdb.jp/pg/img/fig-2-01.png" caption=" Şekil 1. PostgreSQL süreç mimarisi örneği. [https://www.interdb.jp/pg/img/fig-2-01.png]" %}

### Postgres Server Process

Postgres Server Process, PostgreSQL sunucusundaki tüm processlerin parent process'idir. Önceki sürümlerde `postmaster` olarak adlandırılırdı.

[pg_ctl](https://www.postgresql.org/docs/9.5/app-pg-ctl.html) yardımcı programının `start` opsiyonuyla postgres server process  başlatılır. Bu process'e bellekte **`shared memory area`** isminde bir alan tahsis edilir ve burada çeşitli background process'ler başlatır, duruma göre replication associated processes ve background worker process'lerini de başlatarak istemcilerden bağlantı isteklerini bekler. Tam çalışır durumdaki veritabanı, artık istemcilerden gelen her bir bağlantı isteği için bir backend process başlatarak gönderilen sorguları işler.

{% include note.html content=" Postgres server process aynı anda yalnızca bir portu dinler. Varsayılan 5432'dir. Dinlenen port numaraları farklı olmak şartıyla aynı hostta birden çok PostgreSQL veritabanı çalıştırmak mümkündür." %}

### Backend Processes

Backend process, postgres server process tarafından başlatılır ve bağlı istemciden gelen sorguları işler. `postgres` olarakta adlandırılır. İstemciyle tek TCP bağlantısı üzerinden iletişim kurar ve istemci bağlantısı kesildiğinde process sonlandırılır. Backend Processes aynı anda yalnızca bir veritabanı üzerinde operasyonlar yapabilir. Bu yüzden PostgreSQL sunucusuna bağlanırken kullanmak istediğiniz veritabanını açıkça belirtmeniz gerekir. Postgres sunucusuna aynı anda birden fazla istemci bağlanabilir. Bağlanacak maksimum istemci sayısı [max_connections](mydoc_baglantilar_kimlik_dogrulama.html) parametresiyle kontrol edilir.Varsayılan değer 100'dür.

PostgreSQL'in dahili bağlantı havuzu (connection pooling) oluşturma özelliği yoktur. Bu sebeple, web uygulamaları gibi istemcinin sık sık PostgreSQL sunucusuyla bağlantı kurma / kesme yaptığında hem yeni bağlantı hem de backend process oluşturma maliyeti artar. Bu veritabanı performansı üzerinde olumsuz bir etkiye sahiptir. Bu tip durumlarda [pgbouncer](mydoc_pgbouncer.html) ve [pgpool-II](mydoc_pgpool.html) gibi connection pooling yazılımlarının kullanımı önerilir.

### Background Processes

Background Process'lerin her biri kendine özel fonksiyonlar içerdiği ve PostgreSQL iç bileşenlerine bağlı olduğudan postgres server ve backend process gibi basitçe açıklamak pek mümkün değildir. Bu kısımda konuyu karmaşıklaştırmamak adına sadece görevleri açıklayıplanmıştır.

|-------|--------|
| **`background writer`** | Shared buffer pool'daki dirty pageleri belirli aralıklarla kalıcı bir depolama alanına (HDD, SSD) yazar. |
| **`checkpointer`** | Her *checkpoint_timeout* periodunda ve *max_wal_size* parametresi aşıldığında checkpoint işlemini gerçekleştirir. |
| **`autovacuum launcher`** | Vacuum işleminin yürütülmesini otomatikleştirir. |
| **`WAL writer`** | WAL buffer'daki verileri düzenli olarak kalıcı depolama alanına yazar. |
| **`stats collector`** | İstatistik bilgileri toplar. Örneğin *pg_stat_activity* ve *pg_stat_database* için. |
| **`logging collector (logger)`** | Hata mesajlarını log dosyalarına yazar. |
| **`archiver`** | Archive.log modundayken WAL dosyasını belirtilen dizine kopyalar. |

{% include image.html file="process_memory_mimarisi-3.png" alt="PostgreSQL sunucusu işlemleri" caption=" Şekil 2. PostgreSQL sunucusu işlemleri. PostgreSQL veritabanı bir postgres server process (pid 9687), iki backend process (pid 9697, 9717) ve listelenen birkaç background process ile çalışmaktadır." %}

## Memory Mimarisi

PostgreSQL'deki memory mimarisi Local ve Shared memory olarak iki temel bileşenden oluşur:

{% include callout.html content=" **`Local Memory (Yerel Bellek)`**: Her backend process tarafından kendi kullanımı için tahsis edilmiş alandır." type="primary" %}

{% include callout.html content=" **`Shared Memory (Paylaşımlı Bellek)`**: PostgreSQL sunucusundaki tüm process'lerin kullanımı için ayrılan ortak alandır." type="primary" %}

{% include image.html file="process_memory_mimarisi-2.png" alt="https://www.interdb.jp/pg/img/fig-2-02.png" caption=" Şekil 3. PostgreSQL memmory mimarisi örneği. [https://www.interdb.jp/pg/img/fig-2-02.png]" %}

### Local Memory Alanı

Her bir backend process'e sorgu işlemesi için local memory alanları tahsis edilir. Bu alanlar work_mem,  temp_buffers, maintenance_work_mem gibi boyutları sabit veya değişken olan alt alanlara ayrılır.

- **`work_mem`**: Executor tarafından ORDER BY, DISTINCT gibi sıralama ve joinleme işlemlerinde kullanılan alandır.
- **`maintenance_work_mem`**: VACUUM, REINDEX gibi bakım işlemleri için kullanılan alandır.
- **`temp_buffers`**: Executor bu alanı geçici tabloları depolamak için kullanır.

### Shared Memory Alanı

PostgreSQL başlatıldığında veritabanı tarafından shared memory alanı tahsis edilir. Bu alan kendi içinde shared buffer pool, WAL buffer, commit log gibi sabit boyutlu alt alanlara ayrılır.

- **`shared buffer pool`**: Daha hızlı okuma yazma işlemleri için verinin tutulduğu bellek alanıdır. Tablo ve indeksler içindeki page'ler, diskten bu alana yüklenerek doğrudan çalıştırılır.
- **`WAL buffer`**: PostgreSQL WAL mekanizmasını çeşitli nedenlerden doğacak veri kaybını engellemek için kullanır. WAL verileri PostgreSQL'deki transaction loglarıdır. WAL buffer, kalıcı bir depolama alanına yazmadan önce WAL verilerinin yani veritabanındaki değişikliklerin geçici olarak tutulduğu alandır.
- **`commit log`**: Commit Log (CLOG), Concurrency Control mekanizması için tüm transactionların `in_progress`, `committed`, `aborted` gibi durumlarını tutar.

**Kaynak**:

[1]. [The Internals of PostgreSQL](https://www.interdb.jp/pg/pgsql02.html)

[2]. [Severalnines: Understanding the PostgreSQL Architecture](https://severalnines.com/database-blog/understanding-postgresql-architecture)

[3]. [wikibooks.org: PostgreSQL/Architecture](https://en.wikibooks.org/wiki/PostgreSQL/Architecture)

{% include links.html %}
