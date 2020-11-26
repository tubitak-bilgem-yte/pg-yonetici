---
title: "PostgreSQL Kullanıcı Hesabı"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 20, 2020
summary: "PostgreSQL Kullanıcı Hesabı"
sidebar: mydoc_sidebar
permalink: mydoc_kullanici_hesabi.html
folder: mydoc
---

## PostgreSQL Kullanıcı Hesabı

Dış dünya tarafından erişilebilen her sunucuda olduğu gibi, PostgreSQL'in ayrı bir kullanıcı hesabı altında çalıştırılması tavsiye edilir. Bu kullanıcı hesabı, yalnızca sunucu tarafından yönetilen verilere sahip olmalı ve diğer arka plan süreçleriyle paylaşılmamalıdır. Özellikle, bu hesabın PostgreSQL çalıştırılabilir dosyalarına sahip olmaması, risk altındaki bir sunucu sürecinin ilgili çalıştırılabilir dosyalarda değişiklik yapmasının engellenmesi için tavsiye edilir.

PostgreSQL'in önceden paketlenmiş sürümleri, genellikle paket kurulumu sırasında otomatik olarak uygun bir kullanıcı hesabı oluşturur.

Sisteme bir Unix kullanıcı hesabı eklemek için `useradd` veya `adduser` komutu kullanılır. **postgres** kullanıcı adı sıklıkla kullanılır ve bu dokümanda varsayılandır. Dilerseniz başka bir ad da kullanabilirsiniz.

{% include links.html %}
