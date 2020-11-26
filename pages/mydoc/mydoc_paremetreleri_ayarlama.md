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

Parametre isimleri büyük / küçük harfe duyarlıdır. Her parametre Boolean, String, Numeric, Floating Point ve Enumerated (enum) olmak üzere beş türden birinin değerini alır. Tip, parametrenin ayarlanması için sözdizimini belirler:

- **Boolean:** Değerler `on`, `off`, `true`, `false`, `yes`, `no`, `1`, `0` (tümü büyük / küçük harfe duyarlı değil) veya bunlardan birinin belirsiz olmayan herhangi bir öneki ( unambiguous prefix ) olarak yazılabilir.

- **String ( dize ):** Değeri tek tırnak içine alın ve değer içindeki tek tırnak işaretlerini çiftleyin. Eğer değer basit sayı veya tanımlayıcıysa, alıntılar genellikle ihmal edilebilir. (Bir SQL anahtar kelimesiyle eşleşen değerler bazı bağlamlarda alıntı yapılmasını gerektirir.)

- **Numeric (integer ve floating point):** Sayısal parametreler, tamsayı ve kayan nokta ( floating-point ) formatlarında belirtilebilir. Kesirli değerler, parametre tamsayı türündeyse en yakın tam sayıya yuvarlanır. Tamsayı parametreleri ayrıca onaltılık ( hexadecimal ) girişi (0x ile başlayan) ve sekizlik ( octal ) girişi (0 ile başlayan) kabul eder, ancak bu formatlar kesirli olamaz. Binlik ayırıcı kullanmayın. Onaltılık giriş dışında tırnak işaretleri gerekli değildir.

- **Numeric with Unit:** Bazı sayısal parametrelerin örtük bir birimi vardır çünkü bunlar bellek veya zaman miktarlarını tanımlarlar. Birim bytes, kilobytes, blocks (tipik olarak sekiz kilobayt), milisaniye, saniye veya dakika olabilir. Bu ayarlardan birisiyle süslenmemiş sayısal değerler ayarın varsayılan birimini kullanır. Bu `pg_settings.unit`'ten öğrenilebilir. Kolaylık sağlanması içim ayarlar açıkça belirtilen bir birimle verilebilir. Örneğin bir zaman değeri için '120 ms' gibi. Bunlar parametrenin gerçek birimi ne olursa olsun dönüştürülecektir. Bu özelliği kullanmak için değerin bir string ( dize ) olarak (tırnak işaretleriyle) yazılması gerekir. Birim ismi büyük / küçük harfe duyarlıdır ve sayısal değer ile birim arasında boşluk olabilir.
  - Geçerli bellek birimleri B (bayt), kB (kilobayt), MB (megabayt), GB (gigabayt) ve TB'dir (terabayt). Bellek birimleri için katlar 1000 değil 1024'tür.
  - Geçerli zaman birimleri `us` (mikrosaniye), `ms` (milisaniye), `s` (saniye), `min` (dakika), `h` (saat) ve `d` (gün).

{% include callout.html content="Bir birimle virgüllü bir değer belirtildiği durumda, varsa sonraki daha küçük birimin katına yuvarlanır. Örneğin 30,1 GB, 32319628902 B'ye değil 30822 MB'ye dönüştürülür. Parametre integer tipindeyse birim dönüştürmeden sonra tam sayıya yuvarlama yapılır." type="info" %}

- **Enumerated:** Enumerated tipindeki parametreler string parametreleriyle aynı şekilde yazılır. Belirli bir değer kümesine sahip olurlar.Enum parametre değerleri büyük / küçük harfe duyarlıdır.

### Yapılandırma Dosyası Üzerinden Parametre Etkileşimi

Bu parametreleri ayarlanması veri dizininde tutulan `postgresql.conf` dosyasında yapılır. Veritabanı küme dizini başlatıldığında varsayılan bir kopya kurulur. Bu dosyanın neye benzeyebileceğine dair örnek:

```bash
# This is a comment
log_connections = yes
log_destination = 'syslog'
search_path = '"$user", public'
shared_buffers = 128MB
```

