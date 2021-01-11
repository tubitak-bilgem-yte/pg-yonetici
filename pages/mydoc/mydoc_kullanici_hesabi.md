---
title: "PostgreSQL Kullanıcı Hesabı"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 21, 2020
sidebar: mydoc_sidebar
permalink: mydoc_kullanici_hesabi.html
folder: mydoc
---

## PostgreSQL Kullanıcı Hesabı

Dış dünya tarafından erişilebilen her sunucuda olduğu gibi, PostgreSQL'in ayrı bir kullanıcı altında çalıştırılması tavsiye edilir. Bu kullanıcı, yalnızca sunucu tarafından yönetilen verilere sahip olmalı ve diğer arka plan süreçleriyle paylaşılmamalıdır. Risk altındaki bir sunucu sürecinin ilgili çalıştırılabilir dosyalarda değişiklik yapması engellemek için bu kullanıcı PostgreSQL çalıştırılabilir dosyalarına sahip olmamalıdır.

PostgreSQL'in önceden paketlenmiş sürümleri, genellikle paket kurulumu sırasında otomatik olarak uygun bir kullanıcı hesabı oluşturur. Standart kurulumda bu **postgres** kullanıcısıdır.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/postgres-user.html)

{% include links.html %}
