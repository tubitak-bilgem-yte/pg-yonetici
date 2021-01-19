---
title: "SSL ile Güvenli TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres, ssl, tcp
last_updated: December 23, 2020
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_ssl.html
folder: mydoc
---

## SSL ile Güvenli TCP / IP Bağlantıları

PostgreSQL, artırılmış güvenlik gereksinimleriniz için istemci / sunucu iletişimini şifrelemek üzere SSL bağlantı desteğine sahiptir. Bu özelliğin kullanılabilir olması için OpenSSL'nin hem istemci hem de sunucu sistemlerinde kurulu ve derleme sırasında bu özelliğin etkinleştirilmiş olması gerekir.

PostgreSQL sunucusu, *postgresql.conf* dosyasından `ssl` parametresi `on` olarak ayarlanarak SSL etkinleştirilmiş halde başlatılabilir. Sunucu aynı TCP bağlantı noktasından hem SSL bağlantılarını hem normal bağlantıları dinler. Sunucu, SSL kullanıp kullanmama konusunda bağlanan istemciyle görüşür. SSL kullanımı, bağlantıların bir kısmı veya tamamı için gerektirecek şekilde ayarlanabilir.

SSL modunda başlamak için, sunucu sertifikası ve private key'i içeren dosyalar mevcut olmalıdır. Bu dosyaların, sunucunun veri dizininde sırasıyla `server.crt` ve `server.key` olarak adlandırılır. Başka isim ve konumlar için [`ssl_cert_file`](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-SSL-CERT-FILE) ve [`ssl_key_file`](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-SSL-KEY-FILE) yapılandırma parametreleri kullanılır.

Unix sistemlerinde `server.key` üzerindeki izinler dışarıya veya gruba herhangi bir erişime izin vermemelidir. Bunu `chmod 0600 server.key` komutuyla sağlayın.

### OpenSSL Yapılandırması

PostgreSQL, sistem geneli OpenSSL yapılandırma dosyası `openssl.cnf`'ı kullanır. Bu dosya, `openssl version -d` tarafından bildirilen dizinde bulunur.

OpenSSL, çeşitli cipher ve kimlik doğrulama algoritmalarını destekler. OpenSSL yapılandırma dosyasında bir cipher listesi belirtirken, *postgresql.conf* içindeki [ssl_ciphers](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-SSL-CIPHERS)'ı değiştirerek veritabanı sunucusu tarafından kullanılmak üzere spesifik olarak cipher belirtebilirsiniz.

### İstemci Sertifikalarını Kullanma

İstemcinin güvenilir bir sertifika sağlamasını zorunlu kılmak için, güvendiğiniz kök sertifika otoritelerinin sertifikalarını veri dizinindeki bir dosyaya yerleştirin, *postgresql.conf*'daki `ssl_ca_file` parametresini yeni dosya adına ayarlayın ve *pg_hba.conf* içindeki ilgili `hostssl` satırlarına `clientcert=verify-ca` veya `clientcert=verify-full` kimlik doğrulama seçeneğini ekleyin. Artık SSL bağlantısı geldiğinde istemciden bir sertifika istenecektir.

`clientcert=verify-ca` ile istemci sertifikasının güvenilir sertifika otoriteleri tarafından imzalandığını doğrular. `clientcert=verify-full` ayarında, sunucu yalnızca sertifika zincirini doğrulamakla kalmaz, aynı zamanda kullanıcı adının veya eşlemesinin sağlanan sertifika `cn` (Common Name) ile eşleşip eşleşmediğini de kontrol eder. `clientcert` kimlik doğrulama seçeneği `hostssl` olarak belirtilen *pg_hba.conf* satırlarında tüm kimlik doğrulama yöntemleri için kullanılabilir.

### SSL Sunucu Dosyası Kullanımı

Aşağıdaki tablo, sunucudaki SSL kurulumuyla ilgili dosyaları özetlemektedir.

| Dosya | İçerik | Aksiyon |
|-------|--------|-------|--------|-------|--------|
| `ssl_cert_file ($PGDATA/server.crt)` | sunucu sertifikası | Sunucunun kimliğini belirtmek için istemciye gönderilir |
| `ssl_key_file ($PGDATA/server.key)` | sunucu özel anahtarı ( private key ) | Sunucu sertifikasının sahibi tarafından gönderildiğini kanıtlar, sertifika sahibinin güvenilir olduğunu göstermez |
| `ssl_ca_file` | güvenilir sertifika otoriteleri | İstemci sertifikasının güvenilir bir sertifika otoritesi tarafından imzalanıp imzalanmadığını kontrol eder |
| `ssl_crl_file` | sertifika yetkilileri tarafından iptal edilen sertifikalar | İstemci sertifikası bu listede olmamalıdır |

Sunucu, bu dosyaları sunucu başlangıcında ve sunucu yapılandırması yeniden yüklendiğinde okur. Sunucu başlangıcınca, bu dosyalarda bir hata tespit edilirse sunucu başlatılmaz. Fakat, bir yapılandırma yeniden yüklemesi sırasında hata algılanırsa, dosyalar yok sayılır ve eski SSL yapılandırması kullanılmaya devam eder. Tüm bu olaylar sunucu günlüğünde raporlanır.

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

{% include note.html content=" Kendinden imzalı bir sertifika test amaçlı kullanılabilir. Production'da bir sertifika otoritesi (CA) tarafından imzalanmış sertifika kullanılmalıdır. "%}

sertifika imzalama isteği ( CSR ) ve public / private anahtar dosyalarını oluşturun:

```bash
openssl req -new -nodes -text -out root.csr \
  -keyout root.key -subj "/CN=root.yourdomain.com"
chmod og-rwx root.key
```

Ardından, kök sertifika otoritesi oluşturmak için isteği anahtarla imzalayın:

```bash
openssl x509 -req -in root.csr -text -days 3650 \
  -extfile /etc/ssl/openssl.cnf -extensions v3_ca \
  -signkey root.key -out root.crt
```

Son olarak, yeni root sertifika otoritesi tarafından imzalanmış bir sunucu sertifikası oluşturun:

```bash
openssl req -new -nodes -text -out server.csr \
  -keyout server.key -subj "/CN=dbhost.yourdomain.com"
chmod og-rwx server.key

openssl x509 -req -in server.csr -text -days 365 \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out server.crt
```

{% include callout.html content=" `server.crt` ve `server.key` sunucuda, `root.crt` ise istemcide depolanmalıdır. `root.key` sonraki sertifikaların oluşturulmasında kullanılmak üzere depolanmalıdır." type="info" %}

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

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/ssl-tcp.html)

{% include links.html %}
