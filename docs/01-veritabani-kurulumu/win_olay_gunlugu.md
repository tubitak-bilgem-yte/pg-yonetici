---
title: Windows'ta Olay Günlüğünü Kaydetme
layout: default
parent: Veritabanı Kurulumu
nav_order: 3
---

## Windows'ta Olay Günlüğünü Kaydetme

Windows olay günlüğü kütüphanesini ( event log library ) kaydetmek için şu komutu verin:

```cmd
regsvr32 pgsql_library_directory/pgevent.dll
```

Bu, PostgreSQL adlı varsayılan olay kaynağı altında event viewer tarafından kullanılan kayıt defteri girdilerini oluşturur.

Farklı bir olay kaynağı ismi belirtmek için [event_source](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-EVENT-SOURCE), `/n` ve `/i` seçeneklerini kullanın:

```cmd
regsvr32 /n /i:event_source_name pgsql_library_directory/pgevent.dll
```

İşletim sisteminden olay günlüğü kütüphanesi kaydı silmek için şu komutu verin:

```cmd
regsvr32 /u [/i:event_source_name] pgsql_library_directory/pgevent.dll
```

{% include note.html content=" Veritabanı sunucusunda olay günlüğünü etkinleştirmek için, `postgresql.conf` dosyasında `log_destination` öğesini **eventlog** içerecek şekilde değiştirin." %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/event-log-registration.html)

{% include links.html %}
