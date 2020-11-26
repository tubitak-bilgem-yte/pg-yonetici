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

Önceki bölümlerde bahsedilen `postgresql.conf` dosyasına ek olarak, PostgreSQL istemci kimlik doğrulamasını kontrol eden, manuel olarak düzenlenmiş iki yapılandırma dosyası daha kullanır (Bu dosyalar [İstemci Kimlik Doğrulaması](https://www.postgresql.org/docs/13/client-authentication.html) bölümden ele alınmıştır.). Üç yapılandırma dosyası varsayılan olarak veritabanı kümesinin veri dizininde depolanır. Bu bölümde ele alınan parametreler yapılandırma dosyalarının başka bir yere yerleştirilmesine izin verir. (Bunu yapmak yönetimi kolaylaştırabilir. Özellikle, ayrı tutulduklarında yapılandırma dosyalarının düzgün şekilde yedeklenmesini sağlamak genellikle daha kolaydır.)

`data_directory (string):`

Veri depolaması için kullanılacak dizini belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.

`config_file (string):`

Ana sunucu yapılandırma dosyasını belirtir (geleneksel `postgresql.conf` olarak isimlendirilir). Bu parametre yalnızca `postgres` komut satırından ayarlanabilir.

`hba_file (string):`

Ana bilgisayar tabanlı kimlik doğrulama için yapılandırma dosyasını belirtir (geleneksel `pg_hba.conf` olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.

`ident_file (string):`

Kullanıcı adı eşlemesi için yapılandırma dosyasını belirtir (geleneksel `pg_ident.conf` olarak isimlendirilir). Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Ayrıntılı bilgi için [User Name Maps](https://www.postgresql.org/docs/current/auth-username-maps.html) bölümüne bakın.

`external_pid_file (string):`

Sunucunun, sunucu yönetim programları tarafından kullanılmak üzere oluşturması gereken süreç kimliği (PID) dosyasının adını belirtir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir.

Yukarıdaki parametrelerin hiçbiri varsayılan bir kurulumda açık bir şekilde ayarlanmamıştır. Bunun yerine, veri dizini `-D` parametresi veya `PGDATA` ortam değişkeni ile belirtilir ve yapılandırma dosyalarının tümü belirtilen veri dizininde bulunur.

Dilerseniz konfigürasyon dosyası isimlerini ve konumlarını `config_file`, `hba_file`, `ident_file` parametrelerini kullanarak ayrı ayrı belirtebilirsiniz. `config_file` sadece `postgres` komut satırında belirtilebilirken diğerleri ana yapılandırma dosyası içinde ayarlanabilir. Üç parametrenin tümü ve `data_directory` açık bir şekilde ayarlanmışsa `-D` veya `PGDATA` belirtilmesi gerekli değildir.

Bu parametreler ayarlanırken `postgres`'in başlatıldığı dizine göre göreceli bir yol yorumlanacaktır.

{% include links.html %}
