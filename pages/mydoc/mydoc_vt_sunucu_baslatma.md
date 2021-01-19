---
title: "Veritabanı Sunucusunu Başlatma"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 21, 2020
sidebar: mydoc_sidebar
permalink: mydoc_vt_sunucu_baslatma.html
folder: mydoc
---

## Veritabanı Sunucusunu Başlatma

Veritabanına herhangi birinin erişebilmesi için veritabanı sunucusunu başlatmanız gerekir. Veritabanı sunucusu programına `postgres` denir. PostgreSQL'in önceden paketlenmiş bir sürümünü kullanıyorsanız, işletim sisteminizin kurallarına göre sunucuyu bir arka plan görevi olarak çalıştırmaya yönelik akış hemen hemen her dağıtım için sağlanmıştır.

Sunucuyu manuel olarak başlatmanın basit yolu, postgres'i `-D` seçeneği ile veri dizininin konumunu belirterek çağırmaktır, örnek:

```bash
$ postgres -D /usr/local/pgsql/data
```

{% include callout.html content="Bu komut PostgreSQL kullanıcı hesabında çalıştırılır ve sunucuyu ön planda çalışır duruma getirir. Sunucu, -D olmadığı durumda PGDATA ortam değişkeni kullanmaya çalışır. Bu değişken de sağlanmazsa işlem başarısız olur." type="info" %}

postgres'i arka planda başlamak için Unix kabuk sözdizimi:

```sql
$ postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```

{% include callout.html content="Sunucunun `stdout` ve `stderr` çıktısını yukarıda gösterildiği gibi bir yerde saklamak denetim ve sorun teşhisinde yardımcı olacaktır." type="info" %}

`pg_ctl` sarmalayıcı programı, bu kabuk sözdizimi karmaşasından kurtulmak için kullanılabilir, Örnek:

```sql
pg_ctl start -l logfile
```

{% include callout.html content="Bu işlem sunucuyu arka planda başlatacak ve çıktıyı adlandırılmış günlük dosyasına yazacaktır. -D seçeneği burada `postgres` komutundaki kullanımla aynı anlama sahiptir. Ayrıca pg_ctl ile sunucu durdurulabilir." type="info" %}

Farklı sistemlerin arka plan yordamlarını önyükleme sırasında başlatmak için farklı kuralları vardır. Çoğu sistemde `/etc/rc.local` veya `/etc/rc.d/rc.local` dosyası bulunur. Diğerleri `init.d` veya `rc.d` dizinlerini kullanır. Sunucu süper kullanıcı PostgreSQL kullanıcı hesabı tarafından çalıştırılmalıdır. Bu yüzden komutlarınızı `su postgres -c '...'` formunda oluşturmalısınız. Örneğin:

```sql
su postgres -c 'pg_ctl start -D /usr/local/pgsql/data -l serverlog'
```

Linux sistemlere özgü birkaç öneri.

Systemd kullanırken, aşağıdaki hizmet birimi dosyasını kullanabilirsiniz (`/etc/systemd/system/postgresql.service` adresinde):

```json
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify
User=postgres
ExecStart=/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

Burada zaman aşımı ayarını dikkatlice düşünün. systemd 90 saniyelik varsayılan zaman aşımına sahiptir ve bu süre içinde hazır olduğunu bildirmeyen işlem sonlandırılacaktır. 0 değeri zaman aşımını devre dışı bırakır.

Sunucu çalışırken, PID'si veri dizinindeki `postmaster.pid` dosyasında saklanır. Bu, birden çok instance'ın aynı veri dizininde çalışmasını önlemek ve sunucuyu kapatmak için kullanılabilir.

### Sunucu Başlatma Hataları

Sunucuyu başlangıçlarının başarısız olmasının birkaç genel nedeni vardır. Bu durumlarda, sunucu loglarını kontrol edin veya elle başlatarak (standart çıktıyı veya standart hatayı yönlendirmeden) hangi hata mesajlarının alındığına bakın. Yaygın hata mesajlarından bazıları;

```text
LOG:  could not bind IPv4 address "127.0.0.1": Address already in use
HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

Bu, çalışmakta olan sunucuyla aynı portta başka bir sunucu başlatmayı denedinizi bildiren bir öneri mesajıdır. Bu çekirdek hata mesajı *Address already in use* veya bir çeşidi değilse farklı bir sorun olabilir. Örneğin, ayırılmış bir portta sunucu başlatmaya çalışmak aşağıdaki gibi bir hata verecektir:

```bash
$ postgres -p 666
LOG:  could not bind IPv4 address "127.0.0.1": Permission denied
HINT:  Is another postmaster already running on port 666? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

```bash
FATAL:  could not create shared memory segment: Invalid argument
DETAIL:  Failed system call was shmget(key=5440001, size=4011376640, 03600).
```

Bu mesaj, muhtemelen çekirdeğinizin paylaşılan bellek boyutu sınırının PostgreSQL'in oluşturmaya çalıştığı çalışma alanından daha küçük olduğu anlamına gelir (örnekte 4011376640 bayt).

Şuna benzer bir hata:

```bash
FATAL:  could not create semaphores: No space left on device
DETAIL:  Failed system call was semget(5440126, 17, 03600).
```

Bu hata, disk alanınızın tükendiği anlamına gelmez. Bu, çekirdeğinizin System V semaforlarının sayısı üzerindeki sınırının PostgreSQL'in oluşturmak istediği sayıdan daha küçük olduğu ifade eder.

### İstemci Bağlantı Sorunları

İstemci tarafında olası hata koşulları oldukça çeşitli ve uygulamaya bağlı olsa da bunlardan birkaçı doğrudan sunucunun nasıl başlatıldığıyla ilgilidir. Aşağıda gösterilenler dışındaki koşullar, ilgili istemci uygulamasıyla belgelenmelidir.

```bash
psql: could not connect to server: Connection refused
        Is the server running on host "server.joe.com" and accepting
        TCP/IP connections on port 5432?
```

Bu genel "konuşacak bir sunucu bulamadım" hatasıdır. TCP / IP bağlantısı denendiğinde yukarıdaki hata alınır. Burada yapılan en yaygın hata sunucuyu TCP / IP bağlantılarına izin verecek şekilde yapılandırmayı unutmaktır.

Benzer şekilde yerel bir sunucuya Unix soket bağlantısı denediğinizde şöyle bir hata alırsınız:

```bash
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

İstemcinin doğru yere bağlanmaya çalıştığını doğrulamak son satır kullanışlıdır. Gerçekte burada çalışan sunucu yoksa, gösterildiği gibi "Connection refused" veya "No such file or directory" şeklinde bir çekirdek hata mesajı alınır. Bir diğer hata mesajı olan "Connection timed out" ise ağ bağlantısının olmaması gibi daha temel sorunları ifade eder.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/server-start.html)

{% include links.html %}
