---
title: "GSSAPI Şifreleme ile TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres, GSSAPI, tcp
last_updated: December 23, 2020
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_gssapi.html
folder: mydoc
---

## GSSAPI Şifreleme ile TCP / IP Bağlantıları

PostgreSQL, artırılmış güvenlik gereksinimleriniz için istemci / sunucu iletişimini şifrelemek üzere GSSAPI desteğine sahiptir.

### Temel Kurulum

PostgreSQL sunucusu, aynı TCP bağlantı noktasında hem normal hem de GSSAPI ile şifrelenmiş bağlantıları dinler. Şifreleme ve kimlik doğrulama için GSSAPI kullanılıp kullanmayacağını bağlanan istemciyle görüşür. Varsayılan olarak, bu karar istemciye bağlıdır. Sunucunun, GSSAPI kullanımını bağlantıların bazıları veya tümü için gerektirecek şekilde ayarlanması mümkündür. Daha fazla bilgi için [bkz.](https://www.postgresql.org/docs/current/gssapi-auth.html)

{% include links.html %}
