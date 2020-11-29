---
title: "Veritabanı Sunucusunu Başlatma"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 23, 2020
summary: "Veritabanı Sunucusunu Başlatma"
sidebar: mydoc_sidebar
permalink: mydoc_vt_sunucu_baslatma.html
folder: mydoc
---

## Veritabanı Sunucusunu Başlatma

Veritabanına herhangi birinin erişebilmesi için veritabanı sunucusunu başlatmanız gerekir. Veritabanı sunucusu programına `postgres` denir.

PostgreSQL'in önceden paketlenmiş bir sürümünü kullanıyorsanız, işletim sisteminizin kurallarına göre sunucuyu bir arka plan görevi olarak çalıştırmaya yönelik akış hemen hemen her dağıtım için sağlanmıştır. Sunucuyu başlatmak için paketin altyapısını kullanmak, kendizin nasıl yapacağınızı belirtmesinden çok daha kolay olacaktır. Ayrıntılar için paket düzeyindeki belgelere bakın.

Sunucuyu manuel olarak başlatmanın basit yolu, postgres'i doğrudan veri dizininin konumunu `-D` seçeneği ile belirterek çağırmaktır, örneğin:

```bash
$ postgres -D /usr/local/pgsql/data
```

{% include callout.html content="Bu komut PostgreSQL kullanıcı hesabında çalıştırılır ve sunucuyu ön planda çalışır duruma getirir. Sunucu, -D olmadığı durumda PGDATA ortam değişkeni tarafından adlandırılan veri dizinini kullanmaya çalışır. Bu değişken de sağlanmazsa işlem başarısız olur." type="info" %}

postgres'i arka planda başlamak için Unix kabuk sözdizimini kullanın:

```sql
$ postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```

{% include callout.html content="Sunucunun `stdout` ve `stderr` çıktısını yukarıda gösterildiği gibi bir yerde saklamak denetim amaçlarınıza ve sorunları teşhis etmeye yardımcı olacaktır." type="info" %}

Bu kabuk sözdizimi karmaşasından kurtulmak için bazı görevleri basitleştirmek için sağlanmış olan `pg_ctl` sarmalayıcı programı kullanılır, Örneğin:

```sql
pg_ctl start -l logfile
```

{% include callout.html content="Bu işlem sunucuyu arka planda başlatacak ve çıktıyı adlandırılmış günlük dosyasına yazacaktır. -D seçeneği burada `postgres` komutundaki kullanımla aynı anlama sahiptir. pg_ctl ile ayrıca sunucu durdurulabilir. " type="info" %}

Normalde, bilgisayar önyüklendiğinde veritabanı sunucusunu başlatmak isteyeceksiniz. Otomatik başlatma komut dosyaları işletim sistemine özeldir. `contrib/start-scripts` dizininde PostgreSQL ile dağıtılan örnek komut dosyalarını root ayrıcalıkları kullanarak kurmak mümkündür.

Farklı sistemlerin arka plan yordamlarını önyükleme sırasında başlatmak için farklı kuralları vardır. Çoğu sistemde `/etc/rc.local` veya `/etc/rc.d/rc.local` dosyası bulunur. Diğerleri `init.d` veya `rc.d` dizinlerini kullanır. Sunucu süper kullanıcı veya başka bir kullanıcı tarafından değil, PostgreSQL kullanıcı hesabı tarafından çalıştırılmalıdır. Bu yüzden komutlarınızı `su postgres -c '...'` formunda oluşturmalısınız. Örneğin:

```sql
su postgres -c 'pg_ctl start -D /usr/local/pgsql/data -l serverlog'
```

İşletim sistemine özgü birkaç öneri. (Genel değerleri gösterdiğimiz her durum için doğru kurulum dizinini ve kullanıcı adını kullandığınızdan emin olun.)

- FreeBSD için, PostgreSQL kaynak dağıtımındaki `contrib/start-scripts/freebsd` dosyasına bakın.
- OpenBSD'de `/etc/rc.local` dosyasına aşağıdaki satırları ekleyin:

```bash
if [ -x /usr/local/pgsql/bin/pg_ctl -a -x /usr/local/pgsql/bin/postgres ]; then
    su -l postgres -c '/usr/local/pgsql/bin/pg_ctl start -s -l /var/postgresql/log -D /usr/local/pgsql/data'
    echo -n ' postgresql'
fi
```

- Linux sistemlerinde

```bash
/usr/local/pgsql/bin/pg_ctl start -l logfile -D /usr/local/pgsql/data
```

`/etc/rc.d/rc.local` veya `/etc/rc.local`'a veya PostgreSQL kaynak dağıtımındaki `contrib/start-scripts/linux` dosyasına bakın.

Systemd kullanırken, aşağıdaki hizmet birimi dosyasını kullanabilirsiniz (ör. `/etc/systemd/system/postgresql.service` adresinde):

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

Burada zaman aşımı ayarını dikkatlice düşünün. systemd 90 saniyelik bir varsayılan zaman aşımına sahiptir ve bu süre içinde hazır olduğunu bildirmeyen bir işlemi sonlandıracaktır. Başlangıçta çökme kurtarma ( crash recovery ) işlemi gerçekleştirmesi gerekebilecek bir PostgreSQL sunucusunun hazır hale gelmesi çok daha uzun süreceğinden önerilen 0 değeri zaman aşımı mantığını devre dışı bırakır.

