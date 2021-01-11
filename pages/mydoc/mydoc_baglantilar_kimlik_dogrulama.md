---
title: "Bağlantılar ve Kimlik Doğrulama"
tags: [PostgreSQL]
keywords: postgres,listen_addresses,max_connections,superuser_reserved_connections,unix_socket_group,ssl_key_file,tcp_keepalives_interval,tcp_keepalives_idle,unix_socket_directories,tcp_user_timeout,authentication_timeout,password_encryption,krb_server_keyfile,krb_server_keyfile
last_updated: December 31, 2020
sidebar: mydoc_sidebar
permalink: mydoc_baglantilar_kimlik_dogrulama.html
folder: mydoc
---

## Bağlantılar ve Kimlik Doğrulama

### Bağlantı Ayarları

{% include callout.html content=" **`listen_addresses (string)`** : İstemci bağlantılarının dinlendiği TCP / IP adres veya adreslerini belirtir. Virgülle ayrılmış host ismi listesi ve / veya IP adres değerlerini alır. `*`, mevcut tüm IP arayüzlerini ifade eder. `0.0.0.0` değeri tüm IPv4 adreslerini, `::` ise tüm IPv6 adreslerini dinlemeye izin verir. Liste boşsa herhangi bir IP arayüzü dinlemez. Bu durumda sunucuya bağlanmak için sadece Unix-domain soketleri kullanılabilir. Varsayılan değer, yalnızca yerel TCP / IP 'loopback' bağlantılarının yapılmasına izin veren localhost'tur. İstemci kimlik doğrulaması mekanizması, sunucuya erişimler konusunda kontroler sağlar. *listen_addresses*, bağlantı girişimlerinin hangi arabirimlerce kabul edileceğini kontrol eder, güvenli olmayan ağ arayüzlerinde tekrarlanan kötü niyetli bağlantı isteklerini önler. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`port (integer)`** : Sunucunun dinlediği TCP portudur. Varsayılan 5432'dir. Sunucu dinlediği her IP adresleri için aynı port numarasını kullanır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`max_connections (integer)`** : Veritabanı sunucusuna maksimum eşzamanlı bağlantı sayısını belirtir. Varsayılan 100 bağlantıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include important.html content=" Bir standby sunucu kurulumunda bu parametre primary sunucudakiyle aynı veya daha yüksek bir değere ayarlanmalıdır. Aksi takdirde, standby sunucusunda sorgulara izin verilmeyecektir." %}

{% include callout.html content=" **`superuser_reserved_connections (integer)`** : Süper kullanıcı bağlantıları için ayrılan slot sayısını belirtir. Aynı anda en fazla `max_connections` kadar bağlantı aktif olabilir. Bir sunucusunda **`max_connections` - `superuser_reserved_connections`** kadar etkin eşzamanlı normal olabilir. Normal aktif bağlantı sayısına ulaşıldığında yeni bağlantılar yalnızca süper kullanıcılar için kabul edilir. Yeni replikasyon bağlantıları kabul edilmez. <br/><br/>
Varsayılan 3 bağlantıdır. Ayarlanacak değer *max_connections* değerinden küçük olmalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`unix_socket_directories (string)`** : Sunucunun istemci uygulamalarından gelen bağlantıları dinleyeceği Unix-domain soket veya soketlerin dizinini belirtir. Birden fazla soket, dizinleri virgülle ayırıp listeleyerek oluşturulabilir. Boş değer, herhangi bir Unix-domain soketinin dinlenmeyeceğini belirtir. Bu durumda sunucu bağlantıları için yalnızca TCP / IP soketleri kullanılır. Varsayılan değer `/tmp`'dir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`unix_socket_group (string)`** : Unix-domain soketlerinin sahiplik grubunu ayarlar. Soketlerin sahibi her zaman sunucuyu başlatan kullanıcıdır. `unix_socket_permissions` parametresiyle birlikte Unix-domain bağlantıları için ek bir erişim kontrol mekanizması olarak kullanılabilir. Öntanımlı değeri, sunucu kullanıcısının grubunu kullanan boş string'tir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`unix_socket_permissions (integer):`** : Unix-domain soketlerinin erişim izinlerini ayarlar. Unix-domain soketleri klasik Unix dosya sistemi izin setini kullanır. Parametre değerleri `chmod` ve `umask` sistem çağrıları tarafından kabul edilen formatta verilir. Varsayılan izinler herkesin bağlanabilir olduğu `0777`'dir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`bonjour (boolean)`**: Bonjour, sunucunun varlığını duyurmayı sağlar. Öntanımlı olarak kapalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`bonjour_name (string)`** : Bonjour servis ismini belirtir. Bu parametre boş string `''` olarak ayarlanırsa (varsayılan böyle) bilgisayar ismi kullanılır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content=" **`tcp_keepalives_idle (integer)`** : İşletim sisteminin istemciye bir TCP keepalive mesajı göndermesi gereken ağ etkinliği olmadan geçen süreyi belirtir. Bu değer birimsiz verilirse saniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_KEEPIDLE` veya eşdeğer bir soket seçeneğini destekleyen sistemlerde ve Windows'ta kullanılabilir. Diğer sistemlerde sıfır olmalıdır. Unix-domain soketi üzerinden bağlanan oturumlarda bu parametre önemsenmez ve sıfır olarak okunur." type="primary" %}

{% include note.html content=" Windows, sistem varsayılan değerlerini okumak için bir yol sağlamadığından 0 değeri vermek bu parametreyi 2 saate ayarlar." %}

{% include callout.html content=" **`tcp_keepalives_interval (integer)`** : İstemci tarafından alındığı bildirilmemiş bir TCP keepalive mesajının ne kadar süre sonra tekrar gönderileceğini belirtir. Bu değer birimsiz belirtilirse saniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_KEEPINTVL` veya eşdeğer bir soket seçeneğini destekleyen sistemlerde ve Windows'ta kullanılabilir. Diğer sistemlerde bu parametre 0 olmalıdır. Unix-domain soketi aracılığıyla bağlanan oturumlarda bu parametre önemsenmez ve sıfır olarak okunur." type="primary" %}

{% include note.html content="Windows, sistem varsayılan değerlerini okumak için bir yol sağlamadığından 0 değeri vermek bu parametreyi 1 saniyeye ayarlar." %}

{% include callout.html content=" **`tcp_keepalives_count (integer)`**: Sunucu-istemci bağlantısı kesildi olarak kabul edilmeden önce kaybolabilecek TCP keepalive mesajlarının sayısını belirtir. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_KEEPCNT`'yi veya eşdeğer bir soket seçeneğini destekleyen sistemlerde kullanılabilir. Diğer sistemlerde bu parametre 0 olmalıdır. Unix-domain soketi ile bağlanan oturumlarda bu parametre önemsenmez ve 0 olarak okunur." type="primary" %}

{% include note.html content=" Bu parametre Windows'ta desteklenmez ve sıfır olmalıdır." %}

{% include callout.html content=" **`tcp_user_timeout (integer)`**: TCP bağlantısı zorla kapatılmadan önce iletilen verilerin onaylanmadan kalabileceği süreyi belirtir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_USER_TIMEOUT`'u destekleyen sistemlerde kullanılabilir. Diğer sistemlerde bu parametre sıfır olmalıdır. Unix-domain soketi ile bağlanan oturumlarda bu parametre önemsenmez ve sıfır olarak okunur." type="primary" %}

{% include note.html content=" Bu parametre Windows'ta desteklenmez, sıfır olmalıdır." %}

### Doğrulama

{% include callout.html content=" **`authentication_timeout (integer)`**: İstemci kimlik doğrulamasının tamamlanması için verilen maksimum süreyi belirtir. İstemci, kimlik doğrulama protokolünü bu süre içinde tamamlamazsa sunucu bağlantıyı kapatır. Asılı istemcilerin bağlantıyı süresiz olarak işgal etmesi bu şekilde önlenir. Birimsiz belirtilmesi durumunda saniye olarak alınır. Öntanımlı değer 1 dakikadır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`password_encryption (enum)`**: Bu parametre, `CREATE ROLE` veya `ALTER ROLE` ile bir parola verildiğinde, parolayı şifrelemek için kullanılacak algoritmayı belirtir. Öntanılı değeri, parolayı bir MD5 hash olarak depolayan `md5`'tir. Bu parametre `scram-sha-256` olarak ayarlanırsa parola SCRAM-SHA-256 ile şifrelenir." type="primary" %}

{% include callout.html content=" **`krb_server_keyfile (string)`**: Kerberos sunucu anahtar dosyasının konumunu ayarlar. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`krb_caseins_users (boolean)`**: GSSAPI kullanıcı adlarının büyük / küçük harfe duyarlı olarak değerlendirilip değerlendirilmeyeceğini ayarlar. Öntanımlı değeri `off` yani büyük / küçük harfe duyarlıdır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`db_user_namespace (boolean)`**: Bu parametre her veritabanı için kullanıcı adlarını etkinleştirir. Varsayılan olarak kapalıdır, sadece *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

### SSL

SSL kurulumu hakkında bilgi için bkz. [SSL ile Güvenli TCP / IP Bağlantıları](mydoc_tcp_ip_ssl.html).

{% include callout.html content=" **`ssl (boolean)`**: SSL bağlantılarını etkinleştirir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri `off`'dur." type="primary" %}

{% include callout.html content=" **`ssl_ca_file (string)`**: SSL sunucu sertifika oteritesini (CA) içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri boştur, hiçbir CA dosyasın yüklenmediği anlamına gelir ve istemci sertifikası doğrulaması yapılmaz." type="primary" %}

{% include callout.html content=" **`ssl_cert_file (string)`**: SSL sunucu sertifikasını içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan *server.crt*'dir." type="primary" %}

{% include callout.html content=" **`ssl_crl_file (string)`**: SSL sunucu sertifikası iptal listesini (CRL) içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan boştur, hiçbir CRL dosyası yüklenmez." type="primary" %}

{% include callout.html content=" **`ssl_key_file (string)`**: SSL sunucusu private key'ini içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan *server.key*'dir ." type="primary" %}

{% include callout.html content=" **`ssl_ciphers (string)`**: SSL bağlantılarında kullanımına izin verilen SSL şifre paketlerinin (SSL cipher suites) listesini belirtir. Bu ayarın sözdizimi ve desteklenen değerlerin listesi için OpenSSL paketindeki ciphers kılavuzu sayfasına bakın. Yalnızca TLS sürüm 1.2 ve altını kullanan bağlantılar etkilenir. TLS sürüm 1.3 bağlantıları tarafından kullanılan cipher seçeneklerini şu an kontrol eden bir ayar yoktur. Varsayılan değer `HIGH:MEDIUM:+3DES:!aNULL` şeklindedir. Varsayılan, özel güvenlik gereksinimleriniz olmadığı sürece makul bir seçimdir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırındam ayarlanabilir." type="primary" %}

{% include callout.html content=" **`ssl_prefer_server_ciphers (boolean)`**: İstemci yerine, sunucunun SSL cipher tercihlerini kullanılıp kullanılmayacağını belirtir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan değeri`on`'dur." type="primary" %}

{% include callout.html content=" **`ssl_ecdh_curve (string)`**: ECDH anahtar değişiminde kullanılacak eğrinin adını belirtir. Bağlanan tüm istemciler tarafından desteklenmesi gereklidir. Sunucunun Eliptik Eğri (Elliptic Curve) anahtarı tarafından kullanılan eğriyle aynı olması gerekmez. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan değeri `prime256v1`'dir." type="primary" %}

{% include callout.html content=" **`ssl_min_protocol_version (enum)`**: Kullanılacak minimum SSL / TLS protokol sürümünü ayarlar. Geçerli değerler şu anda: TLSv1, TLSv1.1, TLSv1.2, TLSv1.3'dir. Varsayılan, bu yazı itibariyle sektörün ihtiyaçlarını karşılayan TLSv1.2'dir." type="primary" %}

{% include callout.html content=" **`ssl_max_protocol_version (enum)`**: Kullanılacak maksimum SSL / TLS protokol sürümünü ayarlar. Geçerli değerleri, `ssl_min_protocol_version` değerleri ve buna ek boş string'tir (her protokol sürümüne izin vermesi için). Varsayılan her sürüme izin verir. Maksimum protokol sürümünün ayarlanması bazı bileşenlerin daha yeni bir protokolle çalışırken sorunlar yaşadığında veya test için kullanılabilir." type="primary" %}

{% include callout.html content=" **`ssl_dh_params_file (string)`**: Geçici DH ailesi olarak adlandırılan SSL cipher'lar için kullanılan Diffie-Hellman parametrelerini içeren dosyanın adını belirtir. Varsayılan boştur, bu durumda derlenmiş varsayılan DH parametreleri kullanılır. Openssl `dhparam -out dhparams.pem 2048` komutuyla kendi DH parametreleri dosyanızı oluşturabilirsiniz. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`ssl_passphrase_command (string)`**: Private key gibi bir SSL dosyasının şifresini çözmek için passphrase alınması gerektiğinde çağrılacak harici komutu ayarlar. Bu parametre varsayılan boştur, yerleşik prompting mekanizması kullanılır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content=" **`ssl_passphrase_command_supports_reload (boolean)`**: Bu parametre, `ssl_passphrase_command` tarafından ayarlanan passphrase komutunun, yapılandırma yeniden yüklemesi sırasında anahtar dosyası bir passphrase gerektirdiğinde çağrılıp çağrılmayacağını belirler. Parametre kapalıysa (varsayılan böyledir) `ssl_passphrase_command`, yeniden yükleme sırasında yok sayılacak ve passphrase gerekirse SSL yapılandırması yeniden yüklenmeyecektir. passphrase'in bir dosyadan elde edilebildiği kullanımda bu parametrenin açık olarak ayarlanması uygun olabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/runtime-config-connection.html)

{% include links.html %}
