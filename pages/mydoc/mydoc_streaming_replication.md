---
title: "Streaming Replikasyon"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 31, 2020
sidebar: mydoc_sidebar
permalink: mydoc_streaming_replication.html
folder: mydoc
---

## PostgreSQL Replikasyon Giriş

Streaming Replication PostgreSQL'e 9.0 sürümü ile birlikte gelen ve PostgreSQL'in içine gömülü replikasyon çözümüdür. Hot Standby da yine 9.0 sürümü ile birlikte gelen ve standby sunucusunu salt okunur olarak çalıştırmaya izin veren bir teknolojidir. 9.1'de synchronous replication, 9.2'de cascading replication, 9.3'de timeline switch following, 9.5'de ise replication slots ve timed delay standbys özellikleri PostgreSQL'e eklenmiştir. Bu bölümde, Streaming Replication (kısaca SR), Hot Standby (kısaca HS), Cascading Replication (CR) ve Synchronous Replication (Sync Rep) yapılandırması ve ipuçları ele alınmıştır.

## Streaming Replikasyon Altyapısının Kurulması

### Streaming Replication ve Hot Standby Hazırlıkları

**Master sunucudaki postgresql.conf değişiklikleri**:

Aşağıdaki değişiklikler Streaming Replication ve Hot Standby için özel değişikliklerdir. Diğer değişiklikleri içermezler ve öntanımlı *postgresql.conf*'a göre olan değişikliklerdir. Önce master sunucudaki değişecek parametereleri yazalım.

*WAL arşivlemesi yöntemi:*

```text
wal_level = replica # 9.6 ve sonrası 
max_wal_senders = 10
wal_keep_segments = 256 
archive_mode=on 
archive_command=/usr/pgsql-9.6/bin/rsyncwal.sh %p %f' 
```

*replication slot yöntemi:*

```text
wal_level = logical 
max_wal_senders=5 
max_replication_slots=5
```

Şimdi bu parametreleri açıklayalım:

**`wal_level`**: PostgreSQL'de transaction loglarının adı bildiğiniz gibi Write Ahead Log (WAL)'dur. `wal_level` parametresi, WAL içine ne kadar bilgi yazılacağını belirtir. 9.6’da bu parametre `minimal`, `replica` ve `logical` değerlerini almaktadır. minimal, wal_level 'ın öntanımlı parametresidir, ve WAL içine PostgreSQL'in sadece normal şekilde işlemesi için gereken bilgilerin yazılmasını sağlar. Bu bilgiler PostgreSQL'in çökme anında kurtarması gereken bilgilerdir. replica ise PITR ya da WAL arşivlemesi için gereken bilgilerin yazılmasını sağlar ya da üstte tarif ettiğimiz HS sunucusunun çalışması için gereken ek bilgilerin de WAL içine yazılmasını sağlar. Son olarak logical ise logical replication içindir. Bu üçü arasında en çok bilgi logical içindedir. Dolayısıyla wal_level parametresinin gereksiz şekilde logical olarak ayarlanması, PostgreSQL'in daha fazla WAL üretmesi anlamına gelecektir. wal_level parametresi minimal olduğunda aşağıdaki işlemlerde WAL loglama güvenli olarak es geçilebilir,ve bu da bu işlemlerin başarımını arttırır:

```sql
CREATE TABLE AS 
CREATE INDEX 
CLUSTER 
COPY (aynı tx içinde yaratılan ya da truncate edilen tablolara) 
```

**`max_wal_senders`**: Standby sunucudan ya da sunuculardan eşzamanlı olarak yapılabilecek eşzamanlı replikasyon bağlantısının sayısını belirler. Bu sayıya base backup almak için yapılan bağlantılar da dahildir. kısacası, en fazla kaç walsender sürecinin başlayacağını belirler. Öntanımlı değeri 0'dır, replikasyon yapılmaz. Bu parametreyi 10 olarak ayarlayabiliriz. Bu parametreden önce üstteki wal_level'ın mutlaka replica olarak ayarlanması gerekmektedir.

**`wal_keep_segments`**: Standby sunucu tarafından kullanılması gereken, archiver süreci başarısız olduğu için checkpoint tarafından da temizlenmeyen ve pg_wal dizininde tutulacak olan WAL sayısını belirtir. Eğer standby sunucu bu parametredeki sayıdan daha fazla geride kalırsa, o zaman master sunucu standby tarafından hala gereksinim duyulan WAL dosyalarını silebilir. Bu durumda replikasyon duracaktır. Bu sayıyı çok arttırmanız pg_wal dizininizde çok sayıda dosyanın birikmesi anlamına gelecektir. Performans açısından sıkıntı yoktur; ancak bekleyen dosyaların birden gönderilmesi network üzerinde baskı oluşturabilir.

