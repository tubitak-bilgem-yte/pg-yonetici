---
title: "Parametreleri Ayarlama"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 23, 2020
sidebar: mydoc_sidebar
permalink: mydoc_paremetreleri_ayarlama.html
folder: mydoc
---

## Parametreleri Ayarlama

### Parametre İsimleri ve Değerleri

Parametre isimleri büyük / küçük harfe duyarlı değildir. Her parametre Boolean, String, Numeric, Floating Point ve Enumerated (enum) olmak üzere beş tip değerden birini alır. Tip, parametrenin ayarlanması için sözdizimini belirler:

- **Boolean:** Bu değerler, tümü büyük / küçük harfe duyarlı olmayan `on`, `off`, `true`, `false`, `yes`, `no`, `1`, `0` veya bunlardan birinin unambiguous prefix'i olarak yazılabilir.

- **String:** Bu tip değerler genelde tek tırnak içinde verilir. Değer içinde tek tırnak işaretleri içeriyorsa bunlar çiftlenmelidir. Sayı ve tanımlayıcı değerlerinde, tırnak işaretleri ihmal edilebilir. Bir SQL anahtar kelimesiyle eşleşen değer bazı durumlarda tırnak işareti gerektirir.

- **Numeric (integer ve floating point):** Numeric parametreler tamsayı (integer) ve ondalıklı (floating-point) sayı formatlarında belirtilebilir. Ondalıklı değerler, parametre integer tipindeyse en yakın tam sayıya yuvarlanır.

- **Birimli Numeric:** Bazı sayısal parametreler bellek ve zaman miktarlarını tanımlayan örtük bir birime sahiptir. Birim bytes, kilobytes, blocks (tipik olarak sekiz kilobayt), milisaniye, saniye veya dakika olabilir. Bu birimlerden herhangi birisiyle belirtilmemiş sayısal değerler, ayarın `pg_settings.unit`'ten öğrenilen varsayılan birimini kullanır. Kolaylık sağlanması için ayarlar açıkça belirtilen bir birimle verilebilir. Örneğin bir zaman değeri '120 ms' şeklinde ifade edilebilir. Bu kullanım, mevcut parametre birimini ne olursa olsun dönüştürecektir. Değerler, string olarak tırnak işaretleriyle verilir. Birim ismi büyük / küçük harfe duyarlıdır, sayısal değer ile birim arasında boşluk olabilir.
  - Geçerli bellek birimleri: B (bayt), kB (kilobayt), MB (megabayt), GB (gigabayt) ve TB (terabayt).
  - Geçerli zaman birimleri: us (mikrosaniye), ms (milisaniye), s (saniye), min (dakika), h (saat) ve d (gün).

{% include callout.html content=" Ondalıklı bir değer birimle belirtildiğinde, varsa bir sonraki küçük birimin katına yuvarlanır. Örneğin 30,1 GB, 32319628902 B'ye değil 30822 MB'ye dönüştürülür. Parametre tamsayı tipindeyse birim dönüşümünden sonra tam sayıya yuvarlama gerçekleşir." type="info" %}

- **Enumerated:** Enumerated tipindeki parametreler string parametreleri ile aynı şekilde yazılır. Bu değerler belirli bir değer kümesiyle sınırlandırılır. Enum değerler büyük / küçük harfe duyarlı değildir.

### Yapılandırma Dosyası Aracılığıyla Parametre Etkileşimi

Parametreleri ayarlamanın en temel yolu `postgresql.conf` dosyasını düzenlemektir. Veritabanı başlatıldığında varsayılan bir kopya, veri dizininde oluşturulur. Bu dosya aşağıdakine benzer:

```bash
# This is a comment
log_connections = yes
log_destination = 'syslog'
search_path = '"$user", public'
shared_buffers = 128MB
```

