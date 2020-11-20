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

Dış dünya tarafından erişilebilen her sunucuda olduğu gibi, PostgreSQL'in ayrı bir kullanıcı hesabı altında çalıştırılması tavsiye edilir. Bu kullanıcı hesabı yalnızca sunucu tarafından yönetilen verilere sahip olmalı ve diğer arka plan süreçleri ile paylaşılmamalıdır. Özellikle, bu kullanıcı hesabının PostgreSQL çalıştırılabilir dosyalarına sahip olmaması, risk altındaki bir sunucu işleminin bu çalıştırılabilir dosyaları değiştirememesini sağlamak için tavsiye edilir.

PostgreSQL'in önceden paketlenmiş sürümleri, genellikle paket kurulumu sırasında otomatik olarak uygun bir kullanıcı hesabı oluşturur.

Sisteminize bir Unix kullanıcı hesabı eklemek için `useradd` veya `adduser` komutu kullanılır. **postgres** kullanıcı adı sıklıkla kullanılır ve bu dokümanda varsayılandır, ancak isterseniz başka bir ad da kullanabilirsiniz.

{% include links.html %}