**`archive_mode`**: WAL arşivlemesini etkinleştirir. Restart gerektiren bir parametredir. archiver sürecini başlatır.

**`archive_command`**: Dolan ya da `archive_timeout` parametresi ile aktarma zamanı gelen WAL dosyalarının nereye nasıl gönderileceğini belirten parametredir. Burada 2 makro kullanılabilir:

- `%p`: Arşivlenecek WAL dosyasının tam yolu (dosya adı dahil)
- `%f`: Arşivlenecek dosyanın sadece adı

Bu komut mutlaka 0 döndürmelidir. Aksi taktirde komut başarısız olmuş sayılacaktır. Arşivleme başarısız olursa, işlem başarılı olana kadar archive_command tekrar tekrar çalıştırılacaktır. Burada bir script kullanabilir; ya da doğrudan *rsync/scp/cp* gibi bir komut da kullanabilirsiniz.

{% include note.html content="  Bu parametrenin içerdiği dizini/dizinleri standby sunucularda yaratmayı unutmayınız. Ayrıca, eğer 1'den fazla standby sunucunuz varsa, WALların her sunucuya ayrı ayrı atılması gerekir" %}

Örnek bir rsyncwal.sh scripti şu şekilde olabilir:

```bash
#!/bin/bash 
 
rsync -q -ae ssh $1 \ postgres@192.168.122.85:/pgsql/9.6walarchive/$2
 
if [ $? != 0 ] then 
    echo "Archiver error:" 
    exit 1 
fi
 
 
exit 0
```

Burada $1 ve $2, archive_command parametresinde verilen 2 değişkeni sırası ile gösterir. $1 %p, $2 de %f 'ye karşılık gelir. Bu dosyanın izinlerini düzenleyelim:

```bash
chown postgres: /usr/pgsql-9.6/bin/rsyncwal.sh 
chmod 700 /usr/pgsql-9.6/bin/rsyncwal.sh
```

**Master sunucuda replikasyon kullanıcısı yaratmak**:

Replikasyon için özel bir kullanıcı yaratacağız, ve hatta postgres kullanıcısının da bu yetkisini geri alacağız. Bu özellik PostgreSQL replikasyon güvenliğini arttırır:

```sql
CREATE ROLE replicauser WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'deneme'; 
ALTER ROLE postgres NOREPLICATION;
```

**Master sunucudaki pg_hba.conf değişiklikleri**:

Master sunucuda walreceiver sürecine izin vermeniz gerekmektedir. Bunun için standby sunucusunun ip'sini *pg_hba.conf* içine ekleyiniz. Burada özel olak “replication” sanal veritabanını kullanıyoruz. Aşağıda örnek bir satır bulabilirsiniz.

```text
host replication replicauser 192.168.1.10/32 md5
```

Bu işlemlerden sonra PostgreSQL'i restart etmeyi unutmayın.

**Replication slot yaratmak:**

Replikasyonu replication slot ile yaptığımızda, master sunucuda aşağıdaki komutu çalıştıralım. mgm_slot yerine istediğiniz adı verebilirsiniz. Bu adı sonraki aşamalarda pg_receivexlog ile kullanacağız:

```text
SELECT * FROM pg_create_physical_replication_slot('mgm_slot');
```

**Base backup almak**:

SR+HS'ı çalıştırmak için, sunucunun dosya sistemi seviyesinde yedeğini almak gerekir. Bunun için de SQL komutlarını ya da komut satırı araçlarını kullanacağız.

**Parolasız SSH yapmak (WAL arşivlemesi için)**:

WAL dosyalarının karşı sunucuya aktarılması için, sftp/rsync işlemlerinin parolasız olması gerekmektedir. Aksi takdirde her işlemde parola sorulur ve bu işlem etkileşimli olmadığı için WAL arşivlemesi yapılamaz. Aşağıdaki komutu iki sunucuda da, postgres kullanıcısı olarak çalıştırın ve sizden parola istendiğinde enter'a basıp devam edin:

```bash
ssh-keygen -t rsa 
```

