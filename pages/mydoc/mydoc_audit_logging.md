---
title: "PostgreSQL Audit Logging"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 25, 2020
sidebar: mydoc_sidebar
permalink: mydoc_audit_logging.html
folder: mydoc
---

## Audit Logging

### Neden Gereklidir ?

PostgreSQL kullanılan kuruma bağlı olarak, özellikle kurumlar büyüdükçe ve de kanuni olarak takip edilmesi gereken yerler olduğunda ( SPK'nın kontrolünde olan kurumlar gibi ) audit bir zorunluluk halini almaktadır. En temel haliyle, hangi kullanıcının ne zaman ve hangi IP adresinden giriş yaptıkları önemli bir bilgi olmaktadır. Benzer şekilde, veritabanındaki nesnelerin kimler tarafından yaratıldığı, kaldırıldığı gibi şeyler de çok önemli olmaktadır.

Bir başka açıdan da DBA'lerin veritabanından veri toplaması için de audit gereklidir.

PostgreSQL'in loglama özellikleri son birkaç yılda gelişmiştir. Ancak ne yazık ki bu özellikler bile her gereksinimi karşılayamamaktadır. Artan loglama ihtiyaçlarınız için pgaudit eklentisi de alt bölümde ele alınmıştır.

### Audit Parametreleri

Bu başlık altında, yorumsuz olarak önemli parametreleri incelenmiştir. Bir sonraki bölümde de hangi parametrenin neye ayarlanması gerektiğini belirtilecektir.

{% include callout.html content="**`log_destination`**: PostgreSQL'in hangi yöntemle log göndereceğini belirtir. `stderr` ile mesajların standart error'a basılmasını, `csvlog` ile logların csv formatında tutulmasını, `syslog` ile logların `syslog` deamon üzerinden gönderilmesini ve `eventlog` ile Windows'daki event logger'ın kullanılmasını sağlayabilirsiniz. Bu parametrenin geçerli olması için `logging_collector` parametresinin etkin olması gerekir. RPM kurulumlarında bu parametre `stderr` olarak ayarlanır." type="primary" %}

{% include callout.html content="**`logging_collector`**: Eğer `log_destination` parametresi `stderr` ya da `csvlog` olarak ayarlanmışsa, postmaster bunları yakalamak için yeni bir alt süreç başlatır. Bu ayarda yapılan değişikliğin etkin olması için PostgreSQL'in yeniden başlatılması gereklidir. ps çıktısında logger process olarak görünen süreci bu parametre başlatır." type="primary" %}

{% include callout.html content="**`log_directory`**: `logging_collector` açık ise logların tutulacağı dizini verir. $PGDATA'nın altındaki bir dizini ya da mutlak bir dizini gösterebilir. Örneğin, eğer bu parametre `pg_log` ise $PGDATA'nın altındaki *pg_log* dizininde loglar tutulur. Ayrı bir yerde tutmak isterseniz */var/log/postgresql* gibi bir dizin verebilirsiniz. Bu parametre birçok kurulumda `pg_log` olarak ayarlanır." type="primary" %}

{% include callout.html content="**`log_filename`**: Log dosyasının adını bu parametre ile belirleyebilirsiniz. Burada kullanacağınız tüm parametreleri `date` komutunun man sayfasına bakarak alabilirsiniz. Örneğin, `'postgresql-%a.log'` ile logların *postgresql-Mon.log*, *postgresql-Tue.log* ... gibi tutulmasını sağlayabilirsiniz." type="primary" %}

{% include callout.html content="**`log_truncate_on_rotationg`**: Eğer bu parametre etkinse, üstte belirtilen `log_filename` ayarındaki dosyanın üzerine yazılacağı zaman dosyanın içeriği temizlenir. Genelde bu parametrenin açılması önerilir. Öntanımlı olarak bu parametre kapalıdır (RPM kurulumlarında açılır)" type="primary" %}

{% include callout.html content="**`log_rotation_age`**: Log dosyalarının belirtilen süre sonunda rotate edilmesini sağlar. Öntanımlı değeri `1d` (1 day, 1 gün)'dir. " type="primary" %}

{% include callout.html content="**`log_rotation_size`**: Log dosyaları bu parametrede belirtilen değere gelince otomatik olarak rotate edilir. " type="primary" %}

{% include callout.html content="**`client_min_messages`**: İstemcilere gönderilecek mesajların seviyesini bu parametre ile belirtebilirsiniz. Bu parametre `error` ise sadece hata mesajları gönderilir. En üst seviyedeki `debug5` ise en fazla ayrıntıyı verir.Bu parametreye debug5, debug4, debug3, debug2, debug1, log, notice, warning ya da error değerlerinden birisi verilebilir." type="primary" %}

{% include callout.html content="**`log_min_messages`**: Log dosyasına gönderilecek mesajların miktarı belirtilir. Bu parametre `error` ise sadece panic mesajları gönderilir. En üst seviyedeki `debug5` ise en fazla ayrıntıyı verir. Bu parametreye debug5, debug4, debug3, debug2, debug1, info, notice, warning, error, log, fatal ya da panic değerlerinden birisi verilebilir." type="primary" %}

{% include callout.html content="**`log_min_error_statemen`**: Hataya neden olan SQL mesajının loglanıp loglanmamasını bu parametre ile belirlenir. Geçerli değerler: debug5, debug4 ,debug3, debug2, debug1, info, notice, warning, error, log, fatal ya da panic'dir. Öntanımlı değeri `error`' dır. Bu ayar sadece superuser tarafından değiştirilebilir." type="primary" %}

{% include callout.html content="**`log_min_duration_statemen`**: milisaniye cinsinden belirtilen bu parametre ile, belirtilen süreye eşit ya da belirtilen süreden daha fazla süren sorguların loglanmasını sağlayabilirsiniz. Eğer bu değeri 0 yaparsanız tüm sorguları loglayabilirsiniz. Bu değeri -1 yaparsanız bu işlemi kapatırsınız. Bu parametre `log_statement` ile çakışmaz." type="primary" %}

{% include callout.html content="**`log_checkpoints, log_connections, log_disconnections, log_duration, log_hostname:`**: Sırası ile checkpointler, yapılan bağlantılar, kapatılan bağlantılar, sorgu süresi ve bağlantı yapan sunucunun adresi loglanır. Öntanımlı olarak hepsi kapalıdır. " type="primary" %}

{% include callout.html content="**`log_error_verbosity`**: Öntanımlı değeri `DEFAULT` olan bu parametre ile, loglanan mesajların ayrıntı miktarını ayarlayabilirsiniz. Terse modunda CONTEXT, HINT, DETAIL ve QUERY hata bilgileri verilmez. VERBOSE modunda ise SQLSTATE hata mesajları, kaynak kod dosya adı, fonksiyonadı ve hatanın gerçekleştiği kodun satır numarası da loglara yazılır.Geliştiriciler için daha uygun bir seçenektir. Sadece superuserlar tarafından değiştirilebilir." type="primary" %}

{% include callout.html content="**`log_line_prefix`**: Log dosyasının her bir satırının başına eklenecek önekibu parametre ile ayarlayabilirsiniz. Olası değerler şunlardır: " type="primary" %}

|-------|--------|
| `%a` | = application name İstemci uygulamalarından gelen uygulama adı |
| `%u` | = user name : Kullanıcı adı |
| `%d` | = database name : Veritabanı adı |
| `%r` | = remote host ve port : Veritabanı sunucusuna bağlanan istemcinin adresi ve portu |
| `%h` | = remote host : Veritabanı sunucusuna bağlanan istemcinin adresi |
| `%p` | = process ID : İlgili sürecin (process) numarası |
| `%t` | = timestamp without milliseconds : milisaniye olmadan timestamp değeri |
| `%m` | = timestamp with milliseconds: milisaniyeyi de içeren timestamp değeri |
| `%i` | = command tag : Komut etiketi (Örnek: idle, authentication, SQL tümceleri vs) |
| `%e` | = SQL state : SQL state id |
| `%c` | = session ID : Oturum ID'si |
| `%l` | = session line number : O oturumdaki satır no |
| `%s` | = session start timestamp : O oturumun başlama zamanı |
| `%v` | = virtual transaction ID : Virtual Xid |
| `%x` | = transaction ID (0 if none) : txid |

{% include callout.html content="**`log_statemen`**: Hangi SQL ifadelerinin loglananacağını belirten parametredir. Öntanımlı değeri olan `none`'da hiçbir ifade loglanmaz. Bu değer `ddl` olursa o zaman DDL ifadeleri ( CREATE / DROP / ALTER ) ifadeleri loglanır. Eğer bu değer `mod`  olursa, ddl ifadelerinin yanısıra INSERT, UPDATE, COPY, EXPLAIN ANALYZE, PREPARE / EXECUTE ve TRUNCATE gibi ifadeler de loglanır. Sadece superuser bu ayarı değiştirebilir." type="primary" %}

{% include callout.html content="**`log_lock_waits`**: Eğer bir sorgu deadlock_timeout değerinden daha fazlasüre  lock almak için beklerse, o zaman log dosyasına bu durum ile ilgili birmesaj düşülür. Örnek bir log mesajı: " type="primary" %}

```sql
LOG: process 32603 still waiting for ShareLock on transaction 3284after 1000.125 ms
STATEMENT: UPDATE film SET fulltext ='NewFullText';
```

Eğer bu transaction bir süre sonra lock alırsa:

```sql
LOG:   process 32603 acquired ShareLock on transaction 3284 after98925.356 ms
STATEMENT:  UPDATE film SET fulltext ='NewFullText';
```

gibi bir mesaj loglanır.

{% include callout.html content="**`log_temp_files`**: Belirtilen boyuta eşit ya da ondan büyük geçici dosya yaratıldığında, bu bilginin loglanmasını sağlayan parametredir. `-1` bu seçeneği kapatır (öntanımlı seçenek budur). `0` olursa da tüm geçicidosyaları loglar. `work_mem` değerinin yetersiz olduğu durumları yakalamak için iyi bir seçenektir." type="primary" %}

{% include callout.html content="**`log_timezone`**: Loga yazılan timestamp bilgilerinin zaman dilimi (timezone) bilgisini ayarlar. Bu ayarda yapılan değişikliğin etkin olması için PostgreSQL'in yeniden başlatılması gereklidir." type="primary" %}

### Güvenlik / İzleme için Değiştirilmesi Önerilen Parametreler

{% include callout.html content="**`log_connections, log_disconnections`**: Bu iki parametreyi de `on` yaparsanız veritabanı sunucunuza bağlanan ve bağlantısını kesenleri ayrı ayrı loglarsınız. Ancak, burada küçük bir noktaya dikkat çekmek istiyoruz: Yoğun sunucularda bu loglamanın verdiği yük fazla olabilir. Yine de audit gereksinimi olan her yerde açılması gerektiğinden buna uygun bir önlem (log dosyalarının ayrı bir diske taşınması gibi) alınmalıdır " type="primary" %}

{% include callout.html content="**`log_statement`**: tam bir audit için bu parametrenin `all` olarak ayarlamansı önerilir . Aşağıda örnek çıktıyı görebilirsiniz:" type="primary" %}

```log
<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:26:50.331   BST   >   LOG:      statement:   INSERT   INTO   il   VALUES('35','İzmir','1');

<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:26:50.333 BST > LOG:  duration: 2.067 ms

<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:26:54.480 BST > LOG:  statement: SELECT * from il limit 1;

<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:26:54.480 BST > LOG:  duration: 0.298 ms

<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:27:02.763 BST > LOG:  statement: UPDATE il SET version='2';

<   user=postgres   db=bilgemyte   host=[local]   pid=27245   time=2020-05-1312:27:02.766 BST > LOG:  duration: 3.151 ms
```

### DBA'ler için Değiştirilmesini Önerilen Parametreler

DBA'lerin bilgi toplaması için aşağıdaki parametrelerin değiştirilmesini önerilir. Burada "değiştirilmek" sözcüğünden kasıt, RPM kurulumlarında öntanımlı olarak gelen değerlerin değiştirilmesidir. Başka dağıtımlar ya da kaynak kod kurulumları farklı öntanımlı değerlerle geleceklerdir:

{% include callout.html content="**`log_min_duration_statement`**: Burada kararı size bırakmakla beraber, başlangıçta 1000 (ms), sonra 750, 500, 400,300,200,100 ve 50 değerlerini aşama aşama uygulayıp yavaş sorguları görmek ve sorunu çözmek için uygulayabilirsiniz." type="primary" %}

{% include callout.html content="**`log_checkpoints`**: Bu değeri mutlaka `on` yapınız. Her checkpoint sonunda pgbadger (mevcutsa) tarafından analiz edilecek değerli bilgiler loglanacaktır." type="primary" %}

{% include callout.html content="**`log_line_prefix`**: Önerilen bir değer: `'< user=%u db=%d host=%h pid=%p app=%a time=%m > '`" type="primary" %}

{% include callout.html content="**`log_lock_waits`**: Tüm kurulumlarda `on` yapınız." type="primary" %}

{% include callout.html content="**`log_temp_files`**: Tüm kurulumlarda 0 yapınız ( tüm dosyaları loglama seçeneği )" type="primary" %}

### Daha Fazla Audit: pgaudit

PostgreSQL'in üstte yazdığımız parametre ve bilgilerden daha fazlasına gereksinim duyduğunuz durumlarda `pgaudit` eklentisini kullanabilirsiniz. Pgaudit ücretsiz ve açık kaynak kodludur.

pgaudit, PostgreSQL'in içindeki audit yeteneklerini audit yapacak kuruma daha uygun veri verecek şekilde genişletir. Loglamayı PostgreSQL'in kendi loglama altyapısını kullanarak yapacağından ayrı bir logger deamon çalışmayacaktır.

Pgaudit'in PostgreSQL YUM deposunda paketi bulunmaktadır. `yum -yinstall pgaudit_96` komutu ile kurabilirsiniz. Paketi kurduktan sonra *postgresql.conf* içindeki *shared_preload_libraries* parametresine `pgaudit` değerini eklemeniz gerekir. Örnek:

```text
shared_preload_libraries='pg_stat_statements, pgaudit'
```

Ayrıca, aşağıdaki parametreleri de *postgresql.conf*'un en altına yazabilirsiniz:

```text
pgaudit.log = 'all'
pgaudit.log_parameter='on'
pgaudit.log_relation='on'
```

Bu parametrelerin açıklamaları:

**`pgaudit.log`**: Hangi işlemlerin loglanacağını belirtir. Burada şu seçenekleriniz bulunmaktadır:

|-------|--------|
| `READ` | SELECT ve COPY işlemleri loglanır. |
| `WRITE` | INSERT, UPDATE, DELETE, TRUNCATE ve COPY işlemleri loglanır. |
| `FUNCTION` | DO blokları ve fonksiyon çağrıları loglanır. |
| `ROLE` | GRANT, REVOKE, CREATE/ALTER/DROP ROLE işlemleri loglanır. |
| `DDL` | Üstteki ROLE içeriğinde olmayan tüm DDL'ler loglanır. |
| `MISC` | DISCARD, FETCH, CHECKPOINT, VACUUM gibi diğer komutlar loglanır. |

Önerilen `ALL` seçeneği hepsini loglar. Burada birden fazla seçenek de belirtebilirsiniz:

- **`pgaudit.log = 'ALL, -READ'`** ==>  READ işlemleri dışındakileri loglar.
- **`pgaudit.log = 'READ, WRITE'`** ==> Sadece READ ve WRITE işlemlerini audit eder.

**`pgaudit.log_parameter`**: SQL ifadesinde verilen parametrelerin de loglanıp loglanmayacağını belirtir. Öntanımlı olarak kapalı gelir.
**`pgaudit.log_relation`**: SELECT ya da DML ifadelerinde referans verilen her bir "relation" (TABLE, VIEW, vs) için ayrı bir log satırı eklenmesini sağlar.

Örnek bir log çıktısı şu şekildedir:

```log
< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:35:58.582 BST > LOG:  statement: CREATE TABLE t3 (c1 int);

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:35:58.583 BST > LOG:  AUDIT: SESSION,2,1,DDL,CREATE TABLE,TABLE,public.t3,CREATE TABLE t3 (c1 int);,<not logged>

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:35:58.585 BST > LOG:  duration: 3.092 ms

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:06.291 BST > LOG:  statement: INSERT INTO t3 VALUES (1);

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:06.291 BST > LOG:  AUDIT: SESSION,3,1,WRITE,INSERT,TABLE,public.t3,INSERT INTO t3 VALUES (1);,<not logged>

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:06.293 BST > LOG:  duration: 1.604 ms

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:09.940 BST > LOG:  statement: DROP TABLE t3;

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:09.941 BST > LOG:  AUDIT: SESSION,4,1,DDL,DROP TABLE,TABLE,public.t3,DROP TABLE t3;,<not logged>

< user=postgres db=postgres host=[local] pid=29129 time=2017-07-23 12:36:09.943 BST > LOG:  duration: 2.634 ms
```
{% include links.html %}
