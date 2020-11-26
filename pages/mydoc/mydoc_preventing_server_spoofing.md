---
title: "Sunucu Sahtekarlığını Önleme"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 24, 2020
summary: "Sunucu Sahtekarlığını Önleme"
sidebar: mydoc_sidebar
permalink: mydoc_preventing_server_spoofing.html
folder: mydoc
---

## Sunucu Sahtekarlığını Önleme

Sunucu çalışırken kötü niyetli bir kullanıcının mevcut veritabanı sunucusunu taklit etmesi mümkün değildir, fakat sunucu çalışmadığında yerel bir kullanıcının kendi sunucusunu başlatarak normal sunucuyu taklit etmesi mümkündür. Sahte sunucu, istemciler tarafından gönderilen parolaları ve sorguları okuyabilir, ancak herhangi bir veri döndüremez çünkü `PGDATA` dizini, dizin izinleri nedeniyle hala güvende olacaktır. Adres sahteciliği mümkündür çünkü herhangi bir kullanıcı bir veritabanı sunucusu başlatabilir ve istemci, özel olarak yapılandırılmadıkça geçersiz sunucuyu tespit edemez.

Yerel bağlantıların sahteciliğini önlemenin bir yolu, yalnızca güvenilir bir yerel kullanıcının yazma iznine sahip olduğu bir [Unix domain socket](https://www.postgresql.org/docs/13/runtime-config-connection.html#GUC-UNIX-SOCKET-DIRECTORIES) dizini (unix_socket_directories) kullanmaktır. Böylece kötü niyetli bir kullanıcının o dizinde kendi soket dosyasını oluşturması engellenir. Bazı uygulamaların soket dosyası için hala `/tmp` referansına başvurabileceğinden ve tehdide açık olabileceğinden endişeleniyorsanız, işletim sistemi başlatılırken yeniden konumlandırılan soket dosyasını işaret eden sembolik bir bağlantı `/tmp/.s.PGSQL.5432` oluşturun. Sembolik bağlantının kaldırılmasını önlemek için `/tmp` temizleme script'inizi de değiştirmeniz gerekebilir.

Yerel bağlantılar için bir diğer yol ise istemcilerin sokete bağlı sunucu sürecinin sahibini belirtmek için [`requirepeer`](https://www.postgresql.org/docs/13/libpq-connect.html#LIBPQ-CONNECT-REQUIREPEER) kullanmasıdır.

TCP bağlantılarında sahtekarlığı önlemek için, ya SSL sertifikaları kullanın ve istemcilerin sunucunun sertifikasını kontrol ettiğinden emin olun ya da GSSAPI şifrelemesini kullanın (veya ayrı bağlantılarda iseler ikisini birden).

SSL kullanarak sahtekarlığı önlemek için sunucunun yalnızca `hostssl` [bağlantılarını](https://www.postgresql.org/docs/13/auth-pg-hba-conf.html) kabul edecek şekilde yapılandırılması ve SSL anahtarı ve sertifika [dosyalarına](https://www.postgresql.org/docs/13/ssl-tcp.html) sahip olması gerekir. TCP istemcisi, `sslmode=verify-ca` veya `verify-full` kullanarak bağlanmalı ve ilgili root sertifika dosyasını kurmalıdır [](https://www.postgresql.org/docs/13/ssl-tcp.html).

GSSAPI ile sahtekarlığı önlemek için, sunucunun yalnızca `hostgssenc` bağlantılarını kabul edecek [](https://www.postgresql.org/docs/13/auth-pg-hba-conf.html) ve bunlarla `gss` kimlik doğrulamasını kullanacak şekilde yapılandırılması gerekir. TCP istemcisi `gssencmode = require` kullanarak bağlanmalıdır.

{% include links.html %}
