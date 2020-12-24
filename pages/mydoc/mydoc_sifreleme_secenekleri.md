---
title: "Şifreleme Seçenekleri"
tags: [PostgreSQL]
keywords: postgres, SCRAM, MD5, Pgcrypto, SSL, Encryption, Authentication
last_updated: December 21, 2020
sidebar: mydoc_sidebar
permalink: mydoc_sifreleme_secenekleri.html
folder: mydoc
---

## Şifreleme Seçenekleri

PostgreSQL çeşitli düzeylerde sağladığı şifreleme imkanlarıyla kullanıcılara esneklik sunar.

{% include callout.html content="**`Parola Şifreleme`**: Veritabanı kullanıcı parolaları hash'lenerek saklanır. Böylece yönetici, kullanıcı parolalarını göremez. `SCRAM` ve `MD5` şifreleme kullanımında, şifrelenmemiş parola sunucuda geçici olarak bile tutulmaz. Bir İnternet standardı olan SCRAM, PostgreSQL'e özgü MD5 kimlik doğrulama protokolünden daha güvenlidir." type="primary" %}

{% include callout.html content="**`Sütun Bazında Şifreleme`**: [Pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html) modülü belirli alanların şifreli olarak saklanması için kullanılır. Verilerinizin bir kısmının hassas olduğu durumlarda kullanılır. İstemci decryption key sağlar ve verilerin şifresi sunucuda çözülürek istemciye gönderilir." type="primary" %}

{% include callout.html content="**`Data Partition Encryption`**: Dosya sistemi ve blok seviyesinde depolama şifrelemesi yapmak mümkündür. eCryptfs ve EncFS linux dosya sistemi şifreleme seçeneklerindendir. dm-crypt + LUKS, Linux'ta blok seviyesi ve tam disk şifreleme seçenekleridir.<br/><br/>

Bu mekanizma, sürücüler veya bilgisayarın tamamı çalınırsa şifrelenmemiş verilerin sürücülerden okunmasını engeller. Dosya sistemi mount edilmişken saldırılara karşı koruma sağlamaz. " type="primary" %}

{% include callout.html content="**`Ağ Üzerindeki Verileri Şifreleme`**: SSL, ağ üzerinden gönderilen verileri şifreler: parola, sorgu ve döndürülen veriler. Hangi host'un şifrelenmemiş bağlantıları kullanacağı, hangisinin SSL şifreli bağlantılar gerektirdiği `pg_hba.conf` dosyasında belirtilir.<br/><br/>

GSSAPI şifreli bağlantılar, sorgular ve döndürülen veriler dahil olmak üzere ağ üzerinden gönderilen tüm verileri şifreler. Hangi host'un şifrelenmemiş, hangisini GSSAPI ile şifrelenmiş bağlantılar gerektirdiği `pg_hba.conf` dosyasında belirtilir.<br/><br/>

Akışı şifrelemek için Stunnel ve SSH da kullanılabilir." type="primary" %}

{% include callout.html content="**`SSL Host Authentication`**: Hem istemcinin hem de sunucunun birbirine SSL sertifikası sağlaması mümkündür. Her iki tarafta da fazladan yapılandırma gerektirse de parola kullanımından daha güçlü kimlik doğrulaması sağlar. İstemci ile sunucu arasındaki bir bilgisayarın sunucu gibi davranarak tüm verileri okuyup ilettiği 'man in the middle' saldırılarını önler." type="primary" %}

{% include callout.html content="**`Client-Side Encryption`**: Sistem yöneticisine güvenilmediğinde istemcinin verileri şifrelemesi gerekir. Veriler, sunucuya gönderilmeden önce istemcide şifrelenir. Veritabanı sonuçlarının kullanılmadan önce istemcide şifresinin çözülmesi gerekir." type="primary" %}

{% include links.html %}
