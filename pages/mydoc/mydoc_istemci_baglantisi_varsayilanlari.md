---
title: "İstemci Bağlantısı Varsayılanları"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 18, 2020
summary: "İstemci Bağlantısı Varsayılanları"
sidebar: mydoc_sidebar
permalink: mydoc_istemci_baglantisi_varsayilanlari.html
folder: mydoc
---

## İstemci Bağlantısı Varsayılanları

{% include callout.html content="**`client_min_messages (enum)`**: İstemciye hangi mesaj seviyelerinin gönderileceğini kontrol eder. Geçerli değerler `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `LOG`, `NOTICE`, `WARNING` ve `ERROR` şeklindedir. Her seviye, onu takip eden sonraki tüm seviyeleri kapsar. Seviye ne kadar sonra olursa, o kadar az mesaj gönderilir. Varsayılan, `NOTICE`'dir. `LOG`'un burada [`log_min_messages`](mydoc_error_reporting_logging.html#when-to-log)'dakinden farklı sıralamada olduğuna dikkat edin.<br/><br/>

`INFO` seviyesindeki mesajlar her zaman istemciye gönderilir." type="primary" %}

{% include callout.html content="**`search_path (string)`**: Bu parametre, bir nesneye (tablo, veri türü, işlev, vb.) şema belirtilmeden başvurulduğunda şemaların aranma sırasını belirtir. Farklı şemalarda aynı isimli nesneler olduğunda, arama yolunda ilk bulunan nesne kullanılır. Arama yolundaki şemaların hiçbirinde bulunmayan bir nesneye yalnızca içerdiği şema noktalı bir adla belirtilerek başvurulur.<br/><br/>

`search_path` değeri şema adlarının virgülle ayrılmış bir listesi olmalıdır. Mevcut bir şema olmayan veya kullanıcının `USAGE` iznine sahip olmadığı bir şema yok sayılır.<br/><br/>

Liste öğelerinden biri `$user` özel adıysa, eğer böyle bir şema varsa ve kullanıcı bunun için `USAGE` iznine sahipse `CURRENT_USER` tarafından döndürülen isme sahip şema dönderilir, değilse, `$user` dikkate alınmaz.<br/><br/>

`pg_catalog` sistem kataloğu şeması, yolda belirtilmiş olsun veya olmasın her zaman aranır. Yolda belirtilmişse, belirtilen sırada aranacaktır. `pg_catalog` yolda belirtilmemişse, herhangi bir yol öğesi aramadan önce aranacaktır.<br/><br/>

Benzer şekilde, mevcut oturumun geçici tablo şeması, `pg_temp_**nnn**` bulunuyorsa her zaman aranır. `pg_temp` yolda, takma adı kullanılarak listelenebilir. Yolda listelenmemişse `pg_catalog`'dan bile önce aranır. Geçici şema yalnızca ilişki (tablo, görünüm, sequence, vb.) ve veri türü adları için aranır. Hiçbir zaman işlev veya operatör adları aranmaz.<br/><br/>

Nesneler, belirli bir hedef şema belirtilmeden oluşturulduğunda `search_path` içinde adlandırılan ilk geçerli şemaya yerleştirilecektir. Arama yolu boşsa bir hata rapor edilir.<br/><br/>

Bu parametrenin varsayılan değeri '$user', yani `public`'dir. Bu ayar, bir veritabanının paylaşılan kullanımını (hiçbir kullanıcının özel şemasının olmadığı ve tümünün ortak kullanımı paylaştığı), kullanıcı bazlı özel şemaları ve bunların kombinasyonlarını destekler. Diğer etkiler varsayılan arama yolu ayarını global olarak veya kullanıcı bazında değiştirerek elde edilebilir.<br/><br/>

Şema hakkında daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/ddl-schemas.html). Varsayılan konfigürasyon yalnızca veri tabanında tek bir kullanıcılı veya karşılıklı güvenin olduğu birkaç kullanıcılı durumlarda uygundur.<br/><br/>

Arama yolunun mevcut geçerli değeri, `current_schemas` SQL fonksiyonu ile sorgulanabilir (bkz. [](https://www.postgresql.org/docs/current/functions-info.html)). search_path'in değerini incelemekle bu aynı değildir, çünkü `current_schemas`, search_path'te görünen öğelerin nasıl çözümlendiğini gösterir." type="primary" %}

{% include callout.html content="**`row_security (boolean)`**: Bu parametre, satır güvenlik politikası uygulamayı kontrol eder. `on` ayarında, politikalar normal şekilde uygulanır. `off` ayarında, sorgular başarısız olur. Varsayılan açıktır. Sınırlı satır visibility'sini hatalı sonuçlara neden olabileceği durumlarda kapalı olarak değiştirin. örneğin, pg_dump bu değişikliği varsayılan olarak yapar. Bu değişkenin her satır güvenlik politikasını atlayan roller üzerinde, `BYPASSRLS` özniteliğine sahip zekaya, süper kullanıcılara ve rollere etkisi yoktur.<br/><br/>

Satır güvenlik politikaları hakkında daha fazla bilgi için bkz. [CREATE POLICY](https://www.postgresql.org/docs/current/sql-createpolicy.html)." type="primary" %}

{% include callout.html content="**`default_table_access_method (string)`**: Bu parametre, tablolar ve materialized görünümler oluştururken CREATE komutu açıkça bir erişim yöntemi belirtmediğinde veya bir tablo erişim yöntemi belirtmeye izin vermeyen `SELECT ... INTO` kullanıldığında, kullanılacak varsayılan tablo erişim yöntemini belirtir. Varsayılan, `heap`'dir." type="primary" %}


{% include callout.html content="**`default_tablespace (string)`**: Bu parametre, `CREATE` komutu açıkça bir tablespace belirtmediğinde, nesnelerin (tablolar ve indeksler) oluşturulacağı varsayılan tablespace'i belirtir. Ayrıca bölümlenmiş (partitioned) bir ilişkinin gelecekteki partition'ları yönlendireceği tablespace'i de belirler.<br/><br/>

Parametre değeri, bir tablespace adı veya geçerli veritabanının varsayılan tablespace'ini kullanarak belirtmek için boş dizedir. Değer, mevcut herhangi bir tablespace ile eşleşmiyorsa, PostgreSQL otomatik olarak mevcut veritabanının varsayılan tablespace'ini kullanır. Varsayılan olmayan bir tablespace belirtildiğinde kullanıcının bunun için `CREATE` ayrıcalığına sahip olması gerekir, aksi takdirde oluşturma girişimleri başarısız olur.<br/><br/>

Bu parametre, geçici tablolar için kullanılmaz. Bunlar için `temp_tablespaces`'e başvurulur.<br/><br/>

Bu parametre, veritabanları oluşturulurken kullanılmaz. Yeni veritabanı tablespace ayarını varsayılan olarak kopyalandığı template veritabanından devralır.<br/><br/>

Tablespace hakkında daha fazla bilgi için bkz.[](https://www.postgresql.org/docs/current/manage-ag-tablespaces.html)" type="primary" %}


{% include callout.html content="**`temp_tablespaces (string)`**: Bu parametre, CREATE komutu açıkça bir tablespace belirtmediğinde geçici nesnelerin (geçici tablolar ve geçici tablolardaki indeksler) oluşturulacağı tablespace'leri belirtir. Bu tablespace'lerde büyük veri kümelerini sıralamak gibi amaçlar için geçici dosyalar da oluşturulur.<br/><br/>

Parametre değeri, tablespace adlarının bir listesidir. Listede birden fazla isim olduğunda, PostgreSQL her geçici nesne oluşturulduğunda listenin rastgele bir üyesini seçer. Transaction içinde art arda oluşturulan geçici nesneler listedeki ardışık tablespace'lere yerleştirilir. Listenin seçilen öğesi boş bir dizeyse, PostgreSQL otomatik olarak bunun yerine geçerli veritabanının varsayılan tablespace'ini kullanır.<br/><br/>

`temp_tablespaces` etkileşimli olarak ayarlanırken varolmayan bir tablespace'in belirtilmesi kullanıcının CREATE yetkisine sahip olmadığı bir tablespace'in belirtilmesi gibi bir hatadır. Önceden ayarlanmış bir değer kullanılırken, var olmayan tablo alanları ve kullanıcının CREATE yetkisine sahip olmadığı tablo alanları yok sayılır. Bu kural *postgresql.conf* dosyasında ayarlanan bir değer kullanıldığında geçerlidir.<br/><br/>

Varsayılan değer boş bir dizedir, bu da tüm geçici nesnelerin geçerli veritabanının varsayılan tablespace'de oluşturulmasına neden olur. `default_tablespace` parametresine bakın.
" type="primary" %}

{% include callout.html content="**`check_function_bodies (boolean)`**: Bu parametre normalde açıktır. `off` ayarında, `CREATE FUNCTION` sırasında işlev gövde dizesinin doğrulanmasını devre dışı bırakır. Doğrulamayı devre dışı bırakmak, doğrulama sürecinin yan etkilerini ve ileriye dönük referanslar gibi sorunlardan kaynaklı yanlış pozitifleri önler. Diğer kullanıcılar adına işlevleri yüklemeden önce bu parametreyi `off` olarak ayarlayın. pg_dump bunu otomatik olarak yapar." type="primary" %}

{% include callout.html content="**`default_transaction_isolation (enum)`**: Her SQL transaction bir izolasyon seviyesine sahiptir. Bu izolasyon seviyesi; 'read uncommitted', 'read committed', 'repeatable read' veya 'serializable' olabilir. Bu parametre yeni transaction'ların varsayılan izolasyon seviyesini kontrol eder. Varsayılan, 'read committed'dir.<br/><br/>

Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/mvcc.html) ve [SET TRANSACTION](https://www.postgresql.org/docs/current/sql-set-transaction.html)" type="primary" %}

{% include callout.html content="**`default_transaction_read_only (boolean)`**: read-only bir SQL transaction, geçici olmayan tabloları değiştiremez. Bu parametre, her yeni işlemin varsayılan read-only durumunu kontrol eder. Öntanımlı değeri `off`'dur (read/write).

Daha fazla bilgi için bkz. [SET TRANSACTION](https://www.postgresql.org/docs/current/sql-set-transaction.html)" type="primary" %}

{% include callout.html content="**`default_transaction_deferrable (boolean)`**: serializable izolasyon seviyesinde çalışan deferrable read-only bir SQL transaction devam etmesine izin verilmeden önce ertelenebilir. Ancak, bir kez çalıştırılmaya başladığında, serializability'i sağlamak için gereken ek yüklerin hiçbirine maruz kalmaz. Bu nedenle, serialization kodunun eşzamanlı güncellemeler nedeniyle bunu durdurmaya zorlamak için hiçbir nedeni olmayacağından bu seçeneği uzun süreli read-only transaction'lar için uygun kılar.<br/><br/>

Bu parametre, her yeni transaction'ın varsayılan deferrable durumunu kontrol eder. Şu anda okuma-yazma transaction'ları ve `serializable`'dan daha düşük izolasyon seviyesinde çalışanlar üzerinde hiçbir etkisi yoktur. Öntanımlı değeri `off`'dur.<br/><br/>

Daha fazla bilgi için bkz. [SET TRANSACTION](https://www.postgresql.org/docs/current/sql-set-transaction.html)" type="primary" %}

{% include callout.html content="**`session_replication_role (enum)`**: Mevcut oturumun replikasyonla ilgili tetikleyicilerin ve rewrite kurallarının davranışını kontrol eder. Bu parametre ayarı süper kullanıcı yetkisi gerektirir ve önceden önbelleğe alınmış sorgu planlarının atılmasına neden olur. Olası değerler `origin` (varsayılan), `replica` ve `local`'dir.<br/><br/>

Bu parametrenin kullanım amacı şudur: logical replikasyon sistemleri, replike edilen değişiklikleri uygularken bunu `replica` değerine ayarlar. Bunun etkisi, tetikleyicilerin ve kuralların standby'da tetiklenmemesi olacaktır. Daha fazla bilgi için [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html) ENABLE TRIGGER ve ENABLE RULE aksiyonlarına bakın.<br/><br/>

PostgreSQL, `origin` ve `local` ayarları dahili olarak aynı şekilde ele alır. Bu iki değeri üçüncü taraf replikasyon sistemleri dahili amaçları için kullanabilir. örneğin, değişiklikleri replike etmemesi gereken bir oturumu belirlemek için `local` değerini kullanabilir.<br/><br/>

Yabancı anahtarlar tetikleyici olarak uygulandığından, bu parametrenin `replica` ayarlanması tüm yabancı anahtar denetimlerini de devre dışı bırakır. Bu hatalı kullanıldığında verileri tutarsız bir durumda bırakabilir." type="primary" %}

### İfade (Statement) Davranışı

### Locale ve Formatting

### Paylaşılan Kütüphane Ön Yükleme

### Diğer Varsayılanlar

{% include links.html %}