Her satıra bir parametre belirtilir. İsim ve değer arasındaki eşittir işareti isteğe bağlıdır. Tırnak içine alınmış bir parametre değeri dışında boşluklar önemsizdir ve boş satırlar göz ardı edilir. Karma işaretler (#), ilgili satırı yorum olarak belirler. Basit olmayan tanımlayıcılar ve sayısal olmayan parametre değerleri tek tırnaklı olmalıdır. Bir parametre değerinde alıntı yerleştirmek için, iki tırnak işareti (tercih edilen) veya ters eğik çizgi kullanın. Dosya aynı parametre için birden fazla girdi içeriyorsa, sonuncusu hariç tümü görmezden gelinir.

Bu şekilde ayarlanan parametreler küme için varsayılan değerler olur. Aktif oturumlar tarafından görülen ayarlar, geçersiz kılınmadıkları sürece bu değerler olacaktır. Aşağıdaki bölümlerde yönetici veya kullanıcının bu varsayılanları geçersiz kılabileceği yolları açıklamaktadır.

Ana sunucu süreci ( process ) SIGHUP sinyali aldığında yapılandırma dosyası yeniden okunur. Bu sinyal `pg_ctl reload` komutunu kullanarak veya `pg_reload_conf ()` SQL işlevini çağırarak gönderilebilir. Ana sunucu süreci bu sinyali çalışan tüm sunucu süreçlerine gönderir ve mevcut oturumlar da yeni değerlere adepte olur (bu, o anda yürütülen istemci komutunun tamamlanmasından sonra gerçekleşir). Alternatif olarak, sinyali doğrudan bir sunucu sürecine gönderebilirsiniz. Bazı parametreler yalnızca sunucu başlangıcında ayarlanabilir. Yapılandırma dosyasındaki girdilerinde yapılan herhangi bir değişiklik, sunucu yeniden başlatılıncaya kadar yok sayılacaktır. Yapılandırma dosyasındaki geçersiz parametre ayarları benzer şekilde SIGHUP işlemi sırasında göz ardı edilir ancak günlüğe kaydedilir.

PostgreSQL veri dizini postgresql.conf'a ek olarak postgresql.conf ile aynı formatta, otomatik olarak düzenlenmesi amaçlanan `postgresql.auto.conf` dosyasını içerir. Bu dosya, [ALTER SYSTEM](https://www.postgresql.org/docs/current/sql-altersystem.html) komutuyla verilen ayarları içerir. postgresql.auto.conf içindeki ayarlar postgresql.conf içindekileri geçersiz kılar.

Harici araçlar da `postgresql.auto.conf` dosyasını değiştirebilir. Eşzamanlı yapılan `ALTER SYSTEM` komutu bu değişiklikler üzerine yazabileceğinden sunucu çalışırken bunu yapmanız önerilmez. Bu araçlar, yeni ayarları sona ekleyebilir veya yinelenen ayarları ve / veya yorumları kaldırabilir (ALTER SYSTEM'in yapacağı gibi).

`pg_file_settings` sistem görünümü ( view ), yapılandırma dosyalarındaki değişiklikler önceden test edilmek istendiğin veya SIGHUP sinyalinden istenen etkiler alınmadığında sorunları tespit etmek için kullanılabilir.

### SQL Üzerinden Parametre Etkileşimi

PostgreSQL'de yapılandırma varsayılanları oluşturmak için üç SQL komutu kullanılır. Daha önce bahsedilen ALTER SYSTEM komutu, global varsayılanları değiştirmek için SQL ile erişilebilir bir yol sağlar. İşlevsel olarak postgresql.conf'u düzenlemeye eşdeğerdir. Ek olarak, varsayılanların herbir veritabanı ve rol için ayarlanabilmesini sağlayan iki komut vardır:

- `ALTER DATABASE` komutu global ayarların veritabanı bazında geçersiz kılınması için kullanılır.
- `ALTER ROLE` komutu hem global hem de veritabanı bazında ayarların kullanıcıya özel değerlerle geçersiz kılınması için kullanılır.

{% include callout.html content="ALTER DATABASE ve ALTER ROLE ile ayarlanan değerler yalnızca yeni veritabanı oturumu başlatılırken geçerli olur. Yapılandırma dosyalarından ve komut satırından elde edilen değerleri geçersiz kılarak oturumun geri kalanı için varsayılanları oluştururlar." type="info" %}

{% include note.html content=" Bazı ayarlar sunucu başlatıldıktan sonra değiştirilemediği için bu ve aşağıda bahsecedeğimiz komutlarla ayarlanamaz."%}

PostgreSQL, istemci veritabanına bağlandığında oturum bazlı ayarlarıyla etkileşim kurması için iki ek SQL komutu sağlar:

- `SHOW` komutu tüm parametrelerin mevcut değerinin incelenmesinde kullanılır. Karşılık gelen işlev `current_setting (setting_name text)`dir.
- `SET` komutu oturum düzeyinde ayarlanabilen parametrelerin mevcut değerinin değiştirilmesinde kullanılır, diğer oturumlar üzerinde etkisi yoktur. Karşılık gelen işlev `set_config(setting_name, new_value, is_local)`dir.

Ayrıca `pg_settings` sistem görünümü ( view ) de oturum yerel değerlerini görüntülemek ve değiştirmek için kullanılabilir:

- Bu görünümü sorgulamak `SHOW ALL`'e benzemekle birlikte daha fazla ayrıntı sağlar. Ayrıca, filtre belirlemek ve diğer ilişkilerle join'lemek mümkün olduğundan daha esnektir.

- Bu görünümde `UPDATE` ile `setting` sütununu güncellemek `SET` komutunu uygulamakla eşdeğerdir. Örneğin,

```bash
SET configuration_parameter TO DEFAULT;
```

ifadesinin eş değeri,

```bash
UPDATE pg_settings SET setting = reset_val WHERE name = 'configuration_parameter';
```

### Kabuk Üzerinden Parametre Etkileşimi

Veritabanı veya rol düzeyinde global varsayılanlar ayarlama işlemlerine ek olarak, ayarları kabuk imkanları ile PostgreSQL'e aktarabilirsiniz.

- Sunucunun başlatılırken parametre ayarları `-c` parametresi ile `postgres` komutuna gönderilebilir. Bu şekilde yapılan ayarlar, `postgresql.conf` ve `ALTER SYSTEM` ile yapılan ayarlamarı geçersiz kılar, böylece sunucuyu yeniden başlatmadan global olarak değiştirilemezler. Örnek kullanım,

```bash
postgres -c log_connections=yes -c log_destination='syslog'
```

Bir istemci libpq aracılığıyla oturumu başlatırken, parametre ayarları `PGOPTIONS` ortam değişkeniyle belirtilir. Bu şekilde oluşturulan ayarlar ilgili oturumun ömrü için varsayılanları oluştururken diğer oturumları etkilemez. Tarihsel nedenlerden dolayı `PGOPTIONS` formatı `postgres` komutunu başlatırken kullanılan formata benzer şekilde olmakla birlikte `-c` bayrağı belirtilmelidir. Örnek kullanım,

```bash
env PGOPTIONS="-c geqo=off -c statement_timeout=5min" psql
```

### Yapılandırma Dosyası İçeriklerini Yönetme

PostgreSQL, kompleks `postgresql.conf` dosyalarını alt dosyalara ayırmak için çeşitli özellikler sağlar. Bu özellikler, özellikle birbiriyle ilişkili ancak aynı olmayan yapılandırmalara sahip birden çok sunucuyu yönetirken kullanışlıdır.

`postgresql.conf` dosyası özel parametre ayarlarına ek olarak başka bir dosya belirten direktifleri içerebilir. Bu şekilde tanımlamış ifadeler yapılandırma dosyasına eklenmiş gibi okunacak ve işlenecektir. Bu özellik, bir yapılandırma dosyasının fiziksel olarak ayrı parçalara bölünmesine olanak tanır. Direktif dahil etme basitçe şu şekilde uygulanır:

```bash
include 'filename'
```

{% include callout.html content=" Dosya adı mutlak bir yol değilse, referans yapılandırma dosyasını içeren dizine göre alınır." type="info" %}

Ayrıca, başvurulan dosyanın mevcut olmadığı veya okunamadığı durumlar için `include` direktifi ile aynı şekilde davranan bir de `include_if_exists` direktif de vardır. `include` bunu bir hata koşulu olarak kabul ederken `include_if_exists` günlüğe bir mesaj kaydederek referans yapılandırma dosyasını işlemeye devam eder.

`postgresql.conf` dosyası dahil edilecek yapılandırma dosyalarının tüm bir dizinini belirten `include_dir` yönergelerini de içerebilir. Şu şekilde,

```bash
include_dir 'directory'
```

Mutlak olmayan dizin isimleri, referans yapılandırma dosyasını içeren dizine göre alınır. Belirtilen dizin içinde, dizin olmayan `.conf` sonekiyle biten dosyalar dahil edilecektir. Ayrıca, `.` ile başlayan dosyalar bazı platformlarda gizlendiği için bu tür dosyalar hataları önlemek adına önemsenmez. Dahil etme dizini içindeki birden çok dosya, dosya ismi sırasına göre işlenir (C yerel ayar (locale) kurallarına göre, yani harflerden önce sayılar ve küçük harflerden önce büyük harfler).

Dosyaları ve dizinleri dahil etme özelliği, tek büyük postgresql.conf dosyasına sahip olmak yerine veritabanı yapılandırmasının bölümlerini mantıksal olarak ayırmak için kullanılabilir. Her biri farklı miktarda belleğe sahip iki veritabanı sunucusuna sahip bir şirket düşünün. Günlüğe kaydetme gibi her ikisininde paylaşacağı yapılandırma öğeleri elbette vardır. Ancak sunucuların bellekle ilgili parametreleri değişiklik gösterecektir. Ayrıca sunucuya özel özelleştirmeler de olabilir. Bu durumu yönetmenin bir yolu, siteniz için özel yapılandırma değişikliklerini üç dosyaya bölmektir. Bunları dahil etmek için `postgresql.conf` dosyanızın sonuna şu şekilde ekleyebilirsiniz:

```bash
include 'shared.conf'
include 'memory.conf'
include 'server.conf'
```

Tüm sistemler aynı `shared.conf` dosyasına sahip olabilir. Belirli miktarda belleğe sahip her sunucu aynı `memory.conf`'u paylaşabilir. Örneğin, 8GB RAM'e sahip sunucular için bir tane, 16GB olanlar için bir tane olabilir. Ve son olarak `server.conf` içinde sunucuya özgü yapılandırma bilgilerine sahip olabilir.

Diğer bir olanak, bir yapılandırma dosyası dizini oluşturarak dosyaları oraya eklenebilirsiniz. Örneğin, bir `conf.d` dizinine `postgresql.conf`'un sonunda referans verilebilir:

```bash
include_dir 'conf.d'
```

Ardından, `conf.d` dizinindeki dosyaları şu şekilde adlandırabilirniz:

```bash
00shared.conf
01memory.conf
02server.conf
```

Bu adlandırma kuralı, dosyaların yükleneceği açık bir sıra belirler. Sunucu yapılandırma dosyalarını okurken bir parametre için karşılaşılan son ayarı kullanılacağı için bu önemlidir. Verilen örnekte, `conf.d/02server.conf` içerisinde ayarlanan bir parametre, `conf.d/01memory.conf` içinde ayarlanan bir parametreyi geçersiz kılabilir.

Bunun yaklaşım yerine dosyaları daha açıklayıcı bir şekilde adlandırmak için aşağıdaki gibi bir kullanım tercih edebilirsiniz,

```bash
00shared.conf
01memory-8GB.conf
02server-foo.conf
```

Bu tür bir düzen, her bir yapılandırma dosyası varyasyonu için benzersiz bir isim sağlar. Bu kullanım, birden çok sunucunun yapılandırma dosyalarının bir sürüm kontrol deposu gibi tek bir noktada depolandığı kullanımlarda belirsizliği ortadan kaldırmaya yardımcı olabilir. (Veritabanı yapılandırma dosyalarını sürüm kontrolü altında saklamak, dikkate alınması gereken başka bir iyi uygulamadır.)

{% include links.html %}
