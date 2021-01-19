---
title: "Sunucu Sahtekarlığını Önleme"
tags: [PostgreSQL]
keywords: postgres, güvenlik, ssl, GSSAPI
last_updated: December 21, 2020
sidebar: mydoc_sidebar
permalink: mydoc_preventing_server_spoofing.html
folder: mydoc
---

## Sunucu Sahtekarlığını Önleme

Sunucu çalışırken kötü niyetli bir kullanıcının mevcut veritabanı sunucusunu taklit etmesi mümkün değildir, fakat sunucu çalışmadığında yerel bir kullanıcının kendi sunucusunu başlatarak normal sunucuyu taklit etmesi mümkündür. Sahte sunucu, istemciler tarafından gönderilen parolaları ve sorguları okuyabilir, ancak herhangi bir veri döndüremez. ``PGDATA`` dizini, dizin izinleri nedeniyle hala güvende olur. Herhangi bir kullanıcı bir veritabanı sunucusu başlatabildiğinden adres sahteciliği mümkündür. İstemci, özel olarak yapılandırılmadıkça geçersiz sunucuyu tespit edemez.

Yalnızca güvenilir bir local kullanıcının yazma iznine sahip olduğu bir [Unix domain socket](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-UNIX-SOCKET-DIRECTORIES) dizini (unix_socket_directories) kullanarak Local bağlantıların sahteciliğini önlenebilir. Bu şekilde, kötü niyetli kullanıcıların bu dizinde kendi soket dosyasını oluşturması engellenir.

Local bağlantılar için bir diğer yol istemcilerin sokete bağlı sunucu sürecinin sahibini belirtmek için [`requirepeer`](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNECT-REQUIREPEER) kullanmasıdır.

TCP bağlantıları üzerindeki sahtekarlığı önlemek için SSL sertifikaları veya GSSAPI şifrelemesi kullanılır (veya ayrı bağlantılarda iseler ikisini birden).

SSL ile sahtekarlığı önlemek için sunucunun, yalnızca [`hostssl`](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html) bağlantılarını kabul edecek şekilde yapılandırılması ve SSL anahtarı ve sertifika [dosyalarına](https://www.postgresql.org/docs/current/ssl-tcp.html) sahip olması gerekir. TCP istemcisi, `sslmode=verify-ca` veya `verify-full` kullanarak bağlanmalı ve ilgili kök sertifika dosyasını kurmalıdır [bkz.](https://www.postgresql.org/docs/current/ssl-tcp.html)

GSSAPI ile sahtekarlığı önlemek için, sunucunun yalnızca `hostgssenc` bağlantılarını kabul edecek şekilde yapılandırılması gerekir. TCP istemcisi `gssencmode = require` kullanarak bağlanmalıdır.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/preventing-server-spoofing.html)

{% include links.html %}
