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



{% include callout.html content="`wal_level (enum)`: `wal_level`, WAL'a ne kadar bilgi yazılacağını belirler. Varsayılan değer `replica`'dır. replica değeri bir standby sunucu üzerinde çalışan read-only sorgular dahil, WAL archiving ve replication işlemleri için yeterli veriyi sağlar. `minimal`, bir çökme veya ani kapanma durumunda kurtarma işlemleri için gereken bilgileri garantiye alır, toplu veri operaslarıyla ilgili işlemleri (`CREATE TABLE AS SELECT`, `CREATE INDEX`) WAL günlüğüne kaydetmeyerek depolama avantajı sağlar. Son olarak `logical`, logical decoding ve logical replication için gerekli bilgileri sağlar. Her düzey, tüm alt düzeylerde kaydedilen bilgileri içerir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.<br/><br/>

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

{% include callout.html content="`fsync (boolean)`: PostgreSQL sunucusu bu parametre açıksa, `fsync ()` sistem çağrıları veya çeşitli eşdeğer yöntemler yayınlayarak değişikliklerin fiziksel olarak diske yazıldığından emin olmaya çalışır. Bu, veritabanı kümesinin bir işletim sistemi veya donanım çökmesinden sonra tutarlı bir duruma geri yüklenebilmesini sağlar.<br/><br/>

`fsync`'i kapatmak genellikle bir performans avantajı olsamasına rağmen bir elektrik kesintisi veya sistem çökmesi durumunda kurtarılamaz veri bozulmasına neden olabilir. Bu nedenle, yanlızca tüm veritabanınızı dış verilerden kolayca yeniden oluşturabiliyorsanız `fsync`'i kapatmanız önerilir.<br/><br/>

fsync'i kapatmak için güvenli koşullara, yeni veritabanı kümesinin bir yedekleme dosyasından ilk yüklenmesi, sık sık yeniden oluşturulan ve failover için kullanılmayan read-only bir veritabanı klonu örnek olarak verilebilir. Yüksek kaliteli donanım tek başına fsync'i kapatmak için yeterli bir gerekçe değildir.<br/><br/>

`fsync` kapılıdan açık duruma getirildiğinde güvenilir kurtarma sağlamak için, çekirdekteki tüm değiştirilmiş buffer'ları dayanıklı depolamaya zorlamak gerekir. Bu, küme kapalıyken veya `fsync` açıkken `initdb --sync-only` komutuyla, `sync` çalıştırılarak, dosya sisteminin bağlantısını keserek veya sunucuyu yeniden başlatarak yapılabilir.<br/><br/>

Çoğu durumda, kritik olmayan transactionlar için `synchronous_commit`'i kapatmak, `fsync`'i kapatmanın getireceği potansiyel performans avantajının fazlasını veri bozulmasına bağlı riskler olmadan sağlayabilir.<br/><br/>

fsync, yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. Bu parametreyi kapatırsanız, `full_page_writes`'i de kapatmak düşünülebilir." type="primary" %}

{% include callout.html content="`synchronous_commit (enum)`: Veritabanı sunucusu istemciye bir 'success' işareti döndürmeden önce ne kadar WAL işlemenin tamamlanması gerektiğini belirtir. Geçerli değerler `remote_apply`, `on` (varsayılan), `remote_write`, `local` ve `off` şeklindedir.<br/><br/>

`synchronous_standby_names` boşsa, anlamlı ayarlar yalnızca `on` ve `off`'dur; `remote_apply`, `remote_write` ve `local`, `on` aynı yerel senkronizasyon seviyesini sağlar. `off` olmayan tüm modların yerel davranışı, WAL'ın diske yerel olarak flush edilmesini beklemektir. `off` modunda bekleme yoktur, bu nedenle istemciye başarının bildirilmesi ile transaction'ın sonrasında bir sunucu çökmesine karşı güvenli olmasının garanti edilmesi arasında bir gecikme olabilir. (Maksimum gecikme `wal_writer_delay`'in üç katıdır.) fsync'in aksine, bu parametrenin `off` olarak ayarlanması herhangi bir veritabanı tutarsızlığı riski oluşturmaz. İşletim sistemi veya veritabanı çökmesi, yakın zamanda taahhüt edildiği iddia edilen bazı transaction'ların kaybolmasına neden olabilir, ancak veritabanı durumu bu transaction'lar temiz bir şekilde iptal edilmiş gibi olacaktır. Bu nedenle, işlemin dayanıklılığı konusunda kesinlikten yerine performansın daha önemli olduğu durumlarda `synchronous_commit`'i kapatmak faydalı olabili. Daha fazlası için [Asynchronous Commit](https://www.postgresql.org/docs/current/wal-async-commit.html) bölümüne bakın.<br/><br/>

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

{% include callout.html content="`wal_sync_method (enum)`: WAL değişikliklerini diske göndermeye zorlamak için kullanılan yöntem. `fsync` kapalıysa bu ayar geçersizdir. Olası değerler şunlardır: `open_datasync`, `fdatasync`, `fsync`, `fsync_writethrough`, `open_sync`. <br/><br/>

Verilen seçenekler tüm platformlarda mevcut değildir. `fdatasync` Linux'ta varsayılan değerdir. Diğerleri için varsayılan, yukarıdaki listede platform tarafından desteklenen ilk yöntemdir. Varsayılan mutlaka en iyisi değildir; Kilitlenmeye karşı korumalı bir yapılandırma oluşturmak veya optimum performans elde etmek için bu ayarı veya sistem yapılandırmanızın diğer yönlerini değiştirmeniz gerekebilir. Bu parametre yalnızca *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. Bu yönler [Reliability](https://www.postgresql.org/docs/current/wal-reliability.html) bölümünde ele alınmıştır." type="primary" %}

{% include callout.html content="`full_page_writes (boolean)`: PostgreSQL sunucusu bu parametre açık olduğunda, disk sayfalarının tüm içeriğini ilgili sayfanın checkpoint'den sonraki ilk değişikliği sırasında WAL'a yazar. İşletim sistemi çökmesi sırasında işlemde olan bir sayfa yazma kısmen tamamlanarak disk üzerinde eski ve yeni verilerin karışımını içeren bir sayfaya yol açacağı için bu gereklidir. Böyle bir sayfa için WAL'de depolanan satır düzeyinde değişiklik verileri çökme sonrası kurtarma işleminde tamamen geri yüklemek için yeterli olmayacaktır. Tam sayfa görüntünün saklanması, sayfanın doğru bir şekilde geri yüklenmesini garanti ederken WAL'a yazılması gereken veri miktarını artırır.<br/><br/>

(WAL replay her zaman bir checkpoint'den başlar, bunu bir checkpoint'den sonra her sayfanın ilk değişikliği sırasında yapması yeterlidir. Bu nedenle, full-page write maliyetini azaltmanın bir yolu, checkpoint aralığı parametrelerini artırmaktır.)" type="primary" %}

### Checkpoints

### Archiving ( Arşivleme )

### Archive Recovery ( Arşiv Kurtarma )

### Recovery Target ( Kurtarma Hedefi )

{% include links.html %}