Şimdi, ssh anahtarlarımızı saklayacağımız dosyayı yaratalım ve gerekli izinlerini verelim:

```text
touch ~/.ssh/authorized_keys 
chmod 600 ~/.ssh/authorized_keys
```

Ardından sunuculardaki *~/.ssh/id_rsa.pub* dosyasının içeriğini diğer sunucudaki *~/.ssh/authorized_keys* dosyasına tek satırda aynen aktarınız. Son aşamada, iki sunucuya da karşılıklı olarak *postgres* kullanıcılarından ssh yapınız. Hem host keyleri saklamış olur, hem de yaptıklarımızı denemiş olursunuz.

**Aşağıdakini master sunucu üzerinde yapınız**:

**1.** psql ile herhangi bir veritabanına bağlanın ve aşağıdakini çalıştırın (etiket yerine herhangi birşey yazabilirsiniz):

```sql
SELECT pg_start_backup('etiket');
```

**2.** Veri dosyalarını rsync ile standby sunucusuna atın. Öncelikle ssh anahtarı ile yetkilendirmeyi etkinleştirmelisiniz. Bu işlemi postgres kullanıcısı ile yapmalısınız. Örnek komut:

```bash
rsync --delete -ave ssh /var/lib/pgsql/9.6/data/* \ 
192.168.1.10:/var/lib/pgsql/9.6/data
```

rsync bittiğinde, psql ile aşağıdaki komutu çalıştırın:

```sql
SELECT pg_stop_backup();
```

**pg_basebackup ile Base Backup Almak (önerilen yöntem, WAL arşivlemesi yöntemi)**:

PostgreSQL ile gelen pg_basebackup komutu, bize base backup almakta yardımcı olur. Bu komutu standby sunucuda çalıştırmalısınız:

```bash
/usr/pgsql-9.6/bin/pg_basebackup -D /var/lib/pgsql/9.6/standby -c fast -x -P -Fp -R -h 192.168.1.22 -p 5432 -U replicauser
```

**pg_basebackup ile Base Backup Almak (önerilen yöntem, replication slot yöntemi)**:

Replication slot kullandığımızda, komutta küçük bir değişiklik yapacağız:

```bash
/usr/pgsql-9.6/bin/pg_basebackup -D /var/lib/pgsql/9.6/standby -c fast -X stream -P -Fp -R -h 127.0.0.1 -p 5432 -U replicauser
```

Parametreleri açıklayalım:

**`-c`**: PostgreSQL, base backup almadan önce master sunucuda checkpoint yapmak ister. Öntanımlı olarak da bu işlemi “spreaded” yapar (ayrıntılar için kitabımızın checkpoint ile ilgili kısmına bakabilirsiniz). Eğer vakit kaybetmek istemiyorsanız ve sunucunuza gelecek ek yükü gözardı ediyorsanız, pg_basebackup'ın checkpoint'i “fast” modda yapmasını sağlayabilirsiniz.

**`-x`**: base backup içinde WAL dosyalarının eklenmesini sağlar. (WAL arşivlemesi yöntemi)

**`-P`**: “Progress” anlamına gelir, base backup alma sürecini yüzdeli gösterir.

**`-F`**: Base backup “formatını” belirler. Öntanımlı olarak pg_basebackup komutu yedeği sıkıştırarak (tar ile) alır. Yedek amaçlı base backuplarda bunu tercih edebilirsiniz. Ancak, replikasyon hazılıklarında “plain” formatta almanız daha uygun olacaktır. -Fp, bunu sağlar.

**`-h, -p , -U`**: master sunucunun ip adresi/alan adı, portu ve replikasyon için kullanıcı adını buraya yazabilirsiniz. Bu komutu büyük veritabanlarında screen içinde çalıştırabilirsiniz.

**Standby sunucudaki pg_receivexlog başlatmak (replication slot yöntemi)**:

Standby sunucusunda replication slot kullanmak için pg_receivexlog'u başlatalım:

```sql
mkdir -m 700 /var/lib/pgsql/9.6/xlogsb
 
/usr/pgsql-9.6/bin/pg_receivexlog -D /var/lib/pgsql/9.6/xlogsb/ -S mgm_slot -v -h localhost -U replicauser & >> /var/lib/pgsql/9.6/pg_receivexlog.log 2>&1 < /dev/null
```

**Standby sunucudaki pg_hba.conf değişiklikleri**:

