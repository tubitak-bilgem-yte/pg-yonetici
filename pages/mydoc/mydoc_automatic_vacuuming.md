---
title: "Otomatik Vacuuming"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 17, 2020
summary: "Automatic Vacuuming"
sidebar: mydoc_sidebar
permalink: mydoc_automatic_vacuuming.html
folder: mydoc
---

## Otomatik Vacuuming

Bu başlıkta verilen parametre ayarları otomatik vakum özelliğinin davranışını kontrol eder. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#AUTOVACUUM). Bu ayarların çoğu tablo bazında geçersiz kılınabilir bkz. [Depolama Parametreleri](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS).

{% include callout.html content="**`autovacuum (boolean)`**: Sunucunun autovacuum başlatıcı arka plan programını (autovacuum launcher) çalıştırıp çalıştırmayacağını kontrol eder. Bu, varsayılan olarak açıktır; ancak, autovacuum'un çalışması için [track_counts](mydoc_calisma_zamani_istatistikleri.html) paremetresinin de açık olmalısı gereklidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Otomatik vakumlama, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir.<br/><br/>

Bu parametre devre dışı bırakıldığında bile sistemin transaction ID wraparound'ı önlemek için gerektiğinde otomatik vakum süreçlerini başlatacağını unutmayın. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-WRAPAROUND)." type="primary" %}

{% include callout.html content="**`log_autovacuum_min_duration (integer)`**: Autovacuum tarafından yürütülen her eylemin, en az bu parametrede belirtilen süre boyunca çalıştırılması durumunda günlüğe kaydedilmesini sağlar. Bu parametreyi sıfıra ayarlamak, tüm otomatik vakum işlemlerini günlüğe kaydeder. `-1` (varsayılan) otomatik vakum eylemlerinin günlüğe kaydetdilmesini devre dışı bırakır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Örneğin, 250 ms ayarı, 250 ms veya daha uzun süre çalışan tüm otomatik vakumlar ve analizler günlüğe kaydedilir. Bu parametrenin `-1` dışındaki ayar değerlerinde, çakışan bir lock veya eşzamanlı olarak drop edilen bir ilişki sebepli otomatik vakum eylemi atlanırsa günlüğe bir mesaj yazılır. Bu parametrenin etkinleştirilmesi, otomatik vakum aktivitesinin izlenmesinde yardımcı olabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_max_workers (integer)`**: Herhangi bir anda çalışabilen maksimum otomatik vakum süreci sayısını (autovacuum launcher dışında) belirtir. Öntanımlı değeri 3'tür. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="**`autovacuum_naptime (integer)`**: Herhangi bir veri tabanında otomatik vakum çalışmaları arasındaki minimum gecikmeyi belirtir. Her periyotta arka plan programı veritabanını inceler ve bu veritabanındaki tablolar için ihtiyacına göre `VACUUM` ve `ANALYZE` komutlarını verir. Bu değer birimsiz belirtilirse saniye olarak alınır. Öntanımlı değeri bir dakikadır (1 min). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_threshold (integer)`**: Herhangi bir tabloda bir VACUUM tetiklemek için gereken minimum değiştirilmiş ve silinmiş tuple sayısını belirtir. Varsayılan 50 tuple'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_insert_threshold (integer)`**: Herhangi bir tabloda VACUUM tetiklemek için gerekli insert edilen tuple sayısını belirtir. Varsayılan 1000 tuple'dır. `-1` ayarı, insert sayısına bağlı olarak otomatik vakum herhangi bir tablodaki bir VAKUM işlemini tetiklemeyecektir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_analyze_threshold (integer)`**: Herhangi bir tabloda bir ANALYZE tetiklemek için gereken minimum eklenen, değiştirilen veya silinen tuple sayısını belirtir. Varsayılan 50 tuple'dir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_scale_factor (floating point)`**: Bir VACUUM tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_vacuum_threshold`'a eklenecek tablo boyutu kısmını belirtir. Varsayılan 0,2'dir (tablo boyutunun %20'si). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_insert_scale_factor (floating point)`**: Bir VACUUM tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_vacuum_insert_threshold`'a eklenecek tablo boyutu kısmını belirtir. Varsayılan 0,2'dir (tablo boyutunun %20'si). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_analyze_scale_factor (floating point)`**: Bir ANALYZE tetiklenip tetiklenmeyeceğine karar verilirken `autovacuum_analyze_threshold`'a eklenecek tablo boyutu kısmını belirtir. Varsayılan değer 0,1'dir (tablo boyutunun %10'u). Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo saklama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_freeze_max_age (integer)`**: Bir tablonun `pg_class.relfrozenxid` alanının, tablo içinde transaction ID wraparound'ı önlemek için VACUUM işlemi zorlanmadan önce ulaşacağı maksimum yaşı (transaction'da) belirtir. Otomatik vakum başka şekilde devre dışı bırakıldığında bile sistemin wraparound'ı önlemek için otomatik vakum işlemlerini başlatacağını unutmayın.<br/><br/>

Vakum, ayrıca eski dosyaların `pg_xact` alt dizininden kaldırılmasına da izin verdiğinden varsayılan değer nispeten düşük 200 milyon transaction'dır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Bu ayar tablo depolama parametreleri değiştirilerek belirli tablolar için azaltılabilir. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-WRAPAROUND)." type="primary" %}

{% include callout.html content="**`autovacuum_multixact_freeze_max_age (integer)`**: Bir VACUUM işlemi, tablo içinde multixact ID wraparound'ı önlemek için zorlanmadan önce bir tablonun `pg_class.relminmxid` alanının elde edebileceği maksimum yaşı (multixacts'de) belirtir. Otomatik vakum başka şekilde devre dışı bırakıldığında bile sistemin wraparound'ı önlemek için otomatik vakum işlemlerini başlatacağını unutmayın.<br/><br/>

multixacts vakumlamak aynı zamanda eski dosyaların `pg_multixact/members` ve `pg_multixact/offsets` alt dizinlerinden kaldırılmasına da izin verdiğinden varsayılan değer nispeten düşük 400 milyon multixact'tır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Bu ayar tablo depolama parametreleri değiştirilerek belirli tablolar için azaltılabilir. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-MULTIXACT-WRAPAROUND)" type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_cost_delay (floating point)`**: Otomatik VAKUM işlemlerinde kullanılacak maliyet gecikme değerini belirtir. `-1` ayarında normal `vacum_cost_delay` değeri kullanılır. Bu değer birimsiz belirtilirse milisaniye olarak alınır. Varsayılan değer 2 milisaniyedir. Bu parametre yalnızca *postgresql.conf* dosyasında ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include callout.html content="**`autovacuum_vacuum_cost_limit (integer)`**: Otomatik VAKUM işlemlerinde kullanılacak maliyet sınırı değerini belirtir. `-1` ayarı (varsayılan değerdir), normal [vacuum_cost_limit](mydoc_kaynak_tuketimi.html#maliyete-dayal%C4%B1-vacuum-gecikmesi) değeri kullanılır. Birden fazla varsa, değerin çalışan otomatik vakum worker'ları arasında orantılı olarak dağıtıldığına ve böylece her bir worker için limitlerin toplamının bu değişkenin değerini aşmayacağına dikkat edin. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Bu ayar, tablo depolama parametreleri değiştirilerek belirli tablolar için geçersiz kılınabilir." type="primary" %}

{% include links.html %}
