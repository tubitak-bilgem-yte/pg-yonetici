---
title: "Dosya Konumları"
layout: default
parent: Veritabanı Yapılandırması
nav_order: 3
---

## Dosya Konumları

PostgreSQL; *postgresql.conf* dosyasına ek olarak, istemci kimlik doğrulamasını kontrol eden iki yapılandırma dosyası daha kullanır. Bu yapılandırma dosyaları varsayılan olarak veritabanı kümesinin veri dizininde depolanır. Bu bölümde ele alınan parametreler, yapılandırma dosyalarının başka bir yerde konumlandırılmasına olanak tanır. Bu, bazı durumlarda yönetimi kolaylaştırır. Yapılandırma dosyalarının ayrı tutulması yedeklenmesini kolaylaşırır.

### `data_directory`

{% include parameter_info.html parametre="data_directory" %}

{% include callout.html content=" **Verilerin depolanacağı dizini belirtir.** Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

### `config_file`

{% include parameter_info.html parametre="config_file" %}

{% include callout.html content=" **Ana sunucu yapılandırma dosyasını belirtir** (geleneksel *postgresql.conf* olarak isimlendirilir). Bu parametre yalnızca `postgres` komut satırından ayarlanabilir." type="primary" %}

### `hba_file`

{% include parameter_info.html parametre="hba_file" %}

{% include callout.html content=" **host-based kimlik doğrulama işlemleri için yapılandırma dosyasını belirtir** (geleneksel *pg_hba.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

### `ident_file`

{% include parameter_info.html parametre="ident_file" %}

{% include callout.html content=" **User name mapping için yapılandırma dosyasını belirtir** (geleneksel *pg_ident.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir bkz. [User Name Maps](https://www.postgresql.org/docs/current/auth-username-maps.html)." type="primary" %}

### `external_pid_file`

{% include parameter_info.html parametre="external_pid_file" %}

{% include callout.html content=" **Sunucunun, sunucu yönetim programlarının kullanacağı süreç kimliği (PID) dosyası adını belirtir.** Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

Varsayılan bir kurulumda, yukarıda bahsedilen parametrelerin hiçbiri açık şekilde ayarlanmamıştır. Veri dizini, `-D` parametresi veya `PGDATA` ortam değişkeni ile belirtilir ve yapılandırma dosyaları belirtilen bu veri dizininde tutulur.

Konfigürasyon dosyası isim ve konumları `config_file`, `hba_file`, `ident_file` parametreleri kullanılarak ayrı ayrı belirtilebilir. `config_file`, sadece `postgres` komut satırında belirtilebilirken diğerleri yapılandırma dosyası içinde ayarlanabilir. Üç parametrenin tümü ve *data_directory* açık bir şekilde ayarlanmışsa `-D` veya `PGDATA` belirtilmesi gerekli değildir.

Bu parametreler ayarlanırken relative path `postgres`'in başlatıldığı dizine göre yorumlanacaktır.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-file-locations.html)

[2]. [postgresqlco.nf](https://postgresqlco.nf)

{% include links.html %}
