---
title: "Parametreleri Ayarlama"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 26, 2020
summary: "Parametreleri Ayarlama"
sidebar: mydoc_sidebar
permalink: mydoc_paremetreleri_ayarlama.html
folder: mydoc
---

## Parametreleri Ayarlama

### Parametre İsimleri ve Değerleri

Parametre isimleri büyük / küçük harfe duyarlı değildir. Her parametre Boolean, String, Numeric, Floating Point ve Enumerated (enum) olmak üzere beş tip değerden birini alır. Tip, parametrenin ayarlanması için sözdizimini belirler:

- **Boolean:** Bu değerler, tümü büyük / küçük harfe duyarlı olmayan `on`, `off`, `true`, `false`, `yes`, `no`, `1`, `0` veya bunlardan birinin unambiguous prefix'i olarak yazılabilir.

- **String:** Bu tip değerler genelde tek tırnak içinde verilir ancak değer içinde tek tırnak işaretleri içeriyorsa bunlar çiftlenmelidir. Sayı ve tanımlayıcı değerlerinde tırnak işaretleri ihmal edilebilir. Bir SQL anahtar kelimesiyle eşleşen değer bazı durumlarda tırnak işareti gerektirir.

- **Numeric (integer ve floating point):** Numeric parametreler tamsayı (integer) ve ondalıklı (floating-point) sayı formatlarında belirtilebilir. Ondalıklı değerler, parametre integer tipindeyde en yakın tam sayıya yuvarlanır. Integer parametreler ondalılıklı olamaması kaydıyla hexadecimal ve octal değerler alabilir. hexadecimal girdiler dışında tırnak işaretleri gerekli değildir.

- **Numeric with Unit:** Bazı sayısal parametreler bellek ve zaman miktarlarını tanımlayan örtük bir birime sahiptir. Birim bytes, kilobytes, blocks (tipik olarak sekiz kilobayt), milisaniye, saniye veya dakika olabilir. Bu birimlerden biriyle süslenmemiş sayısal değerler, ayarın `pg_settings.unit`'ten referans alınan varsayılan birimini kullanır. Kolaylık sağlanması için ayarlar açıkça belirtilen bir birimle verilebilir. Örneğin bir zaman değeri '120 ms' şeklinde ifade edilebilir. Bu kullanım, mevcut parametre birimi ne olursa olsun dönüştürecektir. Bu özelliik, örnek kullanımda da görüldüğü gibi değeri bir string olarak tırnak işaretleriyle vererek kullanılır. Birim ismi büyük / küçük harfe duyarlıdır, sayısal değer ile birim arasında boşluk olabilir.
  - Geçerli bellek birimleri B (bayt), kB (kilobayt), MB (megabayt), GB (gigabayt) ve TB'dir (terabayt). Bellek birimleri için katlar 1000 değil 1024'tür.
  - Geçerli zaman birimleri `us` (mikrosaniye), `ms` (milisaniye), `s` (saniye), `min` (dakika), `h` (saat) ve `d` (gün).

{% include callout.html content=" Ondalıklı bir değer birimle belirtildiği durumda, varsa bir sonraki küçük birimin katına yuvarlanır. Örneğin 30,1 GB, 32319628902 B'ye değil 30822 MB'ye dönüştürülür. Parametre tamsayı tipindeyse birim dönüşümünden sonra tam sayıya yuvarlama gerçekleşir." type="info" %}

- **Enumerated:** Enumerated tipindeki parametreler string parametrelerle aynı şekilde yazılır. Bu değerler belirli bir değer kümesiyle sınırlandırılır. Bu parametre için izin verilen değerler `pg_settings.enumvals`'den bulunur. Enum parametre değerleri büyük / küçük harfe duyarlı değildir.

### Yapılandırma Dosyası Aracılığıyla Parametre Etkileşimi

Bu parametreleri ayarlamanın en temel yolu veri dizininde tutulan `postgresql.conf` dosyasından düzenlemektir. Veritabanı küme dizini başlatıldığında varsayılan bir kopya kurulur. Bu dosyanın neye benzeyebileceğine dair örnek:

