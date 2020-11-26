---
title: "SSL ile Güvenli TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 25, 2020
summary: "SSL ile Güvenli TCP / IP Bağlantıları"
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_ssl.html
folder: mydoc
---

## SSL ile Güvenli TCP / IP Bağlantıları

PostgreSQL, artırılmış güvenlik gereksinimleriniz için istemci / sunucu iletişimini şifrelemek üzere yerel SSL bağlantı desteğine sahiptir. Bu özelliğin kullanılabilir olması için OpenSSL'nin hem istemci hem de sunucu sistemlerinde kurulu ve derleme sırasında bu özelliğin etkinleştirilmiş olması gerekir.

### Temel Kurulum

SSL desteği derlenmiş haldeyken, PostgreSQL sunucusu `postgresql.conf` dosyasından `ssl` parametresi `on` olarak ayarlanarak SSL etkinleştirilmiş halde başlatılabilir. Sunucu aynı TCP bağlantı noktasından hem SSL bağlantılarını hem normal bağlantıları dinler. Sunucu SSL kullanıp kullanmama konusunda bağlanan istemciyle görüşür. SSL kullanımı, bağlantıların bir kısmı veya tamamı için gerektirecek şekilde ayarlanması gibi konularda ayrıntılı bilgiye [pg_hba.conf Dosyası]("") bölümünden ulaşabilirsiniz.

