---
title: "Çalışma Zamanı İstatistikleri"
tags: [PostgreSQL]
keywords: postgres
last_updated: January 11, 2021
sidebar: mydoc_sidebar
permalink: mydoc_calisma_zamani_istatistikleri.html
folder: mydoc
---

## Çalışma Zamanı İstatistikleri

### Sorgu ve İndeks İstatistikleri Toplayıcı

Bu başlık altında açıklanan parametreler, sunucuda istatistik toplama özelliklerini kontrol eder. İstatistik toplama etkinleştirildiğinde, üretilen verilere `pg_stat` ve `pg_statio` sistem görünümleri ile erişilebilir. Daha fazla bilgi için bkz. [](https://www.postgresql.org/docs/current/monitoring.html).

{% include callout.html content="**`track_activities (boolean)`**: Herbir oturumda komutların yürütülmesi hakkında bilgi toplar. Bu parametre varsayılan olarak açıktır. Sağladığı bilgiler tüm kullanıcılar tarafından görülemez. Yalnızca süper kullanıcılar ve rapor edilen oturumun sahibi olan kullanıcı tarafından görülebilir. Bu nedenle bir güvenlik riski oluşturmaz. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`track_activity_query_size (integer)`**: `pg_stat_activity.query` alanı için ayrılan boyutu bayt cinsinden ayarlar. Bu değer birimsiz belirtilirse bayt olarak alınır. Öntanımlı değeri 1024 bayttır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="**`track_counts (boolean)`**: Veritabanı etkinliği ile ilgili istatistiklerin toplanmasını sağlar. Bu parametre varsayılan olarak açıktır. Autovacuum daemon toplanan bu bilgilere ihtiyaç duyar. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`track_io_timing (boolean)`**: Veritabanı I / O etkinliğinin zamanlama istatistiklerini toplar. Bu parametre varsayılan olarak kapalıdır, çünkü işletim sistemini geçerli saat için tekrar tekrar sorgulayacağından bazı platformlarda önemli ek yüklere neden olabilir. Sisteminizdeki zamanlamanın ek yükünü ölçmek için [`pg_test_timing`](https://www.postgresql.org/docs/current/pgtesttiming.html) aracını kullanabilirsiniz. I / O zamanlama bilgisi [`pg_stat_database`](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-VIEW) içinde, `BUFFERS` opsiyonu kullanıldığında [EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html) çıktısında ve [`pg_stat_statements`](https://www.postgresql.org/docs/current/pgstatstatements.html) tarafından görüntülenir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`track_functions (enum)`**: İşlev çağrısı sayılarını ve kullanım zamanının izlenmesini sağlar. Sağlanan `pl` değerini yalnızca prosedürel dil işlevlerini izlemek için, `all` değerini ise SQL ve C dili işlevlerini de izlemek kullanın. Varsayılan, işlev istatistikleri izlemeyi devre dışı bırakan `none` değeridir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

{% include callout.html content="**`stats_temp_directory (string)`**: Geçici istatistik verilerinin saklanacağı dizini ayarlar. Bu, veri dizinine bağıl bir yol veya mutlak bir yol olabilir. Varsayılan değeri `pg_stat_tmp`'dir. Bunu RAM tabanlı bir dosya sistemine ayarlamak fiziksel I / O gereksinimlerini azaltarak performansın artmasını sağlayabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### İstatistik İzleme

{% include callout.html content="**`log_statement_stats (boolean) / log_parser_stats (boolean) / log_planner_stats (boolean) / log_executor_stats (boolean)`**: Her sorgu için, ilgili modülün performans istatistiklerini sunucu günlüğüne gönderin. Bu, Unix `getrusage ()` işletim sistemi aracına benzer kaba bir profilleme aracıdır. `log_statement_stats` bütün ifade istatistiklerini rapor ederken diğerleri modül bazlı istatistikleri raporlar. `log_statement_stats`, modül bazlı parametrelerin herhangi biriyle birlikte etkinleştirilemez. Bu parametrelerin tümü varsayılan olarak devre dışıdır. Bu ayarları yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-statistics.html)

{% include links.html %}