```bash
# This is a comment
log_connections = yes
log_destination = 'syslog'
search_path = '"$user", public'
shared_buffers = 128MB
```

Her satırda bir parametre belirtilir. Parametre ismi ve değeri arasındaki eşittir işareti isteğe bağlıdır. Tırnak içine alınmış boşlular dışında parametre değerinde boşluklar önemsizdir, boş satırlar göz ardı edilir. Hash işaretleri (#), ilgili satırın yorum satırı olduğunu belirtir. Basit olmayan tanımlayıcılar ve sayısal olmayan parametre değerleri tek tırnak içerisinde verilmelidir. Bir parametre değerinde içinde tek tırnak kullanmak için, iki tırnak işareti (tercih edilen) veya `\'` kullanılır. Dosya aynı parametre için birden fazla kayıt içeriyorsa sonuncusu kabul edilir.

Bu yolla ayarlanan parametreler küme için varsayılan değerler olur. Aktif oturumlar tarafından görülen ayarlar, geçersiz kılınmadıkları sürece bu değerleri kullanır. Aşağıdaki bölümlerde yönetici ve kullanıcının bu varsayılanları geçersiz kılabileceği yollar açıklamaktadır.

Ana sunucu süreci SIGHUP sinyali aldığında yapılandırma dosyasını yeniden okur. SIGHUP sinyali `pg_ctl reload` komutu veya `pg_reload_conf ()` SQL fonksiyonu ile gönderilebilir. Ana sunucu süreci bu sinyali çalışan tüm sunucu süreçlerine yayar ve mevcut oturumlar da yeni değerlere adepte olur. Bu, o anda yürütülen istemci komutunun tamamlanmasından sonra gerçekleşecektir. Alternatif olarak, isteğinize göre sinyal doğrudan bir sunucu sürecine gönderilebilir. Bazı parametreler yalnızca sunucu başlangıcında ayarlanabilir. Yapılandırma dosyasındaki alanlarda yapılan herhangi bir değişiklik, sunucu yeniden başlatılıncaya kadar yok sayılacaktır. Yapılandırma dosyasındaki geçersiz parametre ayarları benzer şekilde SIGHUP işlemi sırasında öenmsenmez ancak günlüğe kaydedilir.

*postgresql.conf* PostgreSQL veri dizinine ek olarak *postgresql.conf* ile aynı formatta, otomatik düzenlenme istekleriniz için `postgresql.auto.conf` dosyasını içerir. *postgresql.auto.conf* [ALTER SYSTEM](https://www.postgresql.org/docs/current/sql-altersystem.html) komutuyla verilen ayarları içerir. Bu dosya içindeki ayarlar *postgresql.conf* içindeki ayarları geçersiz kılar.

*postgresql.auto.conf* dosyasını harici araçlar da değiştirebilir. Eşzamanlı çalıştırılan `ALTER SYSTEM` komutu bu değişiklikler üzerine yazabileceğinden sunucu çalışırken bunu yapmanız önerilmez. Bu araçlar, ALTER SYSTEM'in yapacağı gibi, yeni ayarları sona ekleyebilir veya tekrarlanan ayarları ve yorumları kaldırabilir.

Yapılandırma dosyalarındaki değişiklikler önceden test edilmek istendiğin veya SIGHUP sinyalinden istenen etkiler alınmadığı durumlarda sorunları tespit etmek için `pg_file_settings` sistem view'ı kullanılabilir.

### SQL Aracılığıyla Parametre Etkileşimi

PostgreSQL yapılandırma varsayılanları kurmak için üç SQL komutu sağlar. Daha önce bahsedilen `ALTER SYSTEM` komutu, global varsayılanları değiştirmek için SQL ile erişilebilir yöntem sağlar. İşlevsel olarak *postgresql.conf*'u düzenlemekle eşdeğerdir. Bu komuta ek, herbir veritabanı ve rol için varsayılanların ayarlanabilmesini sağlayan iki komut vardır:

- `ALTER DATABASE`, global ayarların veritabanı bazında geçersiz kılınması için kullanılır.
- `ALTER ROLE`, kullanıcıya özel değerler ile hem global hem de veritabanı bazında ayarların geçersiz kılınması için kullanılır.

{% include note.html content="`ALTER DATABASE` ve `ALTER ROLE` ile yapılan ayarlamalar yalnızca yeni veritabanı oturumu başlangıcında geçerli olur. Yapılandırma dosyalarından ve komut satırından aldığı değerleri geçersiz kılarak oturumun geri kalanı için varsayılanları oluştururlar. Bazı ayarlar sunucu başlatıldıktan sonra değiştirilemediği için bu ve aşağıda bahseceğimiz komutlardan biriyle ayarlanamaz."%}

İstemci, veritabanına bağlandığında oturum bazlı ayarlarla etkileşim kurması için PostgreSQL tarafından iki ek SQL komutu sağlanır:

- `SHOW`, parametrelerin mevcut değerinin incelenmesinde kullanılır. Bu komuta karşılık gelen işlev `current_setting (setting_name text)`dir.
- `SET`, bu parametrelerin oturum düzeyinde mevcut değerini değiştirmek için kullanılır. Yapılan değişikliklerin diğer oturumlar üzerinde etkisi yoktur. Bu komuta karşılık gelen fonksiyon `set_config(setting_name, new_value, is_local)`dir.

Oturum yerel değerlerini görüntüleme, değiştirme işlemlerinde `pg_settings` sistem view'ı da kullanılabilir:

- Bu view'ı sorgulamak, `SHOW ALL` kullanımına benzemekle birlikte daha fazla ayrıntı sağlar. `pg_settings` kullanımında filtre belirlemek ve diğer ilişkilerle join'lemek mümkün olduğundan daha esnektir.

- Bu view üzerinde `UPDATE` ile `setting` sütununu güncellemek `SET` komutunu uygulamakla eşdeğerdir. Örneğin,

```bash
SET configuration_parameter TO DEFAULT;
```

ifadesinin eş değeri,

```bash
UPDATE pg_settings SET setting = reset_val WHERE name = 'configuration_parameter';
```

### Kabuk Aracılığıyla Parametre Etkileşimi

Veritabanı ve rol düzeyinde global varsayılanlar ayarlamaya ek olarak, PostgreSQL'e ayarları kabuk imkanlarını kullanarak aktarabilirsiniz.

- Sunucu başlatılırken, parametre ayarları `-c` komut satırı parametresi ile `postgres` komutuna verilir. Bu yolla yapılan ayarlar, *postgresql.conf* ve `ALTER SYSTEM` ile yapılan ayarlamarı geçersiz kılar, böylece sunucuyu yeniden başlatmadan global olarak değiştirilemez. Örnek kullanım,

```bash
postgres -c log_connections=yes -c log_destination='syslog'
```

İstemci libpq aracılığıyla oturumu başlatırken, parametre ayarları `PGOPTIONS` ortam değişkeniyle belirtilir. Bu şekilde kurulan ayarlar ilgili oturum boyunca varsayılanları teşkil ederken diğer oturumları etkilemez. Tarihsel nedenlerden dolayı `PGOPTIONS`'ın formatı, `postgres` komutunu başlatırken kullanılan formata benzer şekilde `-c` bayrağı belirtilmelidir. Örnek kullanım,

```bash
env PGOPTIONS="-c geqo=off -c statement_timeout=5min" psql
```

### Yapılandırma Dosyası İçeriklerini Yönetme

PostgreSQL, kompleks *postgresql.conf* dosyalarını alt dosyalara bölmek için çeşitli özellikler sağlar. Bu özellikler, özellikle birbiriyle ilişkili olan ancak farklı yapılandırma ayarlarına sahip sunucuları yönetirken kullanışlıdır.

*postgresql.conf* dosyası içerdiği parametre ayarlarına ek olarak başka bir dosya belirten `include direktifleri` içerebilir. *include* yoluyla verilen dosya, bu nokta da yapılandırma dosyasına eklenmiş gibi okunacak ve işlenecektir. Bu özellik, bir yapılandırma dosyasının fiziksel olarak ayrı parçalara bölünmesine olanak tanır. Direktif dahil etme basitçe şu şekilde yapılır:

```bash
include 'filename'
```

{% include callout.html content=" Dosya adı mutlak bir yol değilse yapılandırma dosyasını içeren dizine göre referans alınır." type="info" %}

Başvurulan dosyanın mevcut olmadığı veya okunamadığı durumlar için *include direktifi* ile aynı şekilde davranan `include_if_exists` direktif de verilebilir. *include* bunu bir hata koşulu olarak kabul eder, `include_if_exists` günlüğe bir mesajı yazarak verilen yapılandırma dosyasını işlemeye devam eder.

*postgresql.conf* dosyasında, dahil edilecek tüm yapılandırma dosyalarının bulunduğu bir dizini belirten `include_dir` direktifi verilebilir. Şu şekilde,

```bash
include_dir 'directory'
```

Mutlak olmayan şekilde verilen dizin isimleri, referans yapılandırma dosyalarını içeren dizine göre ilişkilendirilir. Belirtilen dizin içindeki `.conf` sonekiyle biten dosyalar dahil edilecektir. `.` ile başlayan dosyalar bazı platformlarda gizlendiği için bu tür dosyalar hataları önlemek adına görmezden gelinir. *include_dir* içindeki dosyalar, dosya ismi sırasına göre işlenir (C locale ayar kurallarına göre; sayılar harflerden, büyük harfler küçük harflerden önce gelir).

Dosyaları ve dizinleri dahil ederek, tek büyük *postgresql.conf* dosyası kullanmak yerine veritabanı yapılandırma bölümlerini mantıksal olarak ayırarak kullanılabilir. Farklı miktarda bellek kullanan iki veritabanı sunucusuna sahip bir şirket düşünün. Her ikisininde paylaşacağı yapılandırma öğeleri elbette vardır(Logging gibi). Ancak sunucuların bellekle ilgili parametreleri değişiklik gösterecektir. Ayrıca sunucu spesifik özelleştirmeler de olabilir. Bu durumu yönetmenin yolu; sunucunuz bazlı yapılandırma değişikliklerini üç dosyaya bölmektir. Bu dosyaları dahil etmek için *postgresql.conf* dosyanısnın sonuna şu şekilde ekleme yapılır:

```bash
include 'shared.conf'
include 'memory.conf'
include 'server.conf'
```

Tüm sistemler aynı `shared.conf` dosyasına sahip olabilir. Belirli miktarda belleğe sahip her sunucu aynı `memory.conf`'u paylaşabilir. Örneğin, 8GB RAM'e sahip sunucular için bir tane, 16GB olanlar için bir tane olabilir. Son olarak `server.conf` içinde ise sunucu spesifik yapılandırma ayarları bulunabilir.

Bir diğer yol, bir yapılandırma dosyası dizini oluşturup dosyaları buraya eklenebilirsiniz. Örneğin, oluşturulan `conf.d` dizini `postgresql.conf`'un sonunda referans verilebilir:

```bash
include_dir 'conf.d'
```

Ardından, `conf.d` dizinindeki dosyaları şu şekilde adlandırabilirniz:

```bash
00shared.conf
01memory.conf
02server.conf
```

Bu adlandırma düzeni, dosyaların yükleneceği açık bir sıra kurar. Sunucu, yapılandırma dosyalarını okurken bir parametre için karşılaşılan son ayarı kullanılacağından bu önemlidir. Verilen örnekte, `conf.d/02server.conf` içerisinde ayarlanan bir parametre, `conf.d/01memory.conf` içinde ayarlanan bir parametreyi geçersiz kılabilir.

Dosyaları daha açıklayıcı bir şekilde adlandırmak için aşağıdaki gibi bir kullanım tercih edilebilir,

```bash
00shared.conf
01memory-8GB.conf
02server-foo.conf
```

Bu tür bir düzenleme, her bir yapılandırma dosyası varyasyonu için benzersiz bir isim sağlar. Böylece, birden çok sunucunun yapılandırma dosyalarının bir sürüm kontrol deposu gibi tek bir noktada depolandığı kullanımlarda, belirsizliği ortadan kaldırmaya yardımcı olur. (Veritabanı yapılandırma dosyalarını sürüm kontrolü altında saklamak, dikkate alınması gereken bir başka iyi uygulamadır.)

{% include links.html %}
