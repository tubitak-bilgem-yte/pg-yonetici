---
title: "Windows'ta Olay Günlüğünü Kaydetme"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 26, 2020
summary: "Windows'ta Olay Günlüğünü Kaydetme"
sidebar: mydoc_sidebar
permalink: mydoc_win_olay_gunlugu.html
folder: mydoc
---

## Windows'ta Olay Günlüğünü Kaydetme

İşletim sistemiyle bir Windows olay günlüğü kütüphanesini ( event log library ) kaydetmek için şu komutu verin:

```cmd
regsvr32 pgsql_library_directory/pgevent.dll
```

`PostgreSQL` isimli varsayılan olay kaynağı altında olay görüntüleyici tarafından kullanılan kayıt defteri girdilerini ( registry entries )oluşturur.

Farklı bir olay kaynağı ismi belirtmek için [event_source](https://www.postgresql.org/docs/13/runtime-config-logging.html#GUC-EVENT-SOURCE), `/n` ve `/i` seçeneklerini kullanın:

```cmd
regsvr32 /n /i:event_source_name pgsql_library_directory/pgevent.dll
```

İşletim sisteminden olay günlüğü kütüphanesi kaydı silmek ( unregister ) için şu komutu verin:

```cmd
regsvr32 /u [/i:event_source_name] pgsql_library_directory/pgevent.dll
```

{% include note.html content="Veritabanı sunucusunda olay günlüğünü etkinleştirmek için, `postgresql.conf` dosyasında `log_destination` öğesini **eventlog** içerecek şekilde değiştirin." %}

{% include links.html %}
