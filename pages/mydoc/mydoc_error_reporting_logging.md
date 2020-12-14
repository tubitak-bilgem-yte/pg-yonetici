---
title: "Hata Raporlama ve Logging"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 14, 2020
summary: "Hata Raporlama ve Logging"
sidebar: mydoc_sidebar
permalink: mydoc_error_reporting_logging.html
folder: mydoc
---

## Hata Raporlama ve Logging

### Where to Log

{% include callout.html content="**`log_destination (string)`**: PostgreSQL, sunucu mesajlarını günlüğe kaydetmek için stderr, csvlog ve syslog gibi çeşitli yöntemleri destekler. Windows'ta olay günlüğü de desteklenmektedir. Bu parametre değeri, virgülle ayrılmış istenen günlük hedefleri listesidir. Varsayılan, yalnızca stderr'ı günlüğe yazar. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

`log_destination` csvlog içeriyorsa; günlük kayıtları, günlükleri programlara yüklemek için uygun olan virgülle ayrılmış değerler (comma separated value-CSV) formatında yazılır. Ayrıntılar için [CSV-Format Log Çıktısı Kullanma](mydoc_error_reporting_logging.html#csv-format-log-%C3%A7%C4%B1kt%C4%B1s%C4%B1-kullanma) bölümüne bakın. CSV formatında günlük çıktısı oluşturmak için `logging_collector` etkinleştirilmelidir.<br/><br/>

`log_destination` stderr veya csvlog içerdiğinde; logging collector tarafından kullanımda olan günlük dosyalarının konumunu ve ilişkili günlük kaydı hedefini kaydetmek için `current_logfiles` dosyası oluşturulur. Bu, veritabanı tarafından kullanımda olan günlükleri bulmak için uygun bir yol sağlar. Bu dosyanın içeriğinin bir örneği:" type="primary" %}

```bash
stderr log/postgresql.log
csvlog log/postgresql.csv
```

{% include callout.html content=" `current_logfiles`, rotasyonun bir etkisi olarak yeni bir günlük dosyası oluşturulduğunda ve `log_destination` yeniden yüklendiğinde tekrar oluşturulur. `current_logfiles`, `log_destination` stderr veya csvlog içermediğinde ve logging collector devre dışı bırakıldığında kaldırılır." type="primary" %}

{% include note.html content=" Çoğu Unix sistemde, **log_destination** için syslog opsiyonundan yararlanmak için sisteminizin syslog arka plan programının yapılandırmasını değiştirmeniz gerekir . Çalışmasını sağlamak için syslog daemon yapılandırma dosyasına şu şekilde ekleme yapmanız gerekecek:<br/><br/>
**local0. */var/log/postgresql**
"%}

{% include callout.html content="**`logging_collector (boolean)`**: Bu parametre, stderr'e gönderilen günlük mesajlarını yakalayıp bunları günlük dosyalarına yönlendiren bir arka plan süreci olan **logging collector**'ı etkinleştirir. Bu yaklaşım, bazı mesaj türleri syslog çıktısında görünmeyebileceğinden genellikle syslog'a kaydetmekten daha kullanışlıdır. Dynamic-linker hata mesajları, `archive_command` gibi komut dosyaları tarafından üretilen hata mesajları genel örneklerdir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include note.html content=" Logging collector kullanmadan stderr'ı log'lamak mümkündür. Günlük mesajları sadece sunucunun stderr'ının yönlendirildiği yere gidecektir. Ancak, günlük dosyalarını döndürmek için uygun bir yol sağlamadığından, bu yöntem yalnızca düşük günlük hacimleri için uygundur. Ayrıca, logging collector kullanmayan bazı platformlarda, aynı günlük dosyasına aynı anda yazan birden çok süreç birbirinin çıktısının üzerine yazabileceğinden günlük kaydının kaybolmasına veya bozulmasına neden olabilir."%}

{% include note.html content=" Logging collector, mesajları asla kaybetmeyecek şekilde tasarlanmıştır. Bu, aşırı yük durumunda, collector geride kalıp ek günlük mesajları göndermeye çalıştığında sunucu süreçlerinin bloklanabileceği anlamına gelir. Bunun tersine; syslog, mesajları yazamıyorsa drop etmeyi tercih eder. Bu da bu gibi durumlarda bazı mesajları günlüğe kaydetmede başarısız olabileceği, ancak sistemin geri kalanını bloklamayacağı anlamına gelir."%}

{% include callout.html content="**`log_directory (string)`**: Bu parametre `logging_collector` etkinleştirildiğinde günlük dosyalarının oluşturulacağı dizini belirler. Mutlak bir yol olarak veya küme veri dizinine ( data directory ) göre belirtilebilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırındab ayarlanabilir. Öntanımlı değeri `log`'dur." type="primary" %}

{% include callout.html content="**`log_filename (string)`**: Bu parametre `logging_collector` etkinleştirildiğinde oluşturulan günlük dosyalarının adlarını ayarlar. Değer `strftime` kalıbı olarak işlenir. Zamanla değişen dosya adları `%` kaçışları ile belirtilir. Saat dilimine bağlı `%` kaçışları varsa, hesaplama `log_timezone` ile belirtilen bölgede yapılır. Doğrudan sistemin `strft` zamanı kullanılmadığı için platform spesifik uzantılar çalışmaz. Varsayılan, `postgresql-%Y-%m-%d_%H%M%S.log` şeklindedir.<br/><br/>

`log_destination`'da CSV formatında çıktı etkinleştirildiğinde zaman damgalı günlük dosyası adına `.csv` eklenir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`log_file_mode (integer)`**: BBu parametre, `logging_collector` etkinleştirildiğinde, Unix sistemlerde günlük dosyalarının izinlerini ayarlar. (Windows'da bu parametre yoksayılır.) Parametre değerinin, `chmod` ve `umask` sistem çağrıları tarafından kabul edilen formatta sayısal bir mod olması beklenir.<br/><br/>

Varsayılan izinler `0600`'dır, yani yalnızca sunucu sahibi günlük dosyalarını okuyabilir ve yazabilir. Bir diğer yaygın kullanım ayarı `0640`'tır ve sahip grubunun üyelerinin dosyaları okumasına izin verir. Böyle bir ayarı kullanmak için, dosyaları küme veri dizininin dışında bir yerde depolamak için `log_directory`'yi değiştirmeniz gerekecektir. Her durumda, hassas veriler içerdiği için günlük dosyalarını herkes tarafından okunabilir hale getirmek akıllıca değildir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`log_rotation_age (integer)`**: Bu parametre, `logging_collector` etkinleştirildiğinde tek bir günlük dosyasının maksimum kullanım süresini belirler ve bu değerden sonra yeni bir günlük dosyası oluşturulur. Bu değer birimsiz belirtilirse dakika olarak alınır. Varsayılan 24 saattir. Yeni günlük dosyalarının zamana dayalı olarak oluşturulmasını devre dışı bırakmak için 0 olarak ayarlayın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`log_rotation_size (integer)`**: Bu parametre, `logging_collector` etkinleştirildiğinde tek bir günlük dosyasının maksimum boyutunu belirler. Bu miktarda veri bir günlük dosyasına gönderildikten sonra, yeni bir günlük dosyası oluşturulacaktır. Bu değer birimsiz belirtilirse kilobayt olarak alınır. Öntanımlı değeri 10 megabayttır. Yeni günlük dosyalarının boyuta dayalı olarak oluşturulmasını devre dışı bırakmak için 0 olarak ayarlayın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`log_truncate_on_rotation (boolean)`**: Bu parametre, `logging_collector` etkinleştirildiğinde PostgreSQL günlük dosyalarına ekleme yapmak yerine mevcut günlük dosyasını truncate (üzerine yazmasına) eder. Truncate yalnızca zamana dayalı rotasyon sebepli yeni dosya açılırken meydana gelir, sunucu başlangıcında veya boyuta dayalı rotasyon sırasında değil. Kapalı olduğunda, var olan dosyalara her durumda ekleme yapılacaktır. Örneğin, bu ayarı `postgresql-%H.log` gibi bir `log_filename` ile birlikte kullanılması 24 saatlik günlük dosyalarının oluşturulup ve döngüsel olarak bunların üzerine yazılmasıyla sonuçlanır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir.<br/><br/>

Örnek: 7 günlük `server_log.Mon`, `server_log.Tue`, vb. şekilde günlük dosyaları tutmak ve geçen haftanın günlüğünün üzerine bu haftanın günlüğünü otomatik olarak yazmak için, `log_filename` değerini `server_log.%a`, `log_truncate_on_rotation = on` ve `log_rotation_age = 1440` olarak ayarlayın.<br/><br/>

Örnek: Her saatte bir günlük dosyası olarak 24 saatlik günlükler ve günlük dosyası boyutu 1 GB'ı aşarsa daha erken rotasyon için `log_filename` değerini `server_log.%H%M`, `log_truncate_on_rotation = on`, `log_rotation_age = 60` ve `log_rotation_size = 1000000` olarak ayarlayın. `log_filename`'in `%M` içermesi, meydana gelebilecek boyuta dayalı rotasyonlarda mevcut saatlik dosya adından farklı bir dosya adı seçmek içindir" type="primary" %}

{% include callout.html content="**`syslog_facility (enum)`**: Bu parametre syslog'da günlük kaydı etkinleştirildiğinde kullanılacak syslog 'facility' belirler. `LOCAL0`, `LOCAL1`, `LOCAL2`, `LOCAL3`, `LOCAL4`, `LOCAL5`, `LOCAL6`, `LOCAL7` arasından seçim yapabilirsiniz. Öntanımlı değeri `LOCAL0`'dır. Ayrıca sisteminizin syslog daemon belgelerine bakın. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`syslog_ident (string)`**: Bu parametre syslog'da günlük kaydı etkinleştirildiğinde, syslog günlüklerinde PostgreSQL mesajlarını tanımlamak için kullanılan program adını belirler. Varsayılan, `postgres`'tir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`syslog_sequence_numbers (boolean)`**: syslog'da günlük kaydı etkinleştirildiğinde ve bu parametre `on` (varsayılan) ise her mesajın önüne artan bir sıra numarası eklenir ([2] gibi). Bu, birçok syslog uygulamasının varsayılan olarak gerçekleştirdiği '--- last message repeated N times ---' bastırmasını engeller. Daha modern syslog uygulamalarında tekrarlanan mesaj bastırma yapılandırılabildiğinde bu gerekli olmayabilir. Tekrarlanan mesajları bastırmak istiyorsanız bunu kapatabilirsiniz.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`syslog_split_messages (boolean)`**: Bu parametre syslog'da günlük kaydı etkinleştirildiğinde mesajların syslog'a nasıl teslim edileceğini belirler. `on` (varsayılan) ayarında, mesajlar satırlara bölünür ve uzun satırlar, geleneksel syslog uygulamaları için tipik boyut sınırı olan 1024 bayta sığacak şekilde bölünür. `off` ayarında, PostgreSQL sunucusu günlük mesajları syslog servisine olduğu gibi teslim edilir ve büyük mesajlarla başa çıkmak syslog servisine bırakılır.<br/><br/>

Eğer syslog bir metin dosyasına kaydediliyorsa etki her iki şekilde de aynı olacaktır ve çoğu syslog uygulaması büyük iletileri işleyemeyeceği veya bunları işlemek için özel olarak yapılandırılması gerekeceği için en iyisi ayarı açık bırakmaktır. syslog nihayetinde başka bir ortama yazıyorsa, mesajları mantıksal olarak bir arada tutmak gerekli veya daha yararlı olabilir.<br/><br/>

Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`event_source (string)`**: event log'a loglama etkinleştirildiğinde bu parametre, günlükteki PostgreSQL mesajlarını tanımlamak için kullanılan program adını belirler. Varsayılan, `PostgreSQL`'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### When to Log

{% include callout.html content="**`log_min_messages (enum)`**: Sunucu günlüğüne hangi mesaj seviyelerinin yazılacağını kontrol eder. Geçerli değerler `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` ve `PANIC`'dir. Her seviye, onu takip eden tüm seviyeleri kapsar. Seviye ne kadar düşük olursa, günlüğe o kadar az mesaj gönderilir. Varsayılan, `WARNING`'dır. `LOG`'un burada [client_min_messages](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-MIN-MESSAGES)'dakinden farklı bir sıralamada olduğunu unutmayın. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`log_min_error_statement (enum)`**: Hata durumuna neden olan SQL ifadelerinin sunucu günlüğüne kaydedilmesini kontrol eder. Verilen seviyede veya üzerinde hata oluşturan tüm SQL ifadeleri günlüğe kaydedilir. Geçerli değerler `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` ve `PANIC`'dir. Varsayılan `ERROR`'dır. Bu hataya, günlük mesajlarına, fatal error'lara ve paniklere neden olan SQL ifadelerinin günlüğe kaydedileceği anlamına gelir. Başarısız ifadelerin günlüğe kaydedilmesini kapatmak için bu parametreyi `PANIC` olarak ayarlayın. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`log_min_duration_statement (integer)`**: 
En az bu parametrede belirtilen sürede tamamlanan tüm ifadeleri günlüğe kaydeder. Örneğin, 250 ms ayarı, 250 ms ve daha uzun süre çalışan tüm SQL ifadelerini günlüğe kaydeder. Bu parametrenin etkinleştirilmesi, uygulamalarınızdaki optimize edilmemiş sorguların izlenmesinde yardımcı olur. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Bunu 0 olarak ayarlamak, çalışan tüm SQL ifade sürelerini yazdırır. -1 (varsayılan) değeri bu davranışı devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu parametre, `log_min_duration_sample` parametresini geçersiz kılar, yani bu ayarı aşan süreye sahip sorgular örneklemeye tabi değildir ve her zaman günlüğe kaydedilir.<br/><br/>

Genişletilmiş sorgu protokolü kullanan istemciler için Parse, Bind ve Execute adımlarının süreleri bağımsız olarak günlüğe kaydedilir." type="primary" %}

{% include note.html content=" Bu parametre `log_statement` ile birlikte kullanırken, `log_statement` nedeniyle günlüğe kaydedilen ifadeler süre günlüğü mesajında ​​tekrarlanmayacaktır."%}

{% include callout.html content="**`log_min_duration_sample (integer)`**: En belirtilen süre boyunca çalışan tamamlanmış ifadelerin süresinin örneklenmesine (sampling) edilmesine izin verir. `log_statement_sample_rate` tarafından kontrol edilen örnekleme oranıyla yürütülen ifadelerin bir alt kümesi için `log_min_duration_statement` ile aynı türden günlük girişleri üretir. Örneğin, 100 ms ayarları, 100 ms ve daha uzun süre çalışan tüm SQL ifadeleri örnekleme için dikkate alacaktır. Bu parametrenin etkinleştirilmesi, tüm sorguları günlüğe kaydedemeyecek kadar yüksek trafik olduğunda faydalı olabilir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Bunu 0 olarak ayarlamak tüm ifade sürelerini örnekler. -1 (varsayılan) örnekleme ifadesi sürelerini devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir.<br/><br/>

Bu ayar, `log_min_duration_statement`'tan daha düşük önceliğe sahiptir. `log_min_duration_statement`'ı aşan süredeki ifadeler örneklemeye tabi değildir ve her zaman günlüğe kaydedilir.<br/><br/>

`log_min_duration_statement` için verilen notlar bu ayar için de geçerlidir." type="primary" %}

{% include callout.html content="**`log_statement_sample_rate (floating point)`**: Günlüğe kaydedilecek `log_min_duration_sample`'ı aşan süredeki ifadelerin oranını belirler. Örnekleme stokastiktir; örneğin 0.5, istatistiksel olarak herhangi bir ifadenin günlüğe kaydedilme ihtimalinin 1/2 olduğu anlamına gelir. Varsayılan değer 1.0'dır, tüm örneklenmiş ifadeler günlüğe kaydedilir. Bunu 0 olarak ayarlamak örneklenmiş ifade süresi günlük kaydını devre dışı bırakır. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`log_transaction_sample_rate (floating point)`**: Diğer nedenlerle günlüğe kaydedilen ifadelere ek olarak, tüm ifadeleri günlüğe kaydedilen transaction'ların oranını ayarlar. İfadelerinin sürelerine bakılmaksızın her yeni transaction için geçerlidir. Örnekleme stokastiktir, örneğin 0.1, istatistiksel olarak herhangi bir işlemin günlüğe kaydedilme ihtimalinin onda bir olduğu anlamına gelir. `log_transaction_sample_rate`, transaction'ların örneğini oluşturmada yardımcı olabilir. Varsayılan 0'dır, herhangi bir ek transaction'dan gelen ifadelerin günlüğe kaydedilmemesi anlamına gelir. Bunu 1 olarak ayarlamak, tüm transaction'ların tüm ifadelerini günlüğe kaydeder. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

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

### CSV Formatında Log Çıktısı Kullanma

### Süreç Başlığı

{% include links.html %}