SSL modunda başlamak için, sunucu sertifikasını ve özel anahtarı ( private key ) içeren dosyalar mevcut olmalıdır. Bu dosyaların, sunucunun veri dizininde sırasıyla `server.crt` ve `server.key` olarak adlandırılması beklenir. Başka isim ve konumlar için [`ssl_cert_file`](https://www.postgresql.org/docs/13/runtime-config-connection.html#GUC-SSL-CERT-FILE) ve [`ssl_key_file`](https://www.postgresql.org/docs/13/runtime-config-connection.html#GUC-SSL-KEY-FILE) yapılandırma parametreleri kullanılır.

Unix sistemlerinde `server.key` üzerindeki izinler dışarıya veya gruba herhangi bir erişime izin vermemelidir. Bunu `chmod 0600 server.key` komutuyla sağlayın. Alternatif olarak dosya sahibi root olabilir ve grup okuma erişimine (`0640` izinleri) sahip olabilir. Bu kurulum, sertifika ve anahtar dosyalarının işletim sistemi tarafından yönetildiği yüklemelere yöneliktir. PostgreSQL sunucusunun altında çalıştığı kullanıcı, sertifika ve anahtar dosyalarına erişimi olan grubun üyesi yapılmalıdır.

Veri dizini grup okuma erişimine izin veriyorsa, yukarıda belirtilen güvenlik gereksinimlerine uymak için sertifika dosyalarının veri dizininin dışına taşınması gerekebilir. Genellikle ayrıcalıklı olmayan bir kullanıcısının veritabanını yedeklemesine izin vermek için grup erişimi etkinleştirilir. Bu durumda yedekleme yazılımı sertifika dosyalarını okuyamayacak ve büyük olasılıkla hata verecektir.

Özel anahtar ( private key) bir passphrase ile korunuyorsa, sunucu passphrase sorar ve girilene kadar başlamaz. passphrase'i varsayılan olarak kullanmak, sunucuyu yeniden başlatmadan SSL yapılandırması değiştirme yeteneğini devre dışı bırakır, ancak bkz. [`ssl_passphrase_command_supports_reload`](https://www.postgresql.org/docs/13/runtime-config-connection.html#GUC-SSL-PASSPHRASE-COMMAND-SUPPORTS-RELOAD). Ayrıca, passphrase korumalı özel anahtarlar Windows'ta kullanılamaz.

`server.crt`'deki ilk sertifika sunucunun sertifikası olmalıdır çünkü sunucunun özel anahtarıyla eşleşmelidir. "ara ( intermediate )" sertifika otoritelerinin sertifikaları da dosyaya eklenebilir. Böylece, istemcilerde ara sertifikaları depolanma gerekliliği ortadan kalkar. Bunu yaparken tabi kök ve ara sertifikaların `v3_ca` (sertifikanın temel `CA` kısıtlamasını `true` olarak ayarlar.) uzantılarıyla oluşturulduğunu varsayıyoruz. Bu, ara sertifikaların süresinin daha kolay dolmasını sağlar.

Kök sertifikayı `server.crt`'ye eklemek gerekli değildir. Bunun yerine, istemciler sunucunun sertifika zincirinin kök sertifikasına sahip olmalıdır.

### OpenSSL Yapılandırması

PostgreSQL, sistem geneli OpenSSL yapılandırma dosyasını okur. Bu dosya `openssl.cnf` olarak adlandırılır ve `openssl version -d` tarafından bildirilen dizinde bulunur. Bu varsayılan, `OPENSSL_CONF` ortam değişkenini istenen yapılandırma dosyası adına ayarlayarak geçersiz kılınabilir.

OpenSSL, pekçok stratejiye sahip chiper ve kimlik doğrulama algoritmalarını destekler. OpenSSL yapılandırma dosyasında bir chiper listesi belirtirken, `postgresql.conf` içindeki [ssl_ciphers](https://www.postgresql.org/docs/13/runtime-config-connection.html#GUC-SSL-CIPHERS)'ı değiştirerek veritabanı sunucusu tarafından kullanılmak üzere spesifik olarak chiper belirtebilirsiniz.

{% include note.html content=" NULL-SHA veya NULL-MD5 chiper'ları kullanarak şifreleme ek yükü olmadan kimlik doğrulaması yapmak mümkündür. Ancak, istemci ile sunucu arasındaki iletişim aradaki biri tarafından okuyabilir ve iletebilir. Ayrıca, kimlik doğrulama ek yüküne kıyasla şifreleme ek yükü minimumdur. Bu nedenlerden dolayı NULL cipher'lar tavsiye edilmez."%}

### İstemci Sertifikalarını Kullanma

İstemcinin güvenilir bir sertifika sağlamasını zorunlu kılmak için, güvendiğiniz kök sertifika otoritelerinin sertifikalarını veri dizinindeki bir dosyaya yerleştirin, postgresql.conf'daki `ssl_ca_file` parametresini yeni dosya adına ayarlayın ve `pg_hba.conf` içindeki ilgili `hostssl` satırlarına `clientcert=verify-ca` veya `clientcert=verify-full` kimlik doğrulama seçeneğini ekleyin. Artık SSL bağlantısı geldiğinde istemciden bir sertifika istenecektir.

`clientcert=verify-ca` içeren bir `hostssl` girdisi için sunucu, istemcinin sertifikasının güvenilir sertifika otoriteleri tarafından imzalandığını doğrular. `clientcert=verify-full` belirtilirse, sunucu yalnızca sertifika zincirini doğrulamakla kalmaz, aynı zamanda kullanıcı adının veya eşlemesinin sağlanan sertifika `cn` (Common Name) ile eşleşip eşleşmediğini de kontrol eder. Sertifika zinciri doğrulamasının `cert` kimlik doğrulama yöntemi kullanıldığında sağlandığını unutmayın (bkz. [Sertifika Doğrulama](https://www.postgresql.org/docs/13/auth-cert.html)).

`clientcert` kimlik doğrulama seçeneği `hostssl` olarak belirtilen `pg_hba.conf` satırlarında tüm kimlik doğrulama yöntemleri için kullanılabilir.

Kullanıcıların oturum açma sırasında bir sertifika sağlamasını zorunlu kılmak için iki yaklaşım vardır.

İlki, `pg_hba.conf`'daki `hostssl` girdileri için `cert` kimlik doğrulama yöntemini kullanır. Sertifikanın kendisi kimlik doğrulama için kullanılırken aynı zamanda ssl bağlantı güvenliği de sağlar. Ayrıntılar için [Sertifika Doğrulama](https://www.postgresql.org/docs/13/auth-cert.html) bölümüne bakın. (`cert` kimlik doğrulama yöntemini kullanırken herhangi bir `clientcert` seçeneğinin açıkça belirtilmesi gerekli değildir.) Bu durumda, sertifikada sağlanan cn (Common Name) kullanıcı adı veya uygun eşleme ile kontrol edilir.

İkinci yaklaşım, `clientcert` kimlik doğrulama seçeneğini `verify-ca` veya `verify-full` olarak ayarlanan `hostssl` girdileri için kimlik doğrulama yöntemini istemci sertifikalarının doğrulaması ile birleştirir. İlk seçenek yalnızca sertifikanın geçerli olmasını zorunlu kılarken, ikincisi ayrıca sertifikadaki cn (Common Name)'in kullanıcı adıyla veya uygun bir eşlemeyle eşleşmesini sağlar.

### SSL Sunucu Dosyası Kullanımı

Aşağıdaki tablo sunucudaki SSL kurulumuyla ilgili dosyaları özetlemektedir. (Verilen dosya adları varsayılan adlardır. Yerel olarak yapılandırılmış adlar farklı olabilir.)

| Dosya | İçerik | Aksiyon |
|-------|--------|-------|--------|-------|--------|
| `ssl_cert_file ($PGDATA/server.crt)` | sunucu sertifikası | Sunucunun kimliğini belirtmek için istemciye gönderilir |
| `ssl_key_file ($PGDATA/server.key)` | sunucu özel anahtarı ( private key ) | Sunucu sertifikasının sahibi tarafından gönderildiğini kanıtlar, sertifika sahibinin güvenilir olduğunu göstermez |
| `ssl_ca_file` | güvenilir sertifika otoriteleri | İstemci sertifikasının güvenilir bir sertifika otoritesi tarafından imzalanıp imzalanmadığını kontrol eder |
| `ssl_crl_file` | sertifika yetkilileri tarafından iptal edilen sertifikalar | İstemci sertifikası bu listede olmamalıdır |

Sunucu, bu dosyaları sunucu başlangıcında ve sunucu yapılandırması yeniden yüklendiğinde okur. Windows sistemlerde yeni istemci bağlantısı için yeni bir arka uç süreci ( backend process ) oluştuğunda tekrar okunurlar.

Sunucu başlatılırken bu dosyalarda bir hata tespit edilirse sunucu başlatılmaz. Fakat, bir yapılandırma yeniden yüklemesi sırasında hata algılanırsa, dosyalar yok sayılır ve eski SSL yapılandırması kullanılmaya devam eder. Windows sistemlerinde, arka uç ( backend ) başlangıcında bu dosyalarda bir hata tespit edilirse ilgili arka uç ( backend ) bir SSL bağlantısı kuramayacaktır. Tüm bu olaylar, hata durumu sunucu günlüğünde rapor edilir.

### Sertifikaların Oluşturulması

Sunucu için 365 gün geçerli kendinden imzalı bir sertifika oluşturmak için aşağıdaki OpenSSL komutunu kullanın. `dbhost.yourdomain.com` kısmını sunucunun ( host ) ismiyle değiştirin:

```bash
openssl req -new -x509 -days 365 -nodes -text -out server.crt \
  -keyout server.key -subj "/CN=dbhost.yourdomain.com"
```

Sonra şunu uygulayın;

```bash
chmod og-rwx server.key
```

çünkü sunucu izinleri bundan daha fazla ise dosyayı reddedecektir. Özel anahtar ( private key ) ve sertifika oluşturma ilgili daha fazla ayrıntı için OpenSSL belgelerine bakın.

Kendinden imzalı bir sertifika test amaçlı kullanılabilirken, production'da bir sertifika otoritesi (CA) tarafından imzalanmış sertifika kullanılmalıdır.

Kimliği istemciler tarafından doğrulanabilen sunucu sertifikası oluşturmak için, öncelikle sertifika imzalama isteği (certificate signing request - CSR) ve genel ( public) / özel ( private ) anahtar dosyalarını oluşturun:

```bash
openssl req -new -nodes -text -out root.csr \
  -keyout root.key -subj "/CN=root.yourdomain.com"
chmod og-rwx root.key
```

Ardından, kök sertifika otoritesi oluşturmak için isteği anahtarla imzalayın (Linux'ta varsayılan OpenSSL yapılandırma dosyası konumunu kullanarak):

```bash
openssl x509 -req -in root.csr -text -days 3650 \
  -extfile /etc/ssl/openssl.cnf -extensions v3_ca \
  -signkey root.key -out root.crt
```

Son olarak, yeni kök sertifika otoritesi tarafından imzalanmış bir sunucu sertifikası oluşturun:

```bash
openssl req -new -nodes -text -out server.csr \
  -keyout server.key -subj "/CN=dbhost.yourdomain.com"
chmod og-rwx server.key

openssl x509 -req -in server.csr -text -days 365 \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out server.crt
```

`server.crt` ve `server.key` sunucuda depolanmalıdır. `root.crt` ise istemcinin, sunucunun alt (leaf) sertifikasının güvenilir kök sertifikası tarafından imzalandığını doğrulayabilmesi için istemcide depolanmalıdır. `root.key` gelecekteki sertifikaların oluşturulmasında kullanılmak üzere çevrimdışı depolanmalıdır.

Ara sertifikaları içeren bir güven zinciri oluşturmak da mümkündür:

```bash
# root
openssl req -new -nodes -text -out root.csr \
  -keyout root.key -subj "/CN=root.yourdomain.com"
chmod og-rwx root.key
openssl x509 -req -in root.csr -text -days 3650 \
  -extfile /etc/ssl/openssl.cnf -extensions v3_ca \
  -signkey root.key -out root.crt

# intermediate
openssl req -new -nodes -text -out intermediate.csr \
  -keyout intermediate.key -subj "/CN=intermediate.yourdomain.com"
chmod og-rwx intermediate.key
openssl x509 -req -in intermediate.csr -text -days 1825 \
  -extfile /etc/ssl/openssl.cnf -extensions v3_ca \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out intermediate.crt

# leaf
openssl req -new -nodes -text -out server.csr \
  -keyout server.key -subj "/CN=dbhost.yourdomain.com"
chmod og-rwx server.key
openssl x509 -req -in server.csr -text -days 365 \
  -CA intermediate.crt -CAkey intermediate.key -CAcreateserial \
  -out server.crt
```

{% include callout.html content="`server.crt` ve `intermediate.crt`, bir sertifika dosyası paketinde birleştirilmeli ve sunucuda depolanmalıdır. `server.key` sunucuda saklanmalıdır. `root.crt` istemcinin sunucunun leaf sertifikasının güvenilir kök sertifikasına bağlı bir sertifika zinciri tarafından imzalandığını doğrulayabilmesi için istemcide depolanmalıdır. `root.key` ve `intermediate.key` sonraki sertifikaların oluşturulmasında kullanılmak üzere çevrimdışı depolanmalıdır." type="info" %}

{% include links.html %}
