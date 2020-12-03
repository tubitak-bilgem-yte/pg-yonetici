---
title: "Bağlantılar ve Kimlik Doğrulama"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 27, 2020
summary: "Bağlantılar ve Kimlik Doğrulama"
sidebar: mydoc_sidebar
permalink: mydoc_baglantilar_kimlik_dogrulama.html
folder: mydoc
---

## Bağlantılar ve Kimlik Doğrulama

### Bağlantı Ayarları

{% include callout.html content="`listen_addresses (string)`: Sunucunun istemcilerden gelen bağlantıları dinleyeceği TCP / IP adresini veya adreslerini belirtir. Virgülle ayrılmış host ismi listesi ve / veya IP adresleri biçiminde değer alır. `*`, mevcut tüm IP arayüzlerini ifade eder. `0.0.0.0` değeri tüm IPv4 adreslerini, `::` ise tüm IPv6 adreslerini dinlemeye izin verir. Liste boşsa sunucu herhangi bir IP arayüzünü dinlemez. Bu durumda sunucuya bağlanmak için sadece Unix-domain soketleri kullanılabilir. Varsayılan değer, yalnızca yerel TCP / IP 'loopback' bağlantılarının yapılmasına izin veren localhost'tur. [İstemci kimlik doğrulaması](''), sunucuya kimlerin erişebileceği konusunda kontroler sağlar; *listen_addresses*, bağlantı girişimlerinin hangi arabirimlerce kabul edileceğini kontrol ederek güvenli olmayan ağ arayüzlerinde tekrarlanan kötü niyetli bağlantı isteklerini önler. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`port (integer)`: Sunucunun dinlediği TCP portudur. Varsayılan 5432'dir. Sunucu dinlediği tüm IP adresleri için aynı port numarasını kullanır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`max_connections (integer)`: Veritabanı sunucusuna maksimum eşzamanlı bağlantı sayısını belirler. Varsayılan 100 bağlantıdır ama çekirdek ayarlarınız desteklemiyorsa daha az olabilir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include important.html content="Bir standby sunucu kurulumunda bu parametreyi ana sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde, standby sunucusunda sorgulara izin verilmeyecektir." %}

{% include callout.html content="`superuser_reserved_connections (integer)`: PostgreSQL süper kullanıcı bağlantıları için ayrılan slot sayısını belirler. Aynı anda en fazla `max_connections` sayısı kadar bağlantı aktif olabilir. Bir veritabanı sunucusunda normal bağlantılar için **`max_connections` - `superuser_reserved_connections`** kadar etkin eşzamanlı olabilir. Normal aktif bağlantı sayısına ulaşıldığında yeni bağlantılar yalnızca süper kullanıcılar için kabul edilir, yeni replication bağlantıları kabul edilmez. <br/><br/>Varsayılan 3 bağlantıdır. Ayarlanacak değer *max_connections* değerinden küçük olmalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`unix_socket_directories (string)`: Sunucunun istemci uygulamalarından gelen bağlantıları dinleyeceği Unix-domain soket veya soketlerin dizinini belirtir. Birden fazla soket, dizinleri virgülle ayırıp listeleyerek oluşturulabilir. Girdiler arasındaki boşluklar yok sayılır. Boşluk veya virgül eklemeniz gerekiyorsa dizin adının çift tırnak içine alınması gerekir. Boş değer, herhangi bir Unix-domain soketinin dinlenmeyeceğini belirtir. Bu durumda sunucu bağlantıları için yalnızca TCP / IP soketleri kullanılabilir. Varsayılan değer `/tmp` olmakla birlikte derleme sırasında değiştirilebilir. Windows'ta varsayılan boştur, herhangi bir Unix-domain soketi oluşturulmaz. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. <br/><br/> **nnnn** port numarasına sahip '*.s.PGSQL.**nnnn***' olarak isimlendirilen soket dosyasına ek, *unix_socket_directories* dizinlerinin her birinde '*.s.PGSQL.**nnnn**.lock*' isimli bir dosya oluşturulur. Bu dosyalar elle silinmemelidir." type="primary" %}

{% include callout.html content="`unix_socket_group (string)`: Unix-domain soketlerinin sahiplik grubunu ayarlar. Soketlerin sahibi her zaman sunucuyu başlatan kullanıcıdır. `unix_socket_permissions` parametresiyle birlikte Unix-domain bağlantıları için ek bir erişim kontrol mekanizması olarak kullanılabilir. Öntanımlı değeri, sunucu kullanıcısının grubunu kullanan boş string'tir. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. <br/><br/> Bu parametre Windows'ta desteklenmediğinden Windows sistemlerde yapılacak ayar dikkate alınmayacaktır." type="primary" %}

{% include callout.html content="`unix_socket_permissions (integer):`: Unix-domain soketlerinin erişim izinlerini ayarlar. Unix-domain soketleri klasik Unix dosya sistemi izin setini kullanır. Parametre değerinin `chmod` ve `umask` sistem çağrıları tarafından kabul edilen formatta belirtilen sayısal modda olması gerekir. Alışılmış octal format kullanmak için sayı 0 (sıfır) ile başlamalıdır. <br/><br/> Varsayılan izinler herkesin bağlanabilir olduğu `0777`'dir. Makul alternatifler `0770` (yalnızca kullanıcı ve grup, bkz. unix_socket_group) ve `0700` (yalnızca kullanıcı). Unix-domain soketi için sadece yazma izninin önemli olduğunu, okuma veya yürütme izinlerini vermenin veya almanın bir anlamı olmadığını unutmayın. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`bonjour (boolean)`: Bonjour, sunucunun varlığını duyurmayı sağlar. Öntanımlı olarak kapalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`bonjour_name (string)`: Bonjour servis ismini belirtir. Bu parametre boş string `''` olarak ayarlanırsa (varsayılan böyle) bilgisayar ismi kullanılır. Sunucu Bonjour desteği ile derlenmemişse bu parametre önemsenmez. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir." type="primary" %}

{% include callout.html content="`tcp_keepalives_idle (integer)`: İşletim sisteminin istemciye bir TCP keepalive mesajı göndermesi gereken ağ etkinliği olmadan geçen süreyi belirtir. Bu değer birimsiz verilirse saniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını seçer. Bu parametre yalnızca `TCP_KEEPIDLE` veya eşdeğer bir soket seçeneğini destekleyen sistemlerde ve Windows'ta kullanılabilir. Diğer sistemlerde sıfır olmalıdır. Unix-domain soketi üzerinden bağlanan oturumlarda bu parametre önemsenmez ve her zaman sıfır olarak okunur." type="primary" %}

{% include note.html content="Windows, sistem varsayılan değerlerini okumak için bir yol sağlamadığından 0 değeri vermek bu parametreyi 2 saate ayarlar." %}

{% include callout.html content="`tcp_keepalives_interval (integer)`: İstemci tarafından alındığı bildirilmemiş bir TCP keepalive mesajının ne kadar süre sonra tekrar gönderileceğini belirtir. Bu değer birimsiz belirtilirse saniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_KEEPINTVL` veya eşdeğer bir soket seçeneğini destekleyen sistemlerde ve Windows'ta kullanılabilir. Diğer sistemlerde bu parametre 0 olmalıdır. Unix-domain soketi aracılığıyla bağlanan oturumlarda bu parametre önemsenmez ve her zaman sıfır olarak okunur." type="primary" %}

{% include note.html content="Windows, sistem varsayılan değerlerini okumak için bir yol sağlamadığından 0 değeri vermek bu parametreyi 1 saniyeye ayarlar." %}

{% include callout.html content="`tcp_keepalives_count (integer)`: Sunucunun istemciyle bağlantısı kesildi olarak kabul edilmeden önce kaybolabilecek TCP keepalive mesajlarının sayısını belirtir. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_KEEPCNT`'yi veya eşdeğer bir soket seçeneğini destekleyen sistemlerde kullanılabilir. Diğer sistemlerde bu parametre 0 olmalıdır. Unix-domain soketi aracılığıyla bağlanan oturumlarda bu parametre önemsenmez ve her zaman 0 olarak okunur." type="primary" %}

{% include note.html content="Bu parametre Windows'ta desteklenmez ve sıfır olmalıdır." %}

{% include callout.html content="`tcp_user_timeout (integer)`: TCP bağlantısı zorla kapatılmadan önce iletilen verilerin onaylanmadan kalabileceği süreyi belirtir. Bu değer birimsiz belirtilirse milisaniye olarak alınır. 0 değeri (öntanımlı) işletim sisteminin varsayılanını kullanır. Bu parametre yalnızca `TCP_USER_TIMEOUT`'u destekleyen sistemlerde kullanılabilir. Diğer sistemlerde bu parametre sıfır olmalıdır. Unix-domain soketi aracılığıyla bağlanan oturumlarda bu parametre önemsenmez ve her zaman sıfır olarak okunur." type="primary" %}

{% include note.html content="Bu parametre Windows'ta desteklenmez ve sıfır olmalıdır." %}

### Doğrulama

{% include callout.html content="`authentication_timeout (integer)`: İstemci kimlik doğrulamasının tamamlanması için verilen maksimum süre. İstemci, kimlik doğrulama protokolünü bu süre içinde tamamlamazsa sunucu bağlantıyı kapatır. Bu şekilde, asılı istemcilerin bağlantıyı süresiz olarak işgal etmesi önlenir. Birimsiz belirtilmesi durumunda saniye olarak alınır. Öntanımlı değer 1 dakikadır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`password_encryption (enum)`: Bu parametre, `CREATE ROLE` veya `ALTER ROLE` ile bir parola verildiğinde, parolayı şifrelemek için kullanılacak algoritmayı belirtir. Öntanılı değeri, parolayı bir MD5 hash olarak depolayan `md5`'tir. Bu parametre `scram-sha-256` olarak ayarlanırsa parola SCRAM-SHA-256 ile şifrelenir. <br/><br/> Eski istemcilerde SCRAM kimlik doğrulama mekanizmasını desteklemeyebilir, bu nedenle SCRAM-SHA-256 ile şifrelenmiş parolalarla çalışmayabilir. Daha fazla ayrıntı için [Şifre Doğrulaması](https://www.postgresql.org/docs/current/auth-password.html) bölümüne bakın." type="primary" %}

{% include callout.html content="`krb_server_keyfile (string)`: Kerberos sunucu anahtar dosyasının konumunu ayarlar. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`krb_caseins_users (boolean)`: GSSAPI kullanıcı adlarının büyük / küçük harfe duyarlı olarak değerlendirilip değerlendirilmeyeceğini ayarlar. Öntanımlı değeri `off` yani büyük / küçük harfe duyarlıdır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`db_user_namespace (boolean)`: Bu parametre her veritabanı için kullanıcı adlarını etkinleştirir. Varsayılan olarak kapalıdır, sadece *postgresql.conf* dosyasından veya sunucu komut satırından ayarlanabilir. <br/><br/>

Parametre açıksa kullanıcıları **`username@dbname`** şeklinde oluşturmalısınız. **`username`** istemci tarafından gönderildiğinde, **`@`** ve veritabanı adı gelen kullanıcı adına eklenir ve elde edilen veritabanı spesifik kullanıcı adı, sunucuda aranır. SQL ortamında '@' içeren kullanıcılar oluşturduğunuzda, kullanıcı adını tek tırnak içene almanız gerektiğini unutmayın.<br/><br/>

Bu parametre etkinleştirildiğinde normal global kullanıcıları hala oluşturabilirsiniz. İstemcide kullanıcı adını verirken '@' eklemeniz yeterlidir, örn `yte@`. Kullanıcı adı sunucuda aranmadan önce '@' kaldırılacaktır.<br/><br/> 

*`db_user_namespace`*, istemci ve sunucu kullanıcı adı temsillerinin farklı olmasına neden olur. Kimlik doğrulama kontrolleri her zaman sunucunun kullanıcı adıyla yapılır. Bu nedenle kimlik doğrulama yöntemleri yapılandırılırken istemcinin değil sunucunun kullanıcı adı için yapılmalıdır. `md5`, kullanıcı adını hem istemcide hem de sunucuda salt olarak kullandığı için md5 *db_user_namespace* ile kullanılamaz." type="primary" %}

{% include note.html content="Bu özellik, eksiksiz bir çözüm bulunana kadar geçici bir önlem olarak tasarlanmıştır." %}

### SSL

SSL kurulumu hakkında bilgi için [SSL ile Güvenli TCP / IP Bağlantıları](mydoc_tcp_ip_ssl.html) sayfasına bakın.

{% include callout.html content="`ssl (boolean)`: SSL bağlantılarını etkinleştirir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri `off`." type="primary" %}

{% include callout.html content="`ssl_ca_file (string)`: SSL sunucu sertifika oteritesini (CA) içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Öntanımlı değeri boştur, hiçbir CA dosyasın yüklenmediği anlamına gelir ve istemci sertifikası doğrulaması yapılmaz." type="primary" %}

{% include callout.html content="`ssl_cert_file (string)`: SSL sunucu sertifikasını içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. *server.crt* varsayılandır." type="primary" %}

{% include callout.html content="`ssl_crl_file (string)`: SSL sunucu sertifikası iptal listesini (CRL) içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan boştur, hiçbir CRL dosyası yüklenmez." type="primary" %}

{% include callout.html content="`ssl_key_file (string)`: SSL sunucusu private key'ini içeren dosyanın adını belirtir. Relative paths veri diziniyle ilişkilidir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. *server.key* varsayılandır." type="primary" %}

{% include callout.html content="`ssl_ciphers (string)`: SSL bağlantılarında kullanımına izin verilen SSL şifre paketlerinin (SSL cipher suites) listesini belirtir. Bu ayarın sözdizimi ve desteklenen değerlerin listesi için OpenSSL paketindeki ciphers kılavuzu sayfasına bakın. Yalnızca TLS sürüm 1.2 ve altını kullanan bağlantılar etkilenir. TLS sürüm 1.3 bağlantıları tarafından kullanılan cipher seçeneklerini şu anda kontrol eden bir ayar yoktur. Varsayılan değer `HIGH:MEDIUM:+3DES:!aNULL` şeklindedir. Varsayılan, özel güvenlik gereksinimleriniz olmadığı sürece makul bir seçimdir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırındam ayarlanabilir.<br/><br/> 
Varsayılan değerin açıklaması:<br/><br/>
**HIGH**: HIGH grubundan cipher'ları kullanan şifre paketleri (ör. AES, Camellia, 3DES)<br/><br/> 
**MEDIUM**: MEDIUM grubundan cipher'ları kullanan şifre paketleri (ör. RC4, SEED)<br/><br/> 
**+3DES**: HIGH için OpenSSL varsayılan sıralaması problemlidir, burada 3DES, AES128'den daha yüksek sıradadır. Bu hatalıdır, çünkü 3DES, AES128'den daha az güvenlik sunuyor ve daha yavaştır. +3DES, diğer tüm HIGH ve MEDIUM cipher'lardan sonra sıralar.<br/><br/> 
**!aNULL**: Kimlik doğrulaması yapmayan anonim şifre paketlerini devre dışı bırakır. Bu tür şifre paketleri, man-in-the-middle saldırılarına karşı savunmasız olduğu için kullanılmamalıdır.<br/><br/> 
Mevcut şifre paketi ayrıntıları OpenSSL sürümleri arasında değişiklik gösterir. Kurulu OpenSSL sürümünün ayrıntılarını görmek için `openssl ciphers -v 'HIGH: MEDIUM: + 3DES:! ANULL'` komutunu kullanın. Bu liste sunucu anahtar türüne göre çalışma zamanında filtrelenmiştir." type="primary" %}

{% include callout.html content="`ssl_prefer_server_ciphers (boolean)`: İstemci yerine, sunucunun SSL cipher tercihlerini kullanılıp kullanılmayacağını belirtir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan `on`.<br/><br/>

Eski PostgreSQL sürümlerinde bu ayar bulunmaz, her zaman istemcinin tercihlerini kullanır. Bu ayar, esas olarak bu sürümlerle geriye dönük uyumluluk içindir. Sunucunun uygun şekilde yapılandırılmış olma olasılığı daha yüksek olduğu için sunucu tercihlerini kullanmak genellikle daha iyidir." type="primary" %}

{% include callout.html content="`ssl_ecdh_curve (string)`: ECDH anahtar değişiminde kullanılacak eğrinin adını belirtir. Bağlanan tüm istemciler tarafından desteklenmesi gereklidir. Sunucunun Eliptik Eğri (Elliptic Curve) anahtarı tarafından kullanılan eğriyle aynı olması gerekmez. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir. Varsayılan `prime256v1`'dir.<br/><br/>

En yaygın eğriler için OpenSSL adları şunlardır: `prime256v1` (NIST P-256), `secp384r1` (NIST P-384), `secp521r1` (NIST P-521). Kullanılabilir eğrilerin tam listesi `openssl ecparam -list_curves` komutuyla görülebilir. Yine de hepsi TLS'de kullanılamaz." type="primary" %}

{% include callout.html content="`ssl_min_protocol_version (enum)`: Kullanılacak minimum SSL / TLS protokol sürümünü ayarlar. Geçerli değerler şu anda: TLSv1, TLSv1.1, TLSv1.2, TLSv1.3. OpenSSL kütüphanesinin eski sürümleri tüm değerleri desteklemez; desteklenmeyen bir ayar seçilirse hata verilir. TLS 1.0 önceki protokol sürümlerini yani SSL sürüm 2 ve 3'ü devre dışı bırakır. Varsayılan, bu yazı itibariyle sektörün ihtiyaçlarını karşılayan TLSv1.2'dir." type="primary" %}

{% include callout.html content="`ssl_max_protocol_version (enum)`: Kullanılacak maksimum SSL / TLS protokol sürümünü ayarlar. Geçerli değerleri, `ssl_min_protocol_version` değerleri ve buna ek boş bir string'tir (her protokol sürümüne izin vermesi için). Varsayılan her sürüme izin verir. Maksimum protokol sürümünün ayarlanması bazı bileşenlerin daha yeni bir protokolle çalışırken sorunlar yaşadığında veya test için kullanılabilir." type="primary" %}

{% include callout.html content="`ssl_dh_params_file (string)`: Geçici DH ailesi olarak adlandırılan SSL cipher'lar için kullanılan Diffie-Hellman parametrelerini içeren dosyanın adını belirtir. Varsayılan boştur, bu durumda derlenmiş varsayılan DH parametreleri kullanılır. Bir saldırganın iyi bilinen derlenmiş DH parametrelerini kırmayı başarması durumunda, özel DH parametrelerinin kullanılmış olunması maruz kalmayı azaltır. Openssl `dhparam -out dhparams.pem 2048` komutuyla kendi DH parametreleri dosyanızı oluşturabilirsiniz. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`ssl_passphrase_command (string)`: Private key gibi bir SSL dosyasının şifresini çözmek için passphrase alınması gerektiğinde çağrılacak harici komutu ayarlar. Bu parametre varsayılan boştur, yerleşik prompting mekanizması kullanılır.<br/><br/>

Komut, standart çıktıya passphrase yazdırmalı ve kod 0 ile çıkmalıdır. Parametre değerinde`%p`, bir prompt string'i ile değiştirilir. prompt string'inin boşluk içerecebileceğinden uygun şekilde tırnak içine aldığınızdan emin olun. Varsa çıktının sonundanki te satırsonu ifadesi atılır.<br/><br/>

Komutun kullanıcıdan bir passphrase istemesi zorunlu değildir. Bir dosyadan okuyabilir, bir keychain'den veya benzeri bir yerden edinebilir. Seçilen mekanizmanın yeterince güvenli olduğundan emin olmak kullanıcıya kalmıştır. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include callout.html content="`ssl_passphrase_command_supports_reload (boolean)`: Bu parametre, `ssl_passphrase_command` tarafından ayarlanan passphrase komutunun, yapılandırma yeniden yüklemesi sırasında anahtar dosyası bir passphrase gerektirdiğinde çağrılıp çağrılmayacağını belirler. Parametre kapalıysa (varsayılan böyledir) `ssl_passphrase_command` yeniden yükleme sırasında yok sayılacak ve passphrase gerekirse SSL yapılandırması yeniden yüklenmeyecektir. passphrase'in bir dosyadan elde edilebildiği bir örnekte bu parametrenin açık olarak ayarlanması uygun olabilir. Bu parametre yalnızca *postgresql.conf* dosyasından ve sunucu komut satırından ayarlanabilir." type="primary" %}

{% include links.html %}
