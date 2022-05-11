---
title: "Hata Raporlama ve Logging"
layout: default
parent: Veritabanı Yapılandırması
nav_order: 8
---

## Hata Raporlama ve Logging

### Where to Log

#### `log_destination`

{% include parameter_info.html parametre="log_destination" %}

{% include callout.html content=" **PostgreSQL, sunucu mesajlarını günlüğe kaydetmek için stderr, csvlog ve syslog yöntemlerini destekler.** Windows'ta olay günlüğü de desteklenmektedir. Bu parametre değeri, virgülle ayrılmış istenen günlük hedeflerinin listesi şeklindedir. Varsayılan, yalnızca stderr'de günlüğe yazmadır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

`log_destination` listesi csvlog içeriyorsa; günlük kayıtları, günlükleri programlara yüklemek için uygun olan virgülle ayrılmış değerler (comma separated value-CSV) formatında yazılır. Ayrıntılar için [CSV-Format Log Çıktısı Kullanma](mydoc_error_reporting_logging.html#csv-format-log-%C3%A7%C4%B1kt%C4%B1s%C4%B1-kullanma) bölümüne bakın. CSV formatında günlük çıktısı oluşturmak için `logging_collector` etkinleştirilmelidir.<br/><br/>

`log_destination` listesi stderr veya csvlog içerdiğinde; logging collector tarafından kullanımda olan günlük dosyalarının konumunu ve ilişkili günlük kaydı hedefini kaydetmek için `current_logfiles` dosyası oluşturulur. Bu, veritabanı tarafından kullanımda olan günlükleri bulmak için kullanışlı bir yol sağlar. Bu dosyanın içeriğinin bir örneği:" type="primary" %}

```bash
stderr log/postgresql.log
csvlog log/postgresql.csv
```

{% include tip.html content=" Örnek kullanım:<br/><br/>
**log_destination = 'stderr,syslog**'" type="primary" %}

{% include callout.html content=" `current_logfiles`, rotasyonun bir etkisi olarak yeni bir günlük dosyası oluşturulduğunda ve `log_destination` yeniden yüklendiğinde tekrar oluşturulur. `current_logfiles`, `log_destination` stderr veya csvlog içermediğinde ve logging collector devre dışı bırakıldığında kaldırılır." type="primary" %}

{% include note.html content=" Çoğu Unix sistemde, **log_destination**'ın 'syslog' opsiyonunu kullanırken sisteminizin syslog arka plan programının yapılandırmasını değiştirmeniz gerekir. PostgreSQL `LOCAL0`'dan `LOCAL7`'ye kadar syslog facility kullanabilir, ancak çoğu platform bu gibi mesajları yok sayar. Çalışmasını sağlamak için syslog daemon yapılandırma dosyasına şu şekilde ekleme yapmanız gerekir:<br/><br/>
**local0. */var/log/postgresql**
"%}

#### `logging_collector`

{% include parameter_info.html parametre="logging_collector" %}

{% include callout.html content=" **Bu parametre, stderr'e gönderilen günlük mesajlarını yakalayıp bunları günlük dosyalarına yönlendiren bir arka plan süreci olan *logging collector'ı* etkinleştirir.** Bu yaklaşım, bazı mesaj türleri syslog çıktısında görünmeyebileceğinden genellikle syslog'a kaydetmekten daha kullanışlıdır. Dynamic-linker hata mesajları, `archive_command` gibi komut dosyaları tarafından üretilen hata mesajları genel örneklerdir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include note.html content=" Logging collector kullanmadan stderr'ı log'lamak mümkündür. Günlük mesajları sadece sunucunun stderr'ının yönlendirildiği yere gidecektir. Ancak, günlük dosyalarının rotasyonu için kullanışlı bir yol sağlamadığından, bu yöntem yalnızca düşük günlük boyutları için uygundur. Ayrıca, logging collector kullanmayan bazı platformlarda, aynı günlük dosyasına aynı anda yazan birden çok süreç birbirinin çıktısı üzerine yazabileceğinden günlük kaydının kaybolmasına veya bozulmasına neden olabilir."%}

{% include note.html content=" Logging collector, mesajları asla kaybetmeyecek şekilde tasarlanmıştır. Bu, aşırı yük durumunda, collector geride kalıp ek günlük mesajları göndermeye çalıştığında sunucu süreçlerinin bloklanabileceği anlamına gelir. Bunun tersine; syslog, mesajları yazamıyorsa drop etmeyi tercih eder. Bu da böyle durumlarda bazı mesajları günlüğe kaydetmede başarısız olabileceği, ancak sistemin geri kalanını bloklamayacağı anlamına gelir."%}

#### `log_directory`

{% include parameter_info.html parametre="log_directory" %}

{% include callout.html content=" **Bu parametre `logging_collector` etkinleştirildiğinde günlük dosyalarının oluşturulacağı dizini belirler.** Mutlak bir yol olarak veya küme veri dizinine ( data directory ) göre belirtilebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri `log`'dur." type="primary" %}

#### `log_filename`

{% include parameter_info.html parametre="log_filename" %}

{% include callout.html content=" **Bu parametre `logging_collector` etkinleştirildiğinde oluşturulan günlük dosyalarının adlarını ayarlar.** Değer `strftime` kalıbı olarak işlenir. Zamanla değişen dosya adları `%` kaçışları ile belirtilir. Saat dilimine bağlı `%` kaçışları varsa, hesaplama `log_timezone` ile belirtilen bölgede yapılır. Doğrudan sistemin `strft` zamanı kullanılmadığı için platform spesifik uzantıları çalışmaz. Varsayılan, `postgresql-%Y-%m-%d_%H%M%S.log` şeklindedir.<br/><br/>

`log_destination`'da CSV formatında çıktı etkinleştirildiğinde zaman damgalı günlük dosyası adına `.csv` eklenir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `log_file_mode`

{% include parameter_info.html parametre="log_file_mode" %}

{% include callout.html content=" **Bu parametre, `logging_collector` etkinleştirildiğinde, Unix sistemlerde günlük dosyalarının izinlerini ayarlar.** (Windows'da bu parametre yoksayılır.) Parametre değerinin, `chmod` ve `umask` sistem çağrıları tarafından kabul edilen formatta sayısal bir mod olması beklenir.<br/><br/>

Varsayılan izinler `0600`'dır, yani yalnızca sunucu sahibi günlük dosyalarını okuyabilir ve yazabilir. Bir diğer yaygın kullanım ayarı `0640`'tır ve sahip grubunun üyelerinin dosyaları okumasına izin verir. Böyle bir ayarı kullanmak için ve dosyaları küme veri dizininin dışında bir yerde depolamak için `log_directory`'yi değiştirmeniz gerekecektir. Her durumda, hassas veriler içerdiği için günlük dosyalarını herkes tarafından okunabilir hale getirmek akıllıca değildir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `log_rotation_age`

{% include parameter_info.html parametre="log_rotation_age" %}

{% include callout.html content=" **Bu parametre, `logging_collector` etkinleştirildiğinde tek bir günlük dosyasının maksimum kullanım süresini belirler ve bu değerden sonra yeni bir günlük dosyası oluşturulur.** Bu değer birimsiz belirtilirse dakika olarak alınır. Varsayılan 24 saattir. Yeni günlük dosyalarının zamana dayalı olarak oluşturulmasını devre dışı bırakmak için 0 olarak ayarlayın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `log_rotation_size`

{% include parameter_info.html parametre="log_rotation_size" %}

{% include callout.html content=" **Bu parametre, `logging_collector` etkinleştirildiğinde tek bir günlük dosyasının maksimum boyutunu belirler.** Bu miktarda veri bir günlük dosyasına gönderildikten sonra, yeni bir günlük dosyası oluşturulacaktır. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı değeri 10 megabayttır. Yeni günlük dosyalarının boyuta dayalı olarak oluşturulmasını devre dışı bırakmak için 0 olarak ayarlayın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `log_truncate_on_rotation`

{% include parameter_info.html parametre="log_truncate_on_rotation" %}

{% include callout.html content=" **Bu parametre, `logging_collector` etkinleştirildiğinde PostgreSQL günlük dosyalarına ekleme yapmak yerine mevcut günlük dosyasını truncate (üzerine yazmak) eder.** Truncate yalnızca zamana dayalı rotasyon sebepli yeni dosya açılırken meydana gelir, sunucu başlangıcında veya boyuta dayalı rotasyon sırasında değil. Kapalı olduğunda, var olan dosyalara her durumda ekleme yapılacaktır. Örneğin, bu ayarı `postgresql-%H.log` gibi bir `log_filename` ile birlikte kullanılması 24 saatlik günlük dosyalarının oluşturulup ve döngüsel olarak bunların üzerine yazılmasıyla sonuçlanır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Örnek: 7 günlük `server_log.Mon`, `server_log.Tue`, vb. şekilde günlük dosyaları tutmak ve geçen haftanın günlüğünün üzerine bu haftanın günlüğünü otomatik olarak yazmak için, `log_filename` değerini `server_log.%a`, `log_truncate_on_rotation = on` ve `log_rotation_age = 1440` olarak ayarlayın.<br/><br/>

Örnek: Her saatte bir günlük dosyası olarak 24 saatlik günlükler ve günlük dosyası boyutu 1 GB'ı aşarsa daha erken rotasyon için `log_filename` değerini `server_log.%H%M`, `log_truncate_on_rotation = on`, `log_rotation_age = 60` ve `log_rotation_size = 1000000` olarak ayarlayın. `log_filename`'in `%M` içermesi, meydana gelebilecek boyuta dayalı rotasyonlarda mevcut saatlik dosya adından farklı bir dosya adı seçmek içindir." type="primary" %}

#### `syslog_facility`

{% include parameter_info.html parametre="syslog_facility" %}

{% include callout.html content=" **Bu parametre syslog'da günlük kaydı etkinleştirildiğinde kullanılacak syslog 'facility' belirler.** `LOCAL0`, `LOCAL1`, `LOCAL2`, `LOCAL3`, `LOCAL4`, `LOCAL5`, `LOCAL6`, `LOCAL7` arasından seçim yapabilirsiniz. Öntanımlı değeri `LOCAL0`'dır. Ayrıca sisteminizin syslog daemon belgelerine bakın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `syslog_ident`

{% include parameter_info.html parametre="syslog_ident" %}

{% include callout.html content=" **Bu parametre syslog'da günlük kaydı etkinleştirildiğinde, syslog günlüklerinde PostgreSQL mesajlarını tanımlamak için kullanılan program adını belirler.** Varsayılan, `postgres`'tir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `syslog_sequence_numbers`

{% include parameter_info.html parametre="syslog_sequence_numbers" %}

{% include callout.html content=" **syslog'da günlük kaydı etkinleştirildiğinde ve bu parametre `on` (varsayılan) ise her mesajın önüne artan bir sıra numarası eklenir ([2] gibi).** Bu, birçok syslog uygulamasının varsayılan olarak gerçekleştirdiği '--- last message repeated N times ---' bastırmasını engeller. Daha modern syslog uygulamalarında tekrarlanan mesaj bastırma yapılandırılabildiğinde bu gerekli olmayabilir. Tekrarlanan mesajları bastırmak istiyorsanız bunu kapatabilirsiniz.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `syslog_split_messages`

{% include parameter_info.html parametre="syslog_split_messages" %}

{% include callout.html content=" **Bu parametre syslog'da günlük kaydı etkinleştirildiğinde mesajların syslog'a nasıl teslim edileceğini belirler.** `on` (varsayılan) ayarında, mesajlar satırlara bölünür ve uzun satırlar, geleneksel syslog uygulamaları için tipik boyut sınırı olan 1024 bayta sığacak şekilde bölünür. `off` ayarında, PostgreSQL sunucusu günlük mesajları syslog servisine olduğu gibi teslim edilir ve büyük mesajlarla başa çıkmak syslog servisine bırakılır.<br/><br/>

Eğer syslog bir metin dosyasına kaydediliyorsa etki her iki şekilde de aynı olacaktır ve çoğu syslog uygulaması büyük iletileri işleyemeyeceği veya bunları işlemek için özel olarak yapılandırılması gerekeceği için en iyisi ayarı açık bırakmaktır. syslog nihayetinde başka bir ortama yazıyorsa, mesajları mantıksal olarak bir arada tutmak gerekli veya daha yararlı olabilir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `event_source`

{% include parameter_info.html parametre="event_source" %}

{% include callout.html content=" **event log'a loglama etkinleştirildiğinde bu parametre, günlükteki PostgreSQL mesajlarını tanımlamak için kullanılan program adını belirler.** Varsayılan, `PostgreSQL`'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### When to Log

#### `log_min_messages`

{% include parameter_info.html parametre="log_min_messages" %}

{% include callout.html content=" **Sunucu günlüğüne hangi mesaj seviyelerinin yazılacağını kontrol eder.** Geçerli değerler `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` ve `PANIC`'dir. Her seviye, onu takip eden tüm seviyeleri kapsar. Seviye ne kadar düşük olursa, günlüğe o kadar az mesaj gönderilir. Varsayılan, `WARNING`'dır. `LOG`'un burada [client_min_messages](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-MIN-MESSAGES)'dakinden farklı bir sıralamada olduğunu unutmayın. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_min_error_statement`

{% include parameter_info.html parametre="log_min_error_statement" %}

{% include callout.html content=" **Hata durumuna neden olan SQL ifadelerinin sunucu günlüğüne kaydedilmesini kontrol eder.** Verilen seviyede veya üzerinde hata oluşturan tüm SQL ifadeleri günlüğe kaydedilir. Geçerli değerler `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` ve `PANIC`'dir. Varsayılan `ERROR`'dır. Bu hataya, günlük mesajlarına, fatal error'lara ve paniklere neden olan SQL ifadelerinin günlüğe kaydedileceği anlamına gelir. Başarısız ifadelerin günlüğe kaydedilmesini kapatmak için bu parametreyi `PANIC` olarak ayarlayın. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_min_duration_statement`

{% include parameter_info.html parametre="log_min_duration_statement" %}

{% include callout.html content=" **En az bu parametrede belirtilen sürede tamamlanan tüm ifadeleri günlüğe kaydeder.** Örneğin, 250 ms ayarı, 250 ms ve daha uzun süre çalışan tüm SQL ifadelerini günlüğe kaydeder. Bu parametrenin etkinleştirilmesi, uygulamalarınızdaki optimize edilmemiş sorguların izlenmesinde yardımcı olur. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Bunu 0 olarak ayarlamak, çalışan tüm SQL ifade sürelerini yazdırır. -1 (varsayılan) değeri bu davranışı devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu parametre, `log_min_duration_sample` parametresini geçersiz kılar, yani bu ayarı aşan süreye sahip sorgular örneklemeye tabi değildir ve her zaman günlüğe kaydedilir.<br/><br/>

Genişletilmiş sorgu protokolü kullanan istemciler için Parse, Bind ve Execute adımlarının süreleri bağımsız olarak günlüğe kaydedilir." type="primary" %}

{% include note.html content=" Bu parametre `log_statement` ile birlikte kullanırken, `log_statement` nedeniyle günlüğe kaydedilen ifadeler süre günlüğü mesajında ​​tekrarlanmayacaktır."%}

#### `log_min_duration_sample`

{% include parameter_info.html parametre="log_min_duration_sample" %}

{% include callout.html content=" **En az belirtilen süre boyunca çalışan tamamlanmış ifadelerin süresinin örneklenmesine (sampling) edilmesine izin verir.** `log_statement_sample_rate` tarafından kontrol edilen örnekleme oranıyla yürütülen ifadelerin bir alt kümesi için `log_min_duration_statement` ile aynı türden günlük girişleri üretir. Örneğin, 100 ms ayarları, 100 ms ve daha uzun süre çalışan tüm SQL ifadeleri örnekleme için dikkate alacaktır. Bu parametrenin etkinleştirilmesi, tüm sorguları günlüğe kaydedemeyecek kadar yüksek trafik olduğunda faydalı olabilir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Bunu 0 olarak ayarlamak tüm ifade sürelerini örnekler. -1 (varsayılan) örnekleme ifadesi sürelerini devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu ayar, `log_min_duration_statement`'tan daha düşük önceliğe sahiptir. `log_min_duration_statement`'ı aşan süredeki ifadeler örneklemeye tabi değildir ve her zaman günlüğe kaydedilir.<br/><br/>

`log_min_duration_statement` için verilen notlar bu ayar için de geçerlidir." type="primary" %}

#### `log_statement_sample_rate`

{% include parameter_info.html parametre="log_statement_sample_rate" %}

{% include callout.html content=" **Günlüğe kaydedilecek `log_min_duration_sample`'ı aşan süredeki ifadelerin oranını belirler.** Örnekleme stokastiktir; örneğin 0.5, istatistiksel olarak herhangi bir ifadenin günlüğe kaydedilme ihtimalinin 1/2 olduğu anlamına gelir. Varsayılan değer 1.0'dır, tüm örneklenmiş ifadeler günlüğe kaydedilir. Bunu 0 olarak ayarlamak örneklenmiş ifade süresi günlük kaydını devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_transaction_sample_rate`

{% include parameter_info.html parametre="log_transaction_sample_rate" %}

{% include callout.html content=" **Diğer nedenlerle günlüğe kaydedilen ifadelere ek olarak, tüm ifadeleri günlüğe kaydedilen transaction'ların oranını ayarlar.** İfadelerinin sürelerine bakılmaksızın her yeni transaction için geçerlidir. Örnekleme stokastiktir, örneğin 0.1, istatistiksel olarak herhangi bir işlemin günlüğe kaydedilme ihtimalinin onda bir olduğu anlamına gelir. `log_transaction_sample_rate`, transaction'ların örneğini oluşturmada yardımcı olabilir. Varsayılan 0'dır, herhangi bir ek transaction'dan gelen ifadelerin günlüğe kaydedilmemesi anlamına gelir. Bunu 1 olarak ayarlamak, tüm transaction'ların tüm ifadelerini günlüğe kaydeder. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include note.html content=" Diğer ifade kaydı parametreleri gibi bu parametre de ek yükler getirebilir."%}

Aşağıda verilen tabloda, PostgreSQL tarafından kullanılan mesaj önem seviyeleri açıklanmıştır. Günlük çıktısı, syslog veya Windows olay günlüğüne gönderilirdiğinde önem seviyeleri tabloda gösterildiği şekilde çevrilir.

| Seviye | Kullanım | syslog | eventlog |
|-------|--------|-------|--------|
| DEBUG1 .. DEBUG5 | Geliştiriciler tarafından kullanılmak üzere sırayla daha ayrıntılı bilgi sağlar. | DEBUG | INFORMATION |
| INFO | Kullanıcı tarafından dolaylı olarak talep edilen bilgileri sağlar. ör, VACUUM VERBOSE çıktısı. | INFO | INFORMATION |
| NOTICE | Kullanıcılara yardımcı olabilecek bilgiler sağlar. ör, uzun tanımlayıcıların truncation'ına dair bildirim. | NOTICE | INFORMATION |
| WARNING | Olası sorunlara ilişkin uyarılar verir. | NOTICE | WARNING |
| ERROR | Mevcut komutun iptal edilmesine neden olan hatayı bildirir. | WARNING | ERROR |
| LOG | Yöneticilere ilgilendikleri bilgileri raporlar. ör, checkpoint aktivitesi | INFO | INFORMATION |
| FATAL | Mevcut oturumun iptal edilmesine neden olan hatayı bildirir. | ERR | ERROR |
| PANIC | Tüm veritabanı oturumlarının iptal edilmesine neden olan hatayı bildirir. | CRIT | ERROR |

### What to Log

#### `application_name`

{% include parameter_info.html parametre="application_name" %}

{% include callout.html content=" **application_name, `NAMEDATALEN` sabitinden daha az karakterde herhangi bir string olabilir (standart build'de 64 karakter).** Genellikle sunucuya bağlanan uygulama tarafından ayarlanır. Bu isim, `pg_stat_activity` view'ında görüntülenecek ve CSV günlük girişlerine dahil edilecektir. `log_line_prefix` parametresi ile normal günlük girişlerine de dahil edilebilir. Bu parametre değerinde yalnızca yazdırılabilir ASCII karakterleri kullanılabilir. Diğer karakterler soru işaretleriyle (?) değiştirilecektir." type="primary" %}

#### `debug_print_parse (boolean) / debug_print_rewritten (boolean) / debug_print_plan (boolean)`

{% include parameter_info.html parametre="debug_print_parse" %}

{% include parameter_info.html parametre="debug_print_rewritten" %}

{% include parameter_info.html parametre="debug_print_plan" %}

{% include callout.html content=" **Bu parametreler, çeşitli hata ayıklama çıktılarının yayınlanmasını sağlar.** İlgili parametre ayarlandığında; parse tree, query rewriter çıktısı ve execution planları yazılır. Bu mesajlar `LOG` mesajı seviyesinde yayınlanır ve sunucu günlüğünde görünürler ancak istemciye gönderilmezler. Bunu, `client_min_messages` ve / veya `log_min_messages` ayarlayarak değiştirebilirsiniz. Bu parametreler varsayılan olarak kapalıdır." type="primary" %}

#### `debug_pretty_print`

{% include parameter_info.html parametre="debug_pretty_print" %}

{% include callout.html content=" **Bu parametre ayarlandığında, `debug_pretty_print`, `debug_print_parse`, `debug_print_rewritten` ve `debug_print_plan` tarafından üretilen mesajları girintiler.** Bu parametre kapalıyken sağlanan 'kompakt' format daha okunabilirdir ancak çok daha uzun çıktılar verir. Varsayılan olarak açıktır." type="primary" %}

#### `log_checkpoints`

{% include parameter_info.html parametre="log_checkpoints" %}

{% include callout.html content=" **checkpoint ve restartpoint noktalarının sunucu günlüğüne kaydedilmesini sağlar.** Yazılan arabellek sayısı ve bunları yazmak için harcanan zaman dahil olmak üzere bazı istatistikler günlük mesajlarına dahil edilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan olarak kapalıdır." type="primary" %}

#### `log_connections`

{% include parameter_info.html parametre="log_connections" %}

{% include callout.html content=" **Her başarılı bağlantı girişimini günlüğe kaydeder.** Bu parametre oturum başlangıcında yalnızca süper kullanıcılar tarafından ayarlanabilir ve oturum içinde değiştirilemez. Varsayılan olarak kapalıdır." type="primary" %}

{% include note.html content=" psql gibi istemci programları, parola gerekip gerekmediğini belirlerken iki kez bağlanma denemesinde bulunur. Bu nedenle, yinelenen 'connection received' mesajları bir problem olduğu anlamına gelmez."%}

#### `log_disconnections`

{% include parameter_info.html parametre="log_disconnections" %}

{% include callout.html content=" **Oturum sonlandırmalarının günlüğe kaydedilmesini kontrol eder.** Günlük çıktısı, `log_connections` ile benzer olmakla birlikte oturum süresini de içerir. Bu parametre yalnızca süper kullanıcılar tarafından oturum başlangıcında değiştirebilir ve bir oturum içinde hiç değiştirilemez. Varsayılan olarak kapalıdır." type="primary" %}

#### `log_duration`

{% include parameter_info.html parametre="log_duration" %}

{% include callout.html content=" **Tamamlanan ifadelerin süresinin günlüğe kaydedilmesini kontrol eder.** Varsayılan olarak kapalıdır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Extended sorgu protokolü kullanan istemciler için Parse, Bind ve Execute adım süreleri bağımsız olarak günlüğe kaydedilir." type="primary" %}

{% include note.html content=" log_duration'ı etkinleştirmek ve log_min_duration_statement'ı sıfıra ayarlamak arasındaki fark, log_min_duration_statement değerini aşmanın sorgu metnini günlüğe kaydetmeye zorlamasıdır. log_duration açık ve log_min_duration_statement pozitif bir değere sahipse, tüm süreler günlüğe kaydedilirken sorgu metni yalnızca eşiği aşan ifadeler için dahil edilir. Bu davranış, yüksek yüklü kurulumlarda istatistik toplamak için yararlı olabilir."%}

#### `log_error_verbosity`

{% include parameter_info.html parametre="log_error_verbosity" %}

{% include callout.html content=" **Kaydedilen mesajların sunucu günlüğüne yazılan ayrıntı miktarını kontrol eder.** Bu parametre için geçerli değerler `TERSE`, `DEFAULT` ve `VERBOSE`'dur. Her biri görüntülenen mesajlara daha fazla alan ekler. `TERSE`; `DETAIL`, `HINT`, `QUERY` ve `CONTEXT` hata bilgilerinin günlüğe kaydını dahil etmez. `VERBOSE` çıktısı, `SQLSTATE` hata kodunu ve hatayı oluşturan kaynak kod dosyası adını, işlev adını ve satır numarasını içerir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_hostname`

{% include parameter_info.html parametre="log_hostname" %}

{% include callout.html content=" **Bağlantı günlüğü mesajları varsayılan olarak yalnızca bağlanan host'un IP adresini gösterir.** Bu parametrenin açılması, host adının da günlüğe kaydedilmesini sağlar. Bu, host adı çözümleme kurulumuna bağlı olarak performans kaybı getirebilir. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir." type="primary" %}

#### `log_line_prefix`

{% include parameter_info.html parametre="log_line_prefix" %}

{% include callout.html content=" **Her günlük satırı çıktısının başında olan printf-style dizesidir.** `%` karakterleri, aşağıda verilen durum bilgisiyle değiştirilen 'kaçış dizileri' (escape sequences) başlar. Tanınmayan kaçışlar yok sayılır. Diğer karakterler doğrudan günlük satırına kopyalanır. Bazı kaçışlar yalnızca oturum süreçleri tarafından tanınır ve ana sunucu süreci gibi background süreçler tarafından boş olarak değerlendirilir. Durum bilgileri, `%` ve opsiyondan arasında sayısal değişmez bir değer verilerek sola ve sağa hizalanabilir. Negatif bir değer, durum bilgisinin sağını boşlukla doldurarak minimum uzunluk sağlarken, pozitif değer solu boşluklarla doldurur. Doldurma, günlük dosyalarının okunabilirliği için kullanılabilir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan değer, bir zaman damgası ve süreç ID kaydeden `%m [%p]`'dir." type="primary" %}

| Kaçış | Etki | Session only |
|-------|--------|-------|
| `%a` | Uygulama adı | yes |
| `%u` | Kullanıcı adı | yes |
| `%d` | Veri tabanı adı | yes |
| `%r` | Uzak host adı veya IP adresi ve uzak port | yes |
| `%h` | Uzak host adı veya IP adresi | yes |
| `%b` | Backend tipi | no |
| `%p` | Süreç ID | no |
| `%t` | Milisaniyesiz zaman damgası | no |
| `%m` | Milisaniyeli zaman damgası | no |
| `%n` | Milisaniyeli zaman damgası ( Unix epoch olarak)| no |
| `%i` | Komut etiketi: oturumun mevcut komut türü | yes |
| `%e` | SQLSTATE hata kodu | no |
| `%c` | Oturum ID: aşağıdaki açıklamaya bakın | no |
| `%l` | Her oturum ve süreç için günlük satırının numarası, 1'den başlar. | no |
| `%s` | Süreç başlangıç ​​zaman damgası | no |
| `%v` | Sanal işlem kimliği (backendID/localXID) | no |
| `%x` | Transaction ID (atanmamışsa 0) | no |
| `%q` | Çıktı üretmez, oturum dışı süreçlere dizenin bu noktasında durmasını söyler, oturum süreçleri tarafından yok sayılır. | no |
| `%%` | Gerçek % | no |

{% include callout.html content=" Backend türü, [`pg_stat_activity`](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW) view'ındaki 'backend_type' sütununa karşılık gelir. Günlükte, bu view'da gösterilmeyen ek türler görünebilir.<br/><br/>

`%c` kaçışı, noktayla ayrılmış iki 4 baytlık hexadecimal sayıdan oluşan, neredeyse benzersiz bir oturum tanımlayıcısı yazdırır. Sayılar süreç başlama zamanı ve süreç ID'dir, bu nedenle `%c` bu öğeleri yazdırmanın alan tasarrufu sağlayan bir yoludur. Örneğin, pg_stat_activity'den oturum tanımlayıcısını oluşturmak için şu sorguyu kullanın:" type="primary" %}

```sql
SELECT to_hex(trunc(EXTRACT(EPOCH FROM backend_start))::integer) || '.' || to_hex(pid)
FROM pg_stat_activity;
```

{% include tip.html content=" log_line_prefix için boş olmayan bir değer ayarlarsanız, günlük satırının geri kalanından görsel ayrımı sağlamak için son karakteri boşluk yapmalısınız. Noktalama karakteri de kullanılabilir."%}

{% include tip.html content=" syslog kendi zaman damgasını ve süreç ID bilgilerini ürettiğinden, syslog'da loglama yapıyorsanız bu kaçışları dahil etmek istemeyebilirsiniz."%}

{% include tip.html content=" **%q** kaçışı, kullanıcı veya veritabanı adı gibi yalnızca oturum (backend) bağlamında geçerli bilgileri dahil ederken kullanışlıdır. Örnek:<br/><br/>

log_line_prefix = '%m [%p] %q%u@%d/%a '"%}

#### `log_lock_waits`

{% include parameter_info.html parametre="log_lock_waits" %}

{% include callout.html content=" **Bir oturum, kilit almak için [`deadlock_timeout`](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-DEADLOCK-TIMEOUT) süresinden daha uzun süre beklediğinde bir günlük mesajı üretilip üretilmeyeceğini kontrol eder.** Bu parametre, kilit beklemelerinin düşük performansa neden olup olmadığını belirlenirken kullanışlıdır. Varsayılan olarak kapalıdır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_parameter_max_length`

{% include parameter_info.html parametre="log_parameter_max_length" %}

{% include callout.html content=" **Sıfırdan büyükse, hatasız ifade loglama mesajıyla kaydedilen her bir bağlama parametresi (bind parameter) değeri bu kadar bayt'a kırpılır.** `0`, bağlama parametrelerinin günlüğe kaydedilmesini devre dışı bırakır. `-1` (varsayılan), bağlama parametrelerinin olduğu gibi kaydedilmesine izin verir. Bu değer birimsiz belirtilirse bayt olarak alınır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu ayar yalnızca `log_statement`, `log_duration` ve ilgili ayarların bir sonucu olarak yazdırılan günlük mesajlarını etkiler. Bu ayarın `0` olmayan değerleri, özellikle parametreler binary formda gönderildiyse, metne dönüştürme gerekli olduğundan bazı ek yükler getirir." type="primary" %}

#### `log_parameter_max_length_on_error`

{% include parameter_info.html parametre="log_parameter_max_length_on_error" %}

{% include callout.html content=" **Sıfırdan büyükse, hata mesajlarında raporlanan her bağlama parametresi (bind parameter) değeri bu kadar bayt'a kırpılır.** `0` (varsayılan), hata mesajlarına bağlama parametrelerinin dahil edilmesini devre dışı bırakır. `-1`, bağlama parametrelerinin tam olarak yazdırılmasına izin verir. Bu değer birimsiz belirtilirse bayt olarak alınır.

Bu ayarın sıfır olmayan değerleri ek yük getirir. Çünkü PostgreSQL, sonuçta bir hata olsun ya da olmasın parametre değerlerinin metinsel temsillerini her bir ifadenin başlangıcında bellekte saklama ihtiyacı duyar. Bağlama parametreleri binary formda gönderildiğinde, metin olarak gönderildiklerine göre ek yük daha fazladır, çünkü ilk durum veri dönüştürme gerektirirken ikincisi yalnızca dizenin kopyalanmasını gerektirir." type="primary" %}

#### `log_statement`

{% include parameter_info.html parametre="log_statement" %}

{% include callout.html content=" **Hangi SQL ifadelerinin günlüğe kaydedileceğini kontrol eder.** Alabileceği değerler; `none` (kapalı), `ddl`, `mod` ve `all` (tüm ifadeler) 'dir. `ddl`; CREATE, ALTER ve DROP ifadeleri gibi tüm veri tanımlama ifadelerini günlüğe kaydeder. `mod`, tüm `ddl` ifadelerinin yanı da INSERT, UPDATE, DELETE, TRUNCATE ve COPY FROM gibi veri değiştirme ifadelerini günlüğe kaydeder. PREPARE, EXECUTE ve EXPLAIN ANALYZE deyimleri de, içerdikleri komutlar uygun türde ise günlüğe kaydedilir. Genişletilmiş sorgu (extended query) protokolü kullanan istemciler için günlüğe kaydetme, bir Execute mesajı alındığında gerçekleşir ve Bağlama (Bind) parametrelerinin değerleri dahil edilir.

Öntanımlı ayar `none`'dir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include tip.html content=" Basit sözdizimi hataları içeren ifadeler `log_statement = all` ayarıyla bile günlüğe kaydedilmez. Çünkü günlük mesajı, yalnızca ifade türünü belirlemek için temel parsing yapıldıktan sonra yayınlanır. Genişletilmiş sorgu (extended query) protokolü durumunda, bu ayar aynı şekilde Execute aşamasından önce (yani parse analizi veya planlama sırasında) başarısız olan ifadeleri günlüğe kaydetmez. Bu tür ifadeleri günlüğe kaydetmek için `log_min_error_statement` parametresini `ERROR` veya daha düşük olarak ayarlayın."%}

#### `log_replication_commands`

{% include parameter_info.html parametre="log_replication_commands" %}

{% include callout.html content=" **Her replikasyon komutunun sunucu günlüğüne kaydedilmesini sağlar.** Replikasyon komutu hakkında daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/protocol-replication.html) . Öntanımlı değeri `off`. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_temp_files`

{% include parameter_info.html parametre="log_temp_files" %}

{% include callout.html content=" **Geçici dosya adlarının ve boyutlarının günlüğe kaydedilmesini kontrol eder.** Sıralamalar, hash'ler ve geçici sorgu sonuçları için geçici dosyalar oluşturulabilir. Bu ayarla etkinleştirilirse, silinen her geçici dosya için bir günlük girişi yayınlanır. Sıfır değeri tüm geçici dosya bilgilerini günlüğe kaydederken, pozitif değerler yalnızca boyutu belirtilen veri miktarından büyük veya bu miktara eşit olan dosyaları günlüğe kaydeder. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı `-1` ayarı, böyle bir günlük kaydını devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `log_timezone`

{% include parameter_info.html parametre="log_timezone" %}

{% include callout.html content=" **Sunucu günlüğüne yazılan zaman damgaları için kullanılan saat dilimini ayarlar.** Parametre değeri [TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE)'dan farklı olarak küme çapındadır. Böylece tüm oturumlar zaman damgalarını tutarlı bir şekilde rapor eder. Yerleşik varsayılan `GMT`'dir, ancak bu genellikle *postgresql.conf*'ta geçersiz kılınır; initdb, sistem ortamına karşılık gelen bir ayar kuracaktır. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-TIMEZONES). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### CSV Formatında Log Çıktısı Kullanma

`log_destination` listesine `csvlog`'un dahil edilmesi, günlük dosyalarını bir veritabanı tablosuna aktarmak için kullanışlı bir yol sağlar. Bu parametre şu sütunlarla virgülle ayrılmış değerler (CSV) biçiminde günlük satırları yayınlar: milisaniyeli zaman damgası, kullanıcı adı, veritabanı adı, süreç ID; istemci host: bağlantı noktası numarası (port), oturum ID, herbir oturum için satır numarası, komut etiketi, oturum başlangıç ​​zamanı, sanal transaction ID, normal transaction ID, hata düzeyi, `SQLSTATE` kodu, hata mesajı, hata mesajı ayrıntısı, ipucu, hataya neden olan dahili sorgu (varsa), oradaki hata pozisyonunun karakter sayısı, hata bağlamı, hataya yol açan kullanıcı sorgusu (varsa ve `log_min_error_statement` tarafından etkinleştirilmişse), oradaki hata konumunun karakter sayısı, PostgreSQL kaynak kodundaki hatanın konumu (`log_error_verbosity` ayrıntılı olarak ayarlanmışsa), uygulama adı ve backend türü. CSV biçimli günlük çıktısını depolamak için örnek bir tablo tanımı:

```sql
CREATE TABLE postgres_log
(
  log_time timestamp(3) with time zone,
  user_name text,
  database_name text,
  process_id integer,
  connection_from text,
  session_id text,
  session_line_num bigint,
  command_tag text,
  session_start_time timestamp with time zone,
  virtual_transaction_id text,
  transaction_id bigint,
  error_severity text,
  sql_state_code text,
  message text,
  detail text,
  hint text,
  internal_query text,
  internal_query_pos integer,
  context text,
  query text,
  query_pos integer,
  location text,
  application_name text,
  backend_type text,
  PRIMARY KEY (session_id, session_line_num)
);
```

{% include callout.html content=" Bu tabloya bir günlük dosyası aktarmak için, COPY komutunu kullanın:" type="primary" %}

```sql
COPY postgres_log FROM '/full/path/to/logfile.csv' WITH csv;
```

{% include callout.html content=" [file_fdw](https://www.postgresql.org/docs/current/file-fdw.html) modülünü kullanarak dosyaya yabancı bir tablo olarak erişmek de mümkündür.<br/><br/> 

CSV günlük dosyalarını içe aktarmayı basitleştirmek için yapmanız gereken birkaç şey vardır:" type="primary" %}

1. Günlük dosyalarınız için tutarlı, öngörülebilir bir adlandırma düzeni sağlamak için `log_filename` ve `log_rotation_age` parametrelerini ayarlayın. Bu, dosya adını öngörmenizi ve bir günlük dosyasının ne zaman tamamlandığını ve dolayısıyla içe aktarılmaya hazır olduğunu bilmenizi sağlar.
2. Günlük dosyası adının tahmin edilmesini zorlaştıracağından boyut tabanlı günlük rotasyonunu devre dışı bırakmak için `log_rotation_size` değerini 0 olarak ayarlayın.
3. `log_truncate_on_rotation` öğesini açık olarak ayarlayın, böylece eski günlük verileri aynı dosyadaki yenileriyle karıştırılmaz.
4. Yukarıda verilen tablo tanımı, bir birincil anahtar spesifikasyonu içerir. Bu, aynı bilgilerin yanlışlıkla iki kez içe aktarılmasına karşı koruma sağlamak için yararlıdır. `COPY` komutu, içe aktardığı tüm verileri tek seferde işler, bu nedenle herhangi bir hata içe aktarmanın tamamının başarısız olmasına neden olur. Tam olmayan bir günlük dosyasını içe aktarırsanız daha sonra tamamlandığında dosyayı tekrar içe aktardığınızda birincil anahtar ihlali içe aktarmanın başarısız olmasına neden olur. İçe aktarmadan önce, günlük tamamlanana ve kapanana kadar bekleyin. Ayrıca bu prosedür tamamen yazılmamış bir parçalı satırın yanlışlıkla içe aktarılmasına karşı koruma sağlayarak `COPY`'nın başarısız olmasına neden olur.

### Süreç Başlığı

Bu başlık altında verilen ayarlar, sunucu süreçlerinin süreç başlıklarının değiştirilmesini kontrol eder. Süreç başlıkları ps ve Windows Process Explorer gibi programlar kullanılarak görüntülenir. Ayrıntılar için bkz [](https://www.postgresql.org/docs/current/monitoring-ps.html).

#### `cluster_name`

{% include parameter_info.html parametre="cluster_name" %}

{% include callout.html content=" **Bu parametre, veritabanı kümesini tanımlayan bir ad belirtir.** Küme adı, bu kümedeki sunucu süreçlerinin süreç başlığında görünür. Ayrıca, bir standby bağlantısı için varsayılan uygulama adıdır. bkz. [synchronous_standby_names](mydoc_replication.html#primary-server)<br/><br/>

Ad, `NAMEDATALEN`'den daha az karakterde herhangi bir dize olabilir (standart build'de 64 karakter). `cluster_name` parametre değerinde yalnızca yazdırılabilir ASCII karakterler kullanılabilir. Diğer karakterler soru işareti (?) ile değiştirilecektir. Bu parametre boş dizeye ' ' (varsayılan böyle) ayarlanırsa ad gösterilmez. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

#### `update_process_title`

{% include parameter_info.html parametre="update_process_title" %}

{% include callout.html content=" **Sunucu tarafından her yeni SQL komutu alındığında süreç başlığının güncellenmesini sağlar.** Bu ayar, çoğu platformda varsayılan olarak açıktır. Windows'ta süreç başlığının büyük güncellenme ek yükü nedeniyle varsayılan olarak kapalıdır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-logging.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
