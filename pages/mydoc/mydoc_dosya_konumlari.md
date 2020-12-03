---
title: "Dosya Konumları"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 26, 2020
summary: "Dosya Konumları"
sidebar: mydoc_sidebar
permalink: mydoc_dosya_konumlari.html
folder: mydoc
---

## Dosya Konumları

Önceki bölümlerde bahsedilen *postgresql.conf* dosyasına ek olarak, PostgreSQL istemci kimlik doğrulamasını kontrol eden iki yapılandırma dosyası daha kullanır (kullanımları [İstemci Kimlik Doğrulaması]("") bölümünde ele alınmıştır.). Bu yapılandırma dosyaları varsayılan olarak veritabanı kümesinin veri dizininde depolanır. Bu bölümde ele alınan parametreler, yapılandırma dosyalarının başka bir yere konumlandırılmasına olanak tanır. Bazı durumlarda bunu yapmak yönetimi kolaylaştırır. Ayrı tutulduklarında yapılandırma dosyalarının düzgün şekilde yedeklenmesini sağlamak genellikle daha kolaylaşır.

{% include callout.html content="`data_directory (string)`: Veri depolaması için kullanılacak dizini belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`config_file (string)`: Ana sunucu yapılandırma dosyasını belirtir (geleneksel *postgresql.conf* olarak isimlendirilir). Bu parametre yalnızca `postgres` komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`hba_file (string)`: host-based kimlik doğrulama işlemleri için yapılandırma dosyasını belirtir (geleneksel *pg_hba.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`ident_file (string)`: Kullanıcı adı eşlemesi için yapılandırma dosyasını belirtir (geleneksel *pg_ident.conf* olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Ayrıntılı bilgi için [User Name Maps](https://www.postgresql.org/docs/current/auth-username-maps.html) bölümüne bakın." type="primary" %}

{% include callout.html content="`external_pid_file (string)`: Sunucunun, sunucu yönetim programları tarafından kullanılmak üzere oluşturması gereken süreç kimliği (PID) dosyasının adını belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

Varsayılan bir kurulumda, yukarıda bahsedilen parametrelerin hiçbiri açık bir şekilde ayarlanmamıştır. Bunun yerine, veri dizini `-D` parametresi veya `PGDATA` ortam değişkeni ile belirtilir ve yapılandırma dosyalarının tümü belirtilen veri dizininde bulunur.

Yapılandırma dosyaları veri dizininden başka bir yerde tutulmak istenirse `postgres -D` komut satırı seçeneği veya `PGDATA` ortam değişkeni yapılandırma dosyalarını içeren dizini göstermeli ve `data_directory` parametresi veri dizininin asıl konumunu gösterecek şekilde ayarlanmalıdır. data_directory, veri dizininin konumu için `-D` ve `PGDATA`'yı geçersiz kılabilir ancak yapılandırma dosyalarının konumu için bu geçerli değildir.

Dilerseniz konfigürasyon dosyası isimlerini ve konumlarını `config_file`, `hba_file`, `ident_file` parametrelerini kullanarak ayrı ayrı belirtebilirsiniz. `config_file`, sadece `postgres` komut satırında belirtilebilirken diğerleri ana yapılandırma dosyası içinde ayarlanabilir. Üç parametrenin tümü ve *data_directory* açık bir şekilde ayarlanmışsa `-D` veya `PGDATA` belirtilmesi gerekli değildir.

Bu parametreler ayarlanırken relative path `postgres`'in başlatıldığı dizine göre yorumlanacaktır.

{% include links.html %}