- NetBSD'de, tercihe bağlı olarak FreeBSD veya Linux başlatma komut dosyalarını kullanın.
- Solaris'te, aşağıdaki satırı içeren `/etc/init.d/postgresql` adlı bir dosya oluşturun:

```bash
su - postgres -c "/usr/local/pgsql/bin/pg_ctl start -l logfile -D /usr/local/pgsql/data"
```

Sonra `/etc/rc3.d` dosyasında `S99postgresql` olarak sembolik bir bağlantı oluşturun.

Sunucu çalışırken, PID'si veri dizinindeki `postmaster.pid` dosyasında saklanır. Bu, birden çok sunucu örneğinin ( instance ) aynı veri dizininde çalışmasını önlemek ve sunucuyu kapatmak için kullanılabilir.

### Sunucu Başlatma Hataları

Sunucuyu başlatırken başarısız olmanın birkaç genel nedeni vardır. Bu durumlarda sunucunun günlük dosyasını kontrol edin veya elle başlatın (standart çıktıyı veya standart hatayı yeniden yönlendirmeden) ve hangi hata mesajlarının alındığına bakın. Aşağıda en yaygın hata mesajlarından bazılarını ayrıntılı olarak açıklıyoruz.

```text
LOG:  could not bind IPv4 address "127.0.0.1": Address already in use
HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

Bu, zaten çalışmakta olan aynı bağlantı noktasında başka bir sunucu başlatmayı denedinizi bildiren bir öneri mejaşıdır. Burada çekirdek hata mesajı *Address already in use* veya bir çeşidi değilse farklı bir sorun olabilir. Örneğin, kullanılan bir bağlantı noktasında sunucu başlatmaya çalışmak aşağıdaki gibi bir hata verecektir:

```bash
$ postgres -p 666
LOG:  could not bind IPv4 address "127.0.0.1": Permission denied
HINT:  Is another postmaster already running on port 666? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

Şöyle bir mesaj:

```bash
FATAL:  could not create shared memory segment: Invalid argument
DETAIL:  Failed system call was shmget(key=5440001, size=4011376640, 03600).
```

Muhtemelen çekirdeğinizin paylaşılan bellek boyutu sınırının PostgreSQL'in oluşturmaya çalıştığı çalışma alanından daha küçük olduğu anlamına gelir (bu örnekte 4011376640 bayt). Bu, yalnızca `shared_memory_type` öğesini `sysv` olarak ayarladıysanız gerçekleşebilir. Bu durumda, sunucuyu normalden daha az sayıda arabellekle ([shared_buffers]("")) başlatmayı deneyebilir veya izin verilen paylaşılan bellek boyutunu artırmak için çekirdeğinizi yeniden yapılandırabilirsiniz. Aynı makinede birden çok sunucuyu başlatmaya denediğinizde, talep edilen toplam alan çekirdek sınırını aşıyorsa da bu mesajı görebilirsiniz.

Şuna benzer bir hata:

```bash
FATAL:  could not create semaphores: No space left on device
DETAIL:  Failed system call was semget(5440126, 17, 03600).
```

Bu hata, disk alanınızın tükendiği anlamına gelmez. Bu, çekirdeğinizin System V semaforlarının sayısı üzerindeki sınırının PostgreSQL'in oluşturmak istediği sayıdan daha küçük olduğu ifade eder. Sunucuyu maksimum eşzamanlı bağlantı sayısını azaltarak başlatarak sorunu çözebilirsiniz ancak nihai çözüm olarak çekirdek sınırını arttırmak olacaktır.

System V IPC olanaklarının yapılandırılmasına ilişkin ayrıntılar [Çekirdek Kaynaklarını Yönetme]("") bölümünde verilmiştir.

### İstemci Bağlantı Sorunları

İstemci tarafında olası hata koşulları oldukça çeşitli ve uygulamaya bağlı olsa da bunlardan birkaçı doğrudan sunucunun nasıl başlatıldığıyla ilgilidir. Aşağıda gösterilenler dışındaki koşullar, ilgili istemci uygulamasıyla belgelenmelidir.

```bash
psql: could not connect to server: Connection refused
        Is the server running on host "server.joe.com" and accepting
        TCP/IP connections on port 5432?
```

Bu genel "konuşacak bir sunucu bulamadım" hatasıdır. TCP / IP bağlantısı denendiğinde yukarıdaki hata alınır. Burada yapılan en yaygın hata sunucuyu TCP / IP bağlantılarına izin verecek şekilde yapılandırmayı unutmaktır.

Benzer şekilde yerel bir sunucuya Unix-domain soket bağlantısı denediğinizde şöyle bir hata alırsınız:

```bash
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

Son satır, istemcinin doğru yere bağlanmaya çalıştığını doğrulamak için kullanışlıdır. Gerçekte orada çalışan sunucu yoksa, gösterildiği gibi "Connection refused" veya "No such file or directory" şeklinde bir çekirdek hata mesajı alınacaktır. Bir diğer hata mesajı olan "Connection timed out" ise ağ bağlantısının olmaması gibi daha temel sorunları ifade eder.

{% include links.html %}