Slave sunucuda da bazı değişiklikler yapılmalıdır. Öncelikle postgresql.conf' da aşağıdaki değişikliği yapalım (Üstte de yazdığımız gibi, bu değişiklikler performans, vb değişikliklerin dışında olup, sadece standby ile ilgili olan değişikliklerdir). Burada, master sunucu değişikliklerini geri alabilirsiniz, eğer master'dan gelen dosyayı kullanıyorsanız aşağıdaki değişiklikleri yapınız:

```text
hot_standby=on 
hot_standby_feedback=on
```

**WAL arşivlemesi yönteminde:**

```text
standby_mode = 'on' 
primary_conninfo = 'host=192.168.1.5 port=5432 user=replicauser password=test' 
restore_command = 'cp /pgsql/9.6-walarchive/%f %p' 
archive_cleanup_command='/usr/pgsql-9.6/bin/pg_archivecleanup /pgsql/9.6-walarchive %r' 
trigger_file = '/var/lib/pgsql/9.6/data/mgm.failover'
recovery_target_timeline = 'latest'
```

**replication slot yönteminde:**

```text
standby_mode = 'on'
primary_conninfo = 'host=192.168.1.5 port=5432 user=replicauser password=test'
trigger_file = '/var/lib/pgsql/9.6/standby/stop.replication'
recovery_target_timeline = 'latest'
primary_slot_name = 'mgm_slot'
```

**`standby_mode:`** streaming replication'ı etkinleştirir.

**`primary_conninfo:`** master sunucuya bağlantı bilgisini içerir.

**`trigger_file:`** sunucuda görüldüğü zaman streaming replication'ın bitmesini tetikler (failover). Bu dosyayı yaratmak için “touch” komutunu kullanabilirsiniz. Bu dosya yaratıldıktan çok kısa süre sonra PostgreSQL standby sunucuda kendisini master modda açacaktır. Bu işlem sonunda recovery.conf dosyasının adı recovery.done olarak değiştirilecektir.

**`restore_command:`** WAL arşivinden WAL dosyalarının geri yüklenmesini sağlayan komutu içerir. Bu komut özellikle standby sunucu ilk başlatıldığında, standby'ın master sunucuyu yakalayabilmesi için kullanılır. Ayrıca, SR geri kalırsa, PostgreSQL veriyi WAL dosyalarından restore edecektir.

Eğer base backup'ı pg_basebackup ile almadıysanız, *postmaster.pid* dosyasını standby'dan silin.

Şimdi, *postgresql.conf*'daki değişiklikleri uygulayın, *recovery.conf*'u oluşturun ve standby sunucuyu çalıştırın. İkinci sunucu hot standby modunda çalışacaktır.

### Cascading Replikasyon

Cascading (basamaklı) replication, master sunucudan bir standby sunucu replikasyonu oluşturduktan sonra, o standby sunucudan başka bir standby sunucu daha oluşturmak anlamına gelir. Böylece 1 master sunucudan 2 standby çıkmak ve sunucuya yük getirmek yerine, 2. standby'ın yükünü 1. standby'a verip yükü azaltabilirsiniz. Burada sayısal açıdan bir sınır bulunmamakla beraber, pratikte WAL trafiğiyle oluşacak ağ yükü nedeniyle çok fazla standby yapmamak önerilir.

