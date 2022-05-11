---
title: "Otomatik Vacuuming"
layout: default
parent: Veritabanı Yapılandırması
nav_order: 10
---

## Otomatik Vacuuming

Bu başlıkta verilen parametre ayarları otomatik vakum özelliğinin davranışını kontrol eder. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#AUTOVACUUM). Bu ayarların çoğu tablo bazında geçersiz kılınabilir bkz. [Depolama Parametreleri](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS).

### `autovacuum`

{% include parameter_info.html parametre="autovacuum" %}

{% include callout.html content=" **Sunucunun autovacuum başlatıcı arka plan programını (autovacuum launcher) çalıştırıp çalıştırmayacağını kontrol eder.** Varsayılan olarak açıktır; ancak, autovacuum'un çalışması için [track_counts](mydoc_calisma_zamani_istatistikleri.html) paremetresinin de açık olmalısı gereklidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Otomatik vakumlama, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir.<br/><br/>

Bu parametre devre dışı bırakıldığında bile sistemin transaction ID wraparound'ı önlemek için gerektiğinde otomatik vakum süreçlerini başlatacağını unutmayın. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-WRAPAROUND)" type="primary" %}

### `log_autovacuum_min_duration`

{% include parameter_info.html parametre="log_autovacuum_min_duration" %}

{% include callout.html content=" **Autovacuum tarafından yürütülen her eylemin, en az bu parametrede belirtilen süre boyunca çalıştırılması durumunda loglar.** Bu parametreyi sıfıra ayarlamak, tüm otomatik vakum işlemlerini günlüğe kaydeder. `-1` (varsayılan) otomatik vakum eylemlerinin günlüğe kaydedilmesini devre dışı bırakır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Örneğin, 250 ms ayarı, 250 ms veya daha uzun süre çalışan tüm otomatik vakumlar ve analizler günlüğe kaydedilir. Bu parametrenin `-1` dışındaki ayar değerlerinde, çakışan bir lock veya eşzamanlı olarak drop edilen bir ilişki sebepli otomatik vakum eylemi atlanırsa günlüğe bir mesaj yazılır. Bu parametrenin etkinleştirilmesi, otomatik vakum aktivitesinin izlenmesinde yardımcı olur. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_max_workers`

{% include parameter_info.html parametre="autovacuum_max_workers" %}

{% include callout.html content=" **Herhangi bir anda çalışabilen maksimum otomatik vakum süreci sayısını (autovacuum launcher dışında) belirtir.** Öntanımlı değeri 3'tür. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

### `autovacuum_naptime`

{% include parameter_info.html parametre="autovacuum_naptime" %}

{% include callout.html content=" **Herhangi bir veri tabanında otomatik vakum çalışmaları arasındaki minimum gecikmeyi belirtir.** Her periyotta arka plan programı veritabanını inceler ve bu veritabanındaki tablolar için ihtiyacına göre `VACUUM` ve `ANALYZE` komutlarını verir. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı değeri 1 dakikadır (1 min). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### `autovacuum_vacuum_threshold`

{% include parameter_info.html parametre="autovacuum_vacuum_threshold" %}

{% include callout.html content=" **Herhangi bir tabloda VACUUM tetiklemek için gereken minimum değiştirilmiş ve silinmiş tuple sayısını belirtir.** Varsayılan 50 tuple'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_vacuum_insert_threshold`

{% include parameter_info.html parametre="autovacuum_vacuum_insert_threshold" %}

{% include callout.html content=" **Herhangi bir tabloda VACUUM tetiklemek için gerekli insert edilen tuple sayısını belirtir.** Varsayılan 1000 tuple'dır. `-1` ayarı, insert sayısına bağlı olarak otomatik vakum, herhangi bir tablodaki VACUUM işlemini tetiklemeyecektir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_analyze_threshold`

{% include parameter_info.html parametre="autovacuum_analyze_threshold" %}

{% include callout.html content=" **Herhangi bir tabloda bir ANALYZE tetiklemek için gereken minimum eklenen, değiştirilen veya silinen tuple sayısını belirtir.** Varsayılan 50 tuple'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_vacuum_scale_factor`

{% include parameter_info.html parametre="autovacuum_vacuum_scale_factor" %}

{% include callout.html content=" **Bir VACUUM tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_vacuum_threshold`'a eklenecek tablo boyutu kısmını belirtir.** Varsayılan 0,2'dir (tablo boyutunun %20'si). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_vacuum_insert_scale_factor`

{% include parameter_info.html parametre="autovacuum_vacuum_insert_scale_factor" %}

{% include callout.html content=" **Bir VACUUM tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_vacuum_insert_threshold`'a eklenecek tablo boyutu kısmını belirtir.** Varsayılan 0,2'dir (tablo boyutunun %20'si). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_analyze_scale_factor`

{% include parameter_info.html parametre="autovacuum_analyze_scale_factor" %}

{% include callout.html content=" **Bir ANALYZE tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_analyze_threshold`'a eklenecek tablo boyutu kısmını belirtir.** Varsayılan değer 0,1'dir (tablo boyutunun %10'u). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo saklama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_vacuum_cost_delay`

{% include parameter_info.html parametre="autovacuum_vacuum_cost_delay" %}

{% include callout.html content=" **Otomatik VACUUM işlemlerinde kullanılacak maliyet gecikme değerini belirtir.** `-1` ayarında normal `vacum_cost_delay` değeri kullanılır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan değer 2 milisaniyedir. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

### `autovacuum_vacuum_cost_limit`

{% include parameter_info.html parametre="autovacuum_vacuum_cost_limit" %}

{% include callout.html content=" **Otomatik VACUUM işlemlerinde kullanılacak maliyet sınırı değerini belirtir.** `-1` ayarı (varsayılan değerdir) [vacuum_cost_limit](mydoc_kaynak_tuketimi.html#maliyete-dayal%C4%B1-vacuum-gecikmesi) değerini kullanır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
