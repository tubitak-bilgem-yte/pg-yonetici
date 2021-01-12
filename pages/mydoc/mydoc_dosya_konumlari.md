---
title: "Dosya Konumları"
tags: [PostgreSQL]
keywords: postgres, data_directory, config_file, ident_file, hba_file
last_updated: January 6, 2021
sidebar: mydoc_sidebar
permalink: mydoc_dosya_konumlari.html
folder: mydoc
---

## Dosya Konumları

PostgreSQL; *postgresql.conf* dosyasına ek olarak, istemci kimlik doğrulamasını kontrol eden iki yapılandırma dosyası daha kullanır. Bu yapılandırma dosyaları varsayılan olarak veritabanı kümesinin veri dizininde depolanır. Bu bölümde ele alınan parametreler, yapılandırma dosyalarının başka bir yerde konumlandırılmasına olanak tanır. Bu, bazı durumlarda yönetimi kolaylaştırır. Yapılandırma dosyalarının ayrı tutulması yedeklenmesini kolaylaşırır.

{% include callout.html content=" **`data_directory (string)`**: Verilerin depolanacağı dizini belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`config_file (string)`**: Ana sunucu yapılandırma dosyasını belirtir (geleneksel *postgresql.conf* olarak isimlendirilir). Bu parametre yalnızca `postgres` komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`hba_file (string)`**: host-based kimlik doğrulama işlemleri için yapılandırma dosyasını belirtir (geleneksel *pg_hba.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`ident_file (string)`**: User name mapping için yapılandırma dosyasını belirtir (geleneksel *pg_ident.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir bkz. [User Name Maps](https://www.postgresql.org/docs/current/auth-username-maps.html)." type="primary" %}

{% include callout.html content=" **`external_pid_file (string)`**: Sunucunun, sunucu yönetim programlarının kullanacağı süreç kimliği (PID) dosyası adını belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

Varsayılan bir kurulumda, yukarıda bahsedilen parametrelerin hiçbiri açık şekilde ayarlanmamıştır. Veri dizini, `-D` parametresi veya `PGDATA` ortam değişkeni ile belirtilir ve yapılandırma dosyaları belirtilen bu veri dizininde tutulur.

Konfigürasyon dosyası isim ve konumları `config_file`, `hba_file`, `ident_file` parametreleri kullanılarak ayrı ayrı belirtilebilir. `config_file`, sadece `postgres` komut satırında belirtilebilirken diğerleri yapılandırma dosyası içinde ayarlanabilir. Üç parametrenin tümü ve *data_directory* açık bir şekilde ayarlanmışsa `-D` veya `PGDATA` belirtilmesi gerekli değildir.

Bu parametreler ayarlanırken relative path `postgres`'in başlatıldığı dizine göre yorumlanacaktır.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-file-locations.html)

{% include links.html %}