Burada yapmanız gereken şey, genel olarak üstteki ile aynıdır. Farkı şudur: 1. standby sunucusu hem standby olacak (hot_standby=on), hem de 2. standby sunucu için master olacak (max_wal_senders 0'dan büyük olmalı ve diğer master sunucu parametreleri de olmalı). Benzer şekilde, iki standby sunucusunda da recovery.conf dosyası tabii ki olmalıdır.

### Synchronous Replikasyon

Synchronous replication, master sunucudaki bir transaction'ın, (en az) 1 standby sunucuya da aynı anda commit edilmesi demektir. Asynchronous replication'da, master sunucuya girilen veri, WAL dosyasına yazıldıktan sonra standby sunucuya walsender aracılığı ile aktarılır. Synchronous replication'da ise, bu işlem aynı anda yapılır.

Synchronous replication'da, en az 1 aktif standby sunucu olmak zorundadır. Bu sunucu olmazsa, master sunucudaki tüm sorgular duraklatılır ve standby sunuculardan en az birisi ayağa kaldırılana kadar bekletilir. Burada `wal_keep_segments` parametresi daha da önem kazanacaktır.

Öncelikli olarak senkronize edilecek olan sunucu dışındaki standby sunucular “potansiyel standby” diye geçerler. Eğer 1. standby sunucu devre dışı kalırsa, sıradaki sunucu senkron için öncelik sırasına girecektir. İşte bu nedenlerden dolayı synchronous replication kullanırken, en az 2 standby sunucu olması önerilir.

Synchronous replication'ı kurarken kullanmanız gereken tek farklı parametre, `synchronous_standby_names` parametresidir. Burada,
*recovery.conf* içindeki `primary_conninfo` parametresine `application_name` ile girdiğimiz sunucu adlarını yazıyoruz. Örneğin, primary_conninfo parametreleri, *recovery.conf* içinde:

```text
primary_conninfo = 'host=192.168.1.5 port=5432 user=replicauser password='test' application_name='sb1' 
primary_conninfo = 'host=192.168.1.5 port=5432 user=replicauser password='test' application_name='sb2' 
```

olarak ayarlanmışsa, bu parametreyi,

```text
synchronous_standby_names='sb1,sb2'
```

şeklinde yazmak yeterlidir. 1'den fazla standby sunucunuz olduğunda sadece senkron olmasını istediklerinizi buraya yazabilirsiniz. Bu parametreye `*` yazmak, tüm application_name 'leri kabul etmek demektir.

**Standby sunucular için ek replikasyon parametreleri**:

Bu bölümde, *postgresql.conf* içindeki ek standby sunucu replikasyon parametreleri ele alınmıştır.

**`wal_receiver_status_interval:`** Standby sunucu(lar), replikasyon süreci ile ilgili bilgileri bir üstteki sunucuya (bu master sunucu olabilir, ya da kendisinden önce standby sunucu olabilir) gönderirler. İki gönderim arası en fazla, wal_receiver_status_interval karar sürede gerçekleşir. Öntanımlı değeri 10 saniyedir. Güncellemelerin sıklığı iki şekilde ayarlanır:

- wal_receiver_status_interval süresi kadar
- WAL’lara eklenen her yeni transaction işlemi sonrasında

Bunlardan hangisi önce olursa, o zaman bilgiler güncellenir. Dolayısıyla, buradaki bilgiler çok güncel olmayabilir.
Bu bilgiler `pg_stat_replication` içinde görünür. Bu view içindeki kolonların ayrıntılarına [buradan](http://www.postgresql.org/docs/devel/static/monitoring-stats.html#PGSTAT-REPLICATION-VIEW) ulaşabilirsiniz. Bu değer 0 olursa, durum güncelleme devre dışı kalır.

**`hot_standby_feedback`**: Boolean bir değerdir ve öntanımlı değeri `off`'dur. HS sunucusunun, bir üstteki sunucuya, (tanımını önceki parametrede yapmıştık) o anda standby sunucuda çalışan sorgularla ilgili bilgi gönderip göndermeyeceğini belirler. Bu parametreyi engellemek özellikle uzun sürebilecek sorgularda, üstteki sunucuda cleanup işleminin yapılmamasını sağlar ve yararlı olur. Bu işlem, üst sunucudaki bloat'ı arttıracaktır. Bu bilgiler, üstteki `wal_receiver_status_interval` değerinden daha sık gönderilmez. Raporlama amaçlı kullanılan HS sunucularında bu parametreyi açmak yararlı olabilir. Eğer cascading replication varsa, 1. standby'ın altından gelen feedback mesajları, aracı olan sunucular üzerinden master sunucuya aktarılır.

**`wal_receiver_timeout`**: Öntanımlı olarak 60 sn olan bu parametre ile, replikasyon bağlantıları belirtilen zaman aşımı süresinden daha uzun sürerse sonlandırılır. PostgreSQL belgelerinde de yazıldığı gibi, özellikle master sunucunun çöküp çökmediğinin anlaşılması açısından kullanılabilir. Bu değer 0 olursa, bu parametre devre dışı kalır.

**Replikasyon gecikmesini hesaplamak**:

Replikasyon gecikmelerini hesaplamak için, PostgreSQL’deki `pg_xlog_location_diff()` fonksiyonunu kullanabilirsiniz:

```text
SELECT client_hostname, client_addr, 
    pg_xlog_location_diff(pg_stat_replication.sent_location, 
    pg_stat_replication.replay_location) AS byte_lag 
    FROM pg_stat_replication;
```

{% include links.html %}
