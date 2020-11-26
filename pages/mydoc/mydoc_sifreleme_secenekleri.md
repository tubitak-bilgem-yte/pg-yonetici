---
title: "Şifreleme Seçenekleri"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 25, 2020
summary: "Şifreleme Seçenekleri"
sidebar: mydoc_sidebar
permalink: mydoc_sifreleme_secenekleri.html
folder: mydoc
---

## Şifreleme Seçenekleri

PostgreSQL, veritabanı sunucu hırsızlığı, etik olmayan yöneticiler ve güvensiz ağlara karşı verilerinizi ifşa edilmeden korunması için çeşitli düzeylerde şifreleme imkanları ve esneklik sunar. Ayrıca şifreleme tıbbi kayıtlar veya finansal işlemler gibi hassas verilerin güvenliğini sağlamak için de gerekebilir.

`Parola Şifreleme`

Veritabanı kullanıcı parolaları hash'lenerek ([parola_ şifreleme](https://www.postgresql.org/docs/13/runtime-config-connection.html#RUNTIME-CONFIG-CONNECTION-AUTHENTICATION) ayarıyla belirlenir) saklanır. Böylece yönetici kullanıcıya atanan asıl parolayı anlayamaz. İstemci kimlik doğrulaması ( client authentication ) için `SCRAM` veya `MD5` şifrelemesi kullanılıyorsa, şifrelenmemiş parola sunucuda geçici olarak bile tutulmaz çünkü bunu istemci ağ üzerinden gönderilmeden önce şifreler. SCRAM, bir İnternet standardıdır ve PostgreSQL'e özgü MD5 kimlik doğrulama protokolünden daha güvenlidir. Bu sepeble tavsiye edilebilir bir seçenektir.

`Belirli Sütunlar İçin Şifreleme`

[Pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html) modülü belirli alanların şifreli olarak saklanması için kullanılır. Verilerinizin bir kısmının hassas olduğu durumlarda kullanılır. İstemci şifre çözme anahtarını sağlar ( decryption key ) ve verilerin şifresi sunucuda çözülürek istemciye gönderilir.

Şifresi çözülen veriler ve şifre çözme anahtarı, şifresi çözülürken ve istemci ile sunucu arasında iletilirken sunucuda kısa bir süre bulunur. Bu durum, veri ve anahtarların sistem yöneticisi gibi veritabanı sunucusuna tam erişimi olan biri tarafından ele geçirilebileceği kısa bir an olduğu anlamına gelir.

`Data Partition Encryption` ( Veri Bölümü Şifreleme )

Dosya sistemi seviyesinde ve blok seviyesinde depolama şifreleme yapmak mümkündür. Linux dosya sistemi şifreleme seçenekleri arasında eCryptfs ve EncFS bulunur, FreeBSD ise PEFS kullanır. Blok seviyesi veya tam disk şifreleme seçenekleri arasında Linux'ta dm-crypt + LUKS ve FreeBSD'de geli ve gbde GEOM modülleri bulunur. Windows dahil birçok başka işletim sistemi bu işlevi destekler.

Bu mekanizma, sürücüler veya bilgisayarın tamamı çalınırsa şifrelenmemiş verilerin sürücülerden okunmasını engeller. Dosya sistemi takılıyken saldırılara karşı koruma sağlamaz, çünkü bağlandığında işletim sistemi verilerin şifrelenmemiş bir görünümünü sağlar. Ancak, dosya sistemini bağlamak için, şifreleme anahtarının işletim sistemine gönderilmesinde bir yola ihtiyacınız vardır. Bazı durumlarda anahtar diski bağlayan ana bilgisayarda saklanır.

`Ağ Üzerindeki Verileri Şifreleme`

SSL bağlantıları ağ üzerinden gönderilen tüm verileri şifreler: parola, sorgular ve döndürülen veriler. `pg_hba.conf` dosyası yöneticilerin hangi ana bilgisayarların ( host ) şifrelenmemiş bağlantıları kullanabileceğini ve hangilerinin SSL şifreli bağlantılar (`hostssl`) gerektirdiğini belirlemeleri için kullanılır. Ayrıca, istemciler sunuculara yalnızca SSL aracılığıyla bağlanacaklarını belirtebilirler.

GSSAPI şifreli bağlantılar, sorgular ve döndürülen veriler dahil olmak üzere ağ üzerinden gönderilen tüm verileri şifreler. (ağ üzerinden parola gönderilmez.) `pg_hba.conf` dosyası, yöneticilerin hangi ana bilgisayarların ( host ) şifrelenmemiş bağlantıları kullanabileceğini ve hangilerinin GSSAPI ile şifrelenmiş bağlantılar (`hostgssenc`) gerektirdiğini belirlemeleri için kullanılır. Ayrıca, istemciler sunuculara yalnızca GSSAPI şifreli bağlantılarda bağlanacaklarını belirtebilirler (`gssencmode = required`).

Stunnel veya SSH, akışı şifrelemek için de kullanılabilir.

`SSL Host Authentication`

Hem istemcinin hem de sunucunun birbirine SSL sertifikaları sağlaması mümkündür. Her iki tarafta da fazladan yapılandırma gerektirmesine rağmen parola kullanımından daha güçlü kimlik doğrulaması sağlar. Bir bilgisayarın, istemci tarafından gönderilen parolayı okuyacak kadar uzun süre sunucu gibi davranmasını engeller. Ayrıca, istemci ile sunucu arasındaki bir bilgisayarın sunucu gibi davrandığı ve istemci ile sunucu arasındaki tüm verileri okuyup ilettiği "ortadaki adam" ( man in the middle ) saldırılarının önlenmesine yardımcı olur.

`İstemci Tarafı Şifreleme` ( Client-Side Encryption )

Sistem yöneticisine güvenilmiyorsa, istemcinin verileri şifrelemesi gerekir. Böylece şifrelenmemiş veriler asla veritabanı sunucusunda görünmez. Veriler, sunucuya gönderilmeden önce istemcide şifrelenir ve veritabanı sonuçlarının kullanılmadan önce istemcide şifresinin çözülmesi gerekir.

{% include links.html %}