Her satırda bir parametre belirtilir. Parametre ismi ve değeri arasındaki eşittir işareti isteğe bağlıdır. Parametre değerinde tırnak içindeki boşluklar dışındaki diğer boşluklar önemsizdir. Boş satırlar önemsenmez. Hash işaretleri (#), ilgili satırın yorum satırı olduğunu belirtir. Basit olmayan tanımlayıcılar ve sayısal olmayan parametre değerleri tek tırnak içerisinde verilmelidir. Dosya, aynı parametre için birden fazla giriş içeriyorsa en sonuncusu kabul edilir.

Bu yöntemle ayarlanan parametreler, küme için varsayılan değerler olur. Geçersiz kılınmadıkları sürece aktif oturumlar tarafından görülen ayarlar bunlardır.

Ana sunucu süreci SIGHUP sinyali aldığında yapılandırma dosyasını yeniden okur. SIGHUP sinyali `pg_ctl reload` komutu veya `pg_reload_conf ()` SQL işlevi ile gönderilebilir. Ana sunucu süreci bu sinyali, çalışan tüm sunucu süreçlerine yayar ve mevcut oturumlar da yeni değerlere adepte olur. Bu, o anda yürütülen istemci komutunun tamamlanmasından sonra gerçekleşir. İsteğe göre sinyal doğrudan bir sunucu sürecine gönderilebilir. Bazı parametreler yalnızca sunucu başlangıcında ayarlanabilir. Yapılandırma dosyasında yapılan herhangi bir değişiklik, sunucu yeniden başlatılıncaya kadar yok sayılır. Yapılandırma dosyasındaki geçersiz parametre ayarları benzer şekilde SIGHUP işlemi sırasında önemsenmez ancak günlüğe kaydedilir.

PostgreSQL veri dizininde *postgresql.conf* ile aynı formatta, otomatik düzenlemeler için `postgresql.auto.conf` dosyasını içerir. *postgresql.auto.conf* [ALTER SYSTEM](https://www.postgresql.org/docs/current/sql-altersystem.html) komutuyla verilen ayarları içerir. Bu dosya içindeki ayarlar, *postgresql.conf* içindeki ayarları geçersiz kılar. *postgresql.auto.conf* dosyası harici araçlar tarafından da değiştirilebilir. Bu araçlar ALTER SYSTEM'in yaptığı gibi, yeni ayarlar ekleyebilir veya tekrarlanan ayar ve yorumları kaldırabilir.

Yapılandırma dosyalarındaki değişiklikleri önceden test etmek veya SIGHUP sinyalinden istenen etkiler alınmadığı durumlarda sorunları tespit etmek için [`pg_file_settings`](https://www.postgresql.org/docs/current/view-pg-file-settings.html) sistem view'ı kullanılabilir.

### SQL Aracılığıyla Parametre Etkileşimi

PostgreSQL yapılandırma varsayılanları ayarlamak için üç SQL komutu sağlar. `ALTER SYSTEM` komutu, global varsayılanları değiştirmek için SQL erişilebilir bir yöntem sağlar. İşlevsel olarak *postgresql.conf*'u düzenlemekle eşdeğerdir. Bu komuta ek, her bir veritabanı ve rol için varsayılanların ayarlanabilmesi için iki komut vardır:

- `ALTER DATABASE`, global ayarların veritabanı bazında geçersiz kılınması için kullanılır.
- `ALTER ROLE`, kullanıcı spesifik değerlerle hem global hem de veritabanı bazında ayarların geçersiz kılınmasını sağlar.

{% include note.html content="`ALTER DATABASE` ve `ALTER ROLE` ile yapılan ayarlamalar yeni veritabanı oturumu başlangıcında geçerli olur. Yapılandırma dosyalarından ve komut satırından aldığı değerleri geçersiz kılarak oturumun geri kalanı için varsayılanları oluştururlar. Bazı ayarlar sunucu başlatıldıktan sonra değiştirilemediği için bu ve aşağıda bahseceğimiz komutlardan biriyle ayarlanamaz."%}

İstemci, veritabanına bağlandığında oturum bazlı ayarlarla etkileşim kurması için PostgreSQL tarafından iki ek SQL komutu sağlanır:

- `SHOW`, parametrelerin mevcut değerini gösterir. Bu komuta karşılık gelen işlev `current_setting (setting_name text)`dir.
- `SET`, bu parametrelerin oturum düzeyinde mevcut değeri değiştirmek için kullanılır. Yapılan değişikliklerin diğer oturumlar üzerinde etkisi yoktur. Bu komuta karşılık gelen fonksiyon `set_config(setting_name, new_value, is_local)`dir.

Oturum yerel değerlerini görüntüleme, değiştirme işlemlerinde `pg_settings` sistem view'ı da kullanılabilir:

- Bu view'ı sorgulamak, `SHOW ALL`'dan daha fazla ayrıntı sağlar. `pg_settings` kullanımı daha esnektir.

- Bu view üzerinde `UPDATE` ile `setting` sütununu güncellemek `SET` komutunu uygulamakla eşdeğerdir. Örneğin,

```bash
SET configuration_parameter TO DEFAULT;
```

ifadesinin eş değeri,

```bash
UPDATE pg_settings SET setting = reset_val WHERE name = 'configuration_parameter';
```

### Kabuk Aracılığıyla Parametre Etkileşimi

PostgreSQL ayarları, kabuk kullanılarak verilebilir.

- Sunucu başlatılırken, parametre ayarları `-c` komut satırı parametresi ile `postgres` komutuna verilir. Bu yolla yapılan ayarlar, *postgresql.conf* ve `ALTER SYSTEM` ile yapılan ayarlamarı geçersiz kılar. Örnek kullanım,

```bash
postgres -c log_connections=yes -c log_destination='syslog'
```

İstemci oturumu başlatırken, parametre ayarları `PGOPTIONS` ortam değişkeniyle belirtilebilir. Bu şekilde kurulan ayarlar ilgili oturum boyunca varsayılanı teşkil eder, diğer oturumları etkilemez. `PGOPTIONS`, `postgres` komutunu başlatırken kullanılan formatta `-c` bayrağı belirtilmelidir. Örnek kullanım,

```bash
env PGOPTIONS="-c geqo=off -c statement_timeout=5min" psql
```

### Yapılandırma Dosyası İçeriklerini Yönetme

PostgreSQL, kompleks *postgresql.conf* dosyalarını alt dosyalara bölmek için çeşitli özellikler sağlar. Bu özellikler, birbiriyle ilişkili farklı yapılandırma ayarlarına sahip sunucuları yönetirken kullanışlıdır.

*postgresql.conf* dosyası içerdiği parametre ayarlarına ek başka bir dosya belirten `include` direktifleri içerebilir. *include* yoluyla verilen dosya, yapılandırma dosyasına eklenmiş gibi okunur ve işlenir. Bu, bir yapılandırma dosyasının fiziksel olarak ayrı parçalara bölünmesine olanak tanır. Basitçe direktif dahil etme şu şekildedir:

```bash
include 'filename'
```

{% include callout.html content=" Dosya yolu mutlak bir yol değilse yapılandırma dosyasının bulunduğu dizine göre referans alınır." type="info" %}

Başvurulan dosyanın mevcut olmadığı veya okunamadığı durumlar için *include direktifi* ile aynı şekilde davranan `include_if_exists` ile direktif de verilebilir. *include* bunu bir hata koşulu olarak kabul eder, `include_if_exists` günlüğe bir mesajı yazarak verilen yapılandırma dosyasını işlemeye devam eder.

*postgresql.conf* dosyasında, dahil edilecek tüm yapılandırma dosyalarının bulunduğu bir dizini belirten `include_dir` direktifi verilebilir. Örnek:

```bash
include_dir 'directory'
```

Mutlak olmayan dizin isimleri, referans yapılandırma dosyalarını içeren dizine göre ilişkilendirilir. Belirtilen dizin içindeki `.conf` son ekiyle biten dosyalar dahil edilecektir. `.` ile başlayan dosyalar bazı platformlarda gizlendiği için bu tür dosyalar hataları önlemek adına görmezden gelinir. *include_dir* içindeki dosyalar, dosya ismi sırasına göre işlenir.

Dosyaları ve dizinleri dahil edip tek büyük *postgresql.conf* dosyası kullanmak yerine veritabanı yapılandırma bölümlerini mantıksal olarak ayırarak kullanılabilir. Farklı miktarda bellek kullanan iki veritabanı sunucusuna sahip bir şirket düşünün. Her ikisininde paylaşacağı yapılandırma öğeleri elbette vardır (logging gibi). Ancak sunucuların bellekle ilgili parametreleri değişiklik gösterecektir. Sunucu spesifik özelleştirmeler de olabilir. Bu durumu yönetmenin yolu; sunucu bazlı yapılandırma değişikliklerini üç dosyaya bölmektir. Bu dosyaları dahil etmek için *postgresql.conf* sonuna şu şekilde ekleme yapılır:

```bash
include 'shared.conf'
include 'memory.conf'
include 'server.conf'
```

Tüm sistemler aynı `shared.conf` dosyasına sahip olabilir. Belirli miktarda belleğe sahip her sunucu aynı `memory.conf`'u paylaşabilir. Örneğin, 8GB RAM'e sahip sunucular için bir tane, 16GB olanlar için bir tane olabilir. Son olarak `server.conf` içinde ise sunucu spesifik yapılandırma ayarları bulunabilir.

Bir diğer yol, bir yapılandırma dosyası dizini oluşturup dosyaları buraya eklemektir. Örneğin, oluşturulan `conf.d` dizini `postgresql.conf`'un sonunda referans verilebilir:

```bash
include_dir 'conf.d'
```

Ardından, `conf.d` dizinindeki dosyaları şu şekilde adlandırılabilir:

```bash
00shared.conf
01memory.conf
02server.conf
```

Bu adlandırma düzeni, dosyaların yükleneceği sırayı belirtir. Sunucu, yapılandırma dosyalarını okurken bir parametre için karşılaşılan son ayarı kullanacağından bu önemlidir. Verilen örnekte, `conf.d/02server.conf` içerisinde ayarlanan bir parametre, `conf.d/01memory.conf` içinde ayarlanan aynı parametreyi geçersiz kılar.

Dosyaları daha açıklayıcı bir şekilde adlandırmak için aşağıdaki gibi bir kullanım tercih edilebilir:

```bash
00shared.conf
01memory-8GB.conf
02server-foo.conf
```

Bu şekilde yapılmış bir düzen, yapılandırma dosyası varyasyonları için benzersiz bir isim sağlar. Bu, birden çok sunucunun yapılandırma dosyalarının bir sürüm kontrol deposu gibi tek bir noktada depolandığı kullanımlarda, belirsizliği ortadan kaldırmaya yardımcı olur.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/config-setting.html)

{% include links.html %}
