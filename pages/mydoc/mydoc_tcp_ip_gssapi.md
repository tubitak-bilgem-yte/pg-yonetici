---
title: "GSSAPI Şifreleme ile TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 25, 2020
summary: "GSSAPI Şifreleme ile TCP / IP Bağlantıları"
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_gssapi.html
folder: mydoc
---

## GSSAPI Şifreleme ile TCP / IP Bağlantıları

PostgreSQL, artırılmış güvenlik gereksinimleriniz için istemci / sunucu iletişimini şifrelemek üzere yerel GSSAPI desteğine sahiptir. GSSAPI şifreleme, MIT krb5 gibi bir GSSAPI uygulamasının hem istemci hem de sunucu sistemlerine kurulması ve PostgreSQL derlenirken ilgili desteğin etkinleştirilmiş olmasıyla kullanılabilir.

### Temel Kurulum

PostgreSQL sunucusu, aynı TCP bağlantı noktasında hem normal hem de GSSAPI ile şifrelenmiş bağlantıları dinler ve şifreleme (ve kimlik doğrulama için) için GSSAPI kullanılıp kullanılmayacağını bağlanan istemciyle görüşür. Varsayılan olarak, bu karar istemciye bağlıdır. Bağlantıların bazıları veya tümü için GSSAPI kullanımını gerektirecek şekilde sunucunun ayarlanması hakkında [pg_hba.conf Dosyası]("") bölümüne bakın.

Yapılandırma ayarlarıyla ilgili daha fazla bilgi için [GSSAPI Kimlik Doğrulaması]("") bölümüne bakın.

{% include links.html %}
