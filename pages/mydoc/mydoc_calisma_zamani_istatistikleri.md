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

#### `track_activities`

{% include parameter_info.html parametre="track_activities" %}

{% include callout.html content=" **Herbir oturumda komutların yürütülmesi hakkında bilgi toplar.** Bu parametre varsayılan olarak açıktır. Sağladığı bilgiler tüm kullanıcılar tarafından görülemez. Yalnızca süper kullanıcılar ve rapor edilen oturumun sahibi olan kullanıcı tarafından görülebilir. Bu nedenle bir güvenlik riski oluşturmaz. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `track_activity_query_size`

{% include parameter_info.html parametre="track_activity_query_size" %}

{% include callout.html content=" **`pg_stat_activity.query` alanı için ayrılan boyutu bayt cinsinden ayarlar.** Bu değer birimsiz belirtilirse bayt olarak alınır. Öntanımlı değeri 1024 bayttır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

#### `track_counts`

{% include parameter_info.html parametre="track_counts" %}

{% include callout.html content=" **Veritabanı etkinliği ile ilgili istatistiklerin toplanmasını sağlar.** Bu parametre varsayılan olarak açıktır. Autovacuum daemon toplanan bu bilgilere ihtiyaç duyar. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `track_io_timing`

{% include parameter_info.html parametre="track_io_timing" %}

{% include callout.html content=" **Veritabanı I / O etkinliğinin zamanlama istatistiklerini toplar.** Bu parametre varsayılan olarak kapalıdır, çünkü işletim sistemini geçerli saat için tekrar tekrar sorgulayacağından bazı platformlarda önemli ek yüklere neden olabilir. Sisteminizdeki zamanlamanın ek yükünü ölçmek için [`pg_test_timing`](https://www.postgresql.org/docs/current/pgtesttiming.html) aracını kullanabilirsiniz. I / O zamanlama bilgisi [`pg_stat_database`](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-VIEW) içinde, `BUFFERS` opsiyonu kullanıldığında [EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html) çıktısında ve [`pg_stat_statements`](https://www.postgresql.org/docs/current/pgstatstatements.html) tarafından görüntülenir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `track_functions`

{% include parameter_info.html parametre="track_functions" %}

{% include callout.html content=" **İşlev çağrısı sayılarını ve kullanım zamanının izlenmesini sağlar.** Sağlanan `pl` değerini yalnızca prosedürel dil işlevlerini izlemek için, `all` değerini ise SQL ve C dili işlevlerini de izlemek kullanın. Varsayılan, işlev istatistikleri izlemeyi devre dışı bırakan `none` değeridir. Bu ayarı yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

#### `stats_temp_directory`

{% include parameter_info.html parametre="stats_temp_directory" %}

{% include callout.html content=" **Geçici istatistik verilerinin saklanacağı dizini ayarlar.** Bu, veri dizinine bağıl bir yol veya mutlak bir yol olabilir. Varsayılan değeri `pg_stat_tmp`'dir. Bunu RAM tabanlı bir dosya sistemine ayarlamak fiziksel I / O gereksinimlerini azaltarak performansın artmasını sağlayabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### İstatistik İzleme

#### `autovacuum_log_statement_stats / log_parser_stats / log_planner_stats / log_executor_stats / vacuum_cost_limit`

{% include parameter_info.html parametre="autovacuum_log_statement_stats" %}
{% include parameter_info.html parametre="log_parser_stats" %}
{% include parameter_info.html parametre="log_planner_stats" %}
{% include parameter_info.html parametre="log_executor_stats" %}
{% include parameter_info.html parametre="vacuum_cost_limit" %}

{% include callout.html content=" **Her sorgu için, ilgili modülün performans istatistiklerini sunucu günlüğüne gönderin.** Bu, Unix `getrusage ()` işletim sistemi aracına benzer kaba bir profilleme aracıdır. `log_statement_stats` bütün ifade istatistiklerini rapor ederken diğerleri modül bazlı istatistikleri raporlar. `log_statement_stats`, modül bazlı parametrelerin herhangi biriyle birlikte etkinleştirilemez. Bu parametrelerin tümü varsayılan olarak devre dışıdır. Bu ayarları yalnızca süper kullanıcılar değiştirebilir." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-statistics.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
