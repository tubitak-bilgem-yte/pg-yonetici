---
title: "PostgreSQL Yedekleme ve PITR Kavramı"
parent: Yedekleme ve Güncelleme
layout: default
nav_order: 5
---

## Yedekleme ve PITR kavramı

PostgreSQL'de yedekleme iki şekilde yapılabilir.

{% include callout.html content=" **`SQL dump`**: Veritabanının hepsini ya da belirli bir bölümünün o andaki halini almak için kullanılır. Veri kaybının potansiyel olarak büyük olması nedeniyle kullanılmasını pek önerilmez." type="primary" %}

{% include callout.html content=" **`WAL tabanlı yedeklemeler`**: Replikasyon altyapısına benzer şekilde, base backup alınıp WAL dosyalarının yedeklenmesi temeline dayanır.Tüm büyük ya da veri kaybına tahammülü olmayan veritabanlarınıbu şekilde yedeklemek gerekmektedir." type="primary" %}

Point In Time Recovery (PITR) PostgreSQL'de veri kurtarma tekniklerini geliştirmek amacıyla getirilmiş bir özelliktir. Bir çok kurumsal veritabanında da PITR bulunmaktadır. Bu özellik sayesinde çökme anında veri kurtarmanın olası olamayabileceği durumlar en aza indirilir. Bu durumları şu şekilde özetleyebiliriz:

- Veritabanı nesneleri silinmiş olabilir (Örnek: DROP DATABASE)
- WAL'lar, REDO işlemi için yeteri kadar geriye gitmiyor olabilir.
- Veritabanı dosyaları eksiksiz değildir ve REDO işleminden önce tamamen değiştirilmelidir.

{% include callout.html content=" Bu durumlarda, veriyi kurtarabilmek için sisteminizin tam fiziksel yedeğinin olması gerekir. Tablespaceler sayesinde, tablespaceleri teker teker yedekleyip onları kurtarmak mümkündür. Ayrıca WAL'ların normal WAL dosya sisteminden çıkartılıp başka bir yere yedeklenmesinin sağlanması gerekir." type="primary" %}

PITR'nin gerçekleşebilmesi için, belirli bir zaman diliminde PostgreSQL'in çalışma anında alınmış eksiksiz bir fiziksel yedeği bulunmalıdır. Bu yedek tüm veri ve commit loglarının, ayrıca (varsa) kısayollarla belirtilmiş tüm tablespaceleri içermelidir. Ayrıca, tüm WAL'lar da sıralı olarak elinizde bulunmalıdır.

PITR çözümü iki temel işlemden oluşur:

1. **WAL arşivlenmesi**: Verinin kurtarılabilmesi için, üstte de yazıldığı gibi, WAL'ların elimizde olması gerekmektedir. PostgreSQL PITR API'si, tüm logların arşivlenmesi gerektiğini kabul eder. Böyle koşulsuz olarak tüm WALlar, *postgresql.conf* dosyasında belirtilecek şekilde yedeklenir.
2. **Belirli bir zamana kadar kurtarma işlemi (Recovery-to-point-in-time – RPIT)**: Verinin belirli bir zamana kadar kurtarılması,aşağıdaki seçenekleri bize sunmaktadır:
   - WALların sonuna kadar veri kurtarma
   - Mevcut tüm WALların sonuna kadar veri kurtarma.
   - Belirtilen ana kadar olan verilerin kurtarılması
   - Belirtilen transaction id'sine (xid) kadar kurtarma. Son iki işlem UNDO işlemidir.

Bunlardan birincisi zaten PostgreSQL içinde olan bir özellikti (yukarıda açıklanmıştı).

Bunların gerçekleşebilmesi için en temel kural şudur: “PostgreSQL'i archive modda başlatın ve temel yedeğinizi (base backup) alın.”

Veritabanı sunucusu archive modunda iken çökerse, daha öncedenolduğu şekilde (yukardaki ilk madde) veriyi kurtaracaktır. Eğer ana sistem çökerse, ya da başka bir nedenle veri kurtarmaya gereksinim duyulursa, archive recovery modu aşağıdaki süreçlerle tetiklenebilir:

1. Daha önceden alınmış olan eksiksiz veritabanı yedeğinin geri yüklenmesi,
2. `$PGDATA` dizininin içinde *recovery.conf* dosyasının uygun içerikle oluşturulması,
3. PostgreSQL' in yeniden başlatılması,

Kurtarma sürecinin başarılı olması için aşağıdakilerin mutlaka sağlanması gerekir:

1. PostgreSQL içindeki her bir dosyanın tam yedeğinin alınması (eksiksiz olması sürecin gerçekleşebilmesi için çok önemlidir).
2. Temel yedeğin alınmasının hemen ardından başlayan ve kurtarma işleminin sonlanacağı ana kadar olan verileri içeren, arşivlenmiş WAL logları.

{% include callout.html content=" Eğer bu iki maddeden birisinde küçük de olsa bir eksiklik olursa, sadece bu zincirin kırıldığı yere kadar olan kısmın kurtarılabileceğini lütfen unutmayın." type="primary" %}

PostgreSQL size wal yedeklemesinde hiç bir kısıtlama getirmez. Bu parametre içerisine, o işletim sisteminde kullanılabilecek herhangi bir komut verilebilir, hatta kendi özel betiklerinizi (script) de kullanabilirsiniz.

### pg_dump ile Yedekleme

pg_dump’ın sürekli değişen veriyi alamaması, incremental yedek alamaması gibi sebeplerde dolayı yedekleme amaçlı kullanımı önerilen bir yöntem değildir. Ancak, veri taşımak gerektiğinde pg_dump kullanılabilir.

Öncelikle pg_dump komutunu inceleyelim:

{% include callout.html content=" pg_dump, veritabanının yedeğini aldığınız uygulamadır. Veritabanının, şemaların, tabloların ya da hepsinin yedeğini alabilir. Bunu ona vereceğiniz parametrelerle yapar. Bir dump işlemi bir transaction içinde gerçekleştirilir. Bu da uzun süren dumpların uzun süren bir transaction olacağı anlamına gelir. Öntanımlı olarak düz metin formatında dump alır. Bu bir SQL betiği (script) gibidir; ve psql ile veritabanına tekrar yüklenebilir. Veritabanınızı bu formatta yedeklemeniz, yedeklerinizi bir sürüm kontrol yazılımına atabilmenizi ve farkları görebilmenizi de sağlar. Düz metin yedekleri `pg_restore` ile veritabanı sunucusuna tekrar yükleyemezsiniz." type="primary" %}

{% include callout.html content=" pg_dump yedekleyeceği tablolarda ACCESS SHARE lock ister. Bu kararlı bir yedek için gereklidir. pg_dump, sunucu çalışırken yedek alabilme yeteneğine sahiptir. Buna hot backup da denir. pg_dump, yedek almaya başladığı anda veritabanının bir snapshotını alır. Bu, stable bir yedek için gereklidir." type="primary" %}

{% include callout.html content=" pg_dump, diğer client ayarları gibi `-p`, `-h`, `-U` gibi parametreleri bağlantı bilgileri için alır. pg_dump ile alınan yedekler, yedek alma sürecinin başlangıç anındaki verileri içerir. Üstte de belirttiğim gibi bir snapshot aldığı için veri bütünlüğü bozulmaz." type="primary" %}

{% include callout.html content=" pg_dump öntanımlı olarak COPY formatında yedek alır. Bu format yüklerken önemli bir başarım artışı sağlar." type="primary" %}

pg_dump kullanımını basit bir şekilde örnekleyelim:

```sql
pg_dump yte -U postgres > /var/lib/pgsql/13/backups/yte.pgdump

-- ya da

pg_dump yte -U postgres -f /var/lib/pgsql/13/backups/yte.pgdump
```

komutu, yte veritabanının yedeğini alır. Bazı pg_dump parametrelerini inceleyelim:

- Üstte de gördüğünüz `-f` parametresi ile yedeğin yazılacağı dosyaadını belirtebilirsiniz.

- Daha önce de bahsettiğimiz gibi pg_dump öntanımlı olarak düz metin formatınde yedek alır. Yedeğin formatını belirleyen parametre `-F`'dir ve yanına bir harf daha alır. Öntanımlı olan `p` harfi (plain) ile düz metin yedek alır. pg_dump'a `-Fc` parametresini verirseniz, **Custom** formatta bir dump dosyanız olur. Bu format, düz metin formatından daha esnektir. `-Fc` ile alınan yedeği sadece pg_restore okuyabilir, yani bu dosya bir binary dosyadır. Düzenli ve programlı tam yedek almak için custom format en iyi seçimdir. Otomatik olarak sıkıştırılır ve dolayısıyla gerçek anlamda az yer kaplarlar. Son olarak da `-Fd` parametresi ile yedeklerinizi her bir tablo ayrı dosyada olmak üzere bir dizine alabilirsiniz. Bu yöntemin avantajı, pg_dump'ı `-j` parametresi ile paralel olarak çalıştırabilecek ve dump süresini kısaltabilecek olmanızdır.

- `-Ft` parametresi arşivi UNIX tar formatında alır. Bu formatın custom formata göre bir avantajı olmadığı gibi, bazı kısıtlamaları yüzünden de geniş bir kullanım alanı yoktur. Az kullanıldığı için de az test edilmiştir. Kullanmanı pek önerilmez.

- `-n` parametresi ile sadece belirli bir namespace'i (şema) ya da birden fazla namespace'i, `-t` ile de sadece belirli bir tabloyu ya da birden fazla tabloyu yedekleyebilirsiniz. Bu parametreler düzenli ifadeleri (regular expression) de kabul ederler; o yüzden kullanılabilirlikleri oldukça yüksektir. Birden fazla nesne yazmak için aralarına virgül koyabilirsiniz. Benzer şekilde `-N` ve `-T` parametreleri de vardır. `-n` ve `-t` parametrelerinden farklı olarak bu iki parametre ile yedeğe dahil edilmesini istemediğiniz namespace ya da tabloları belirtebilirsiniz.

- `-c` parametresi, yedek dosyasının başına, CREATE satırlarından önce DROP satırlarının konmasını sağlar. Özellikle yedeğinizi yükleyeceğiniz veritabanında tabloları kendiniz kaldırmak istemezseniz kullanabileceğiniz bir parametredir.

### pg_dumpall

{% include callout.html content=" pg_dumpall ile bir PostgreSQL kurulumundaki tüm veritabanlarının yedeğini alabilirsiniz. pg_dump ile hemen hemen aynı parametreleri kullanır; ancak bazı önemli farklılıklar vardır. Sadece düz metin biçiminde yedek üretebilir, yani pg_dump' daki gibi custom format parametreleri yoktur. Ayrıca veritabanı adını da bir parametre olarak kullanmaz." type="primary" %}

- Bu komutun en önemli parametrelerinden birisi de `-g` parametresidir. Bu parametre “global” nesnelerin (role ve tablespace bilgileri) yedeğini alır. Bu parametre sayesinde kullanıcıları bir veritabanından başka PostgreSQLsunucusuna taşıyabilirsiniz. pg_dumpall sadece düz metin (plain text) biçimine yedek alabildiği için tüm veritabanı sunucusunun yedeğini almak için çok uygun bir araç değildir. Burada öncelikle global nesnelerin yedeğini alıp, sonra pg_dump ile her veritabanının yedeğini custom formatta (`pg_dump -Fc`) almak daha iyi olacaktır. Bu sayede tek bir veritabanını aradan yükleyebilirsiniz. Bu pg_dumpall ile alınan yedekle mümkün değildir.

- `-g` parametresi `-r` (sadece roller) ve `-t` (sadece tablespaceler) parametrelerinin toplamı olarak da adlandırılabilir.
- Yine bir fark olarak pg_dump'daki `-n`, `-N`, `-t` parametreleri pg_dumpall'da yoktur.
- Bunların dışında, `-a`, `-c`, `-d`, `-D`, `-f`, `-i` parametreleri pg_dump ile ortaktır.

### pg_restore

Eğer düz metin bir yedek dosyanız varsa, onu üstte bahsettiğimiz gibi psql ya da bir grafiksel arabirim aracılığı ile yükleyebilirsiniz. Eğer custom formatta bir yedeğiniz varsa, bunu pg_restore ile yükleyebilirsiniz. Bu komut size bir miktar esneklik sağlar. Sadece datayı (`-a`), sadece şemayı (`-s`), sadece tabloları (`-t`), sadece namespaceleri (`-n`) yükleyebilir; ayrıca tüm veritabanının yüklenmesini bile sınırlandırmak bile mümkündür. pg_restore ayrıca parallel restore özelliğine de sahiptir. Bu özellik aşağıda ayrıntılı olarak açıklanmıştır.

pg_restore komutunun en önemli ve kullanışlı özelliklerinden birisi deyedeğin içindekileri -l parametresi ile görebilirsiniz.

```sql
pg_restore -l  pagila.customdump | less
```

Sadece belirli bir tablonun yüklenmesi de `-t` parametresi ile olur. Eğer bir dosya adı vermezseniz çıktı ekrana akar. Bu özellik sayesinde paralel veri yüklemesi yapabilirsiniz. Eğer birden fazla işlemciniz ve bol bol disk bant genişliğiniz varsa, her bir işlemciyi ayrı bir restore işleminde kullanarak başarımı arttırabilirsiniz. Benzer şekilde, indexleri de en sona bırakabilir ve onları paralel yaratabilirsiniz. Ancak bu yine de parallel restore özelliği kadar başarım artışısağlamaz. Paralel restore özelliğini anlatmak için pagila veritabanını kullanmayacağım; zira pagilanın veri boyutu bu test için küçüktür. Örnek olarak kendi tablomuzu yaratalım:

```sql
CREATE TABLE restore_denemesi (c1 int);

INSERT
	INTO
	restore_denemesi
SELECT
	generate_series(1,
	10000000);

CREATE INDEX restore_denemesi_c1_idx ON
restore_denemesi (c1);

CREATE TABLE restore_denemesi_2 (c1 int);

CREATE TABLE
INSERT
	INTO
	restore_denemesi_2 SELECTgenerate_series(1,
	100000000);

CREATE INDEX restore_denemesi_2_c1_idx ON
restore_denemesi_2(c1);
```

Bu işlemlerden sonra pg_dump ile yedek alalım

```bash
$ pg_dump pgrestore -Fc -f pgrestore.dump
```

Şimdi bunu yükleyelim. Öncelikle tek görev ile deneyelim:

```bash
$ time pg_restore -d pgrestore pgrestore.dump

real 7m30.753s
user 0m47.639s
sys  0m0.967s
```

7.5 dakika sürdü. Şimdi de 2 job ile deneyelim. Parallel job sayısını `-j` ile belirtebilirsiniz:

```sql
-bash-4.2$ time pg_restore -d pgrestore pgrestore.dump -j 2

real 4m42.706s
user 0m45.938s
sys  0m1.179s
```

### Pgbackrest ile Yedekleme

Yedeklemenin çalışabilmesi için [pgbackrest](https://pgbackrest.org/) sunucusundan PostgreSQL sunucusuna *postgres* kullanıcısından *postgres* kullanıcısına parolasız ssh yapabilmek gereklidir. Benzer şekilde de yedeği geriyüklemek için pgbackrest sunucusundan PostgreSQL sunucusuna da parolasız SSH yapmak gerekir.

Aşağıdaki komutu iki sunucuda da *postgres* kullanıcısı olarak çalıştırın ve sizden parola istendiğinde enter'a basıp devam edin:

```bash
ssh-keygen -t rsa
```

Şimdi, ssh anahtarımızı saklayacağımız dosyayı yaratalım ve gerekliizinlerini verelim. Bunu da postgres kullanıcısı ile çalıştıralım:

```bash
touch ~/.ssh/authorized_keys 
chmod 600 ~/.ssh/authorized_keys
```

Ardından sunuculardaki *~/.ssh/id_rsa.pub* dosyasının içeriğini diğersunucudaki *~/.ssh/authorized_keys* dosyasına tek satırda aynen aktarınız.

Son aşamada, sunucular arasında *postgres* kullanıcısından ssh yapınız. Hem host keyleri saklamış olur, hem de yaptıklarımızı denemiş olursunuz.

Debian/Ubuntu'da öncelikle PostgreSQL'in APT deposunu kurmak gerekmektedir. Sonra pgbackrest'i kurabiliriz. Pgbackrest'i iki sunucuya da kuracağız:

```bash
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" >/etc/apt/sources.list.d/pgdg.list'

apt-get install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt-get update
apt-get upgrade

apt-get install pgbackrest
```

Ardından, aşağıdaki komutları iki sunucuda da çalıştırın:

```bash
mkdir -m 750 /var/lib/pgbackrest
chown postgres:postgres /var/lib/pgbackrest
```

*DB sunucusunda:*

pgbackrest'in yapılandırma dosyası /etc/pgbackrest.conf'dur. Bu dosyayı açın ve şu satırları ekleyin:

```bash
[global]
repo-path=/var/lib/pgbackrest
retention-full=2
backup-host= <BACKUP_SUNUCUSUNUN_IP_ADRESI>
backup-user=postgres

[<stanza ismi>]
db-path=/var/lib/pgsql/13/data
db-port=5432
```

*Yedek sunucusunda:*

```bash
[global]
repo-path=/var/lib/pgbackrest
retention-full=2

[<stanza ismi>]
db-path=/var/lib/pgsql/13/data
db-port=5432
db-host= <POSTGRES_SUNUCUSUNUN_IPSI>
```

Şimdi pgbackrest yapılandırmasına devam edebiliriz. PostgreSQL yapılandırma dosyasında şu satırları değiştirelim:

```bash
listen_addresses = '*'
archive_mode = on
archive_command = 'pgbackrest --stanza=<stanza ismi> archive-push %p'
wal_level = replica
max_wal_senders = 10
```

Bu ayarlardan sonra PostgreSQL'i restart edelim.

Şimdi `stanza-create`'i yedek sunucusunda *postgres* kullanıcısı olarak çalıştıralım:

```bash
pgbackrest --stanza=<stanza ismi> --log-level-console=info stanza-create
```

Eğer herşey düzgün ise aşağıdaki mesajı alacaksınız: *"INFO: stanza-create command end: completed successfully"*

Yedek almak için aşağıdaki komutu crona *postgres* kullanıcısı olarak girelim:

```bash
pgbackrest --stanza=<stanza ismi>  --log-level-console=info check
```

Örnek:

```bash
0 2 * * * pgbackrest –stanza=<stanza ismi> –log-level-console=info backup
```

Bu komutu yedek sunucusunda konsolda bir kez çalıştırıp bir sorun olup olmadığını kontrol edebilirsiniz.

### Pgbackrest ile Alınan Yedekten Geri Dönmek ve PITR

Pgbackrest ile alınan yedeği restore etmek (en son base_backuptan geri dönülür). Bu komutu pgbackrest sunucusunda çalıştırıyoruz:

```bash
pgbackrest --stanza=<stanza ismi> --log-level-console=info restore
```

PITR yapmak içinse tarih ve zamanı belirtebiliriz:

```bash
pgbackrest --stanza=<stanza ismi> --type time "--target=2020-07-03 12:11:30" --set=20200703-115303 --log-level-console=info restore
```

### Diğer Yedekleme Yöntemleri

#### Barman

- Barman, 2ndQuadrant ve topluluk tarafından geliştirilen bir özgür yazılım. [](http://www.pgbarman.org/)
- Sıcak yedek sunucular, PITR, tam ve arttırımlı yedekleme, çoklu ana ve yedek sunucu desteği
- Barman katalogu sayesinde yedek ve geri dönüşleri kolay yönetme.

Ayrıntılı bilgi için [Barman](mydoc_barman.html) sayfasına bakınız.

#### Dosya Sistemi Seviyesinde Yedekleme

PostgreSQL veri dizinini kopyalayarak da yedek alabiliriz

```sh
$ tar cvfz /tmp/yedek.tar.gz /var/lib/pgsql/11/data/
```

Ancak bu yöntemin eksileri var!

- Tutarlı bir yedek olması için PostgreSQL servisinin kapalı olması gerekir.
- Servis kapalı değilse WAL logları lazım, o zaman PostgreSQL kendini kurtarabilir.
- Bu şekilde alınan yedekten tek tek veritabanı ya da tabloları geri yüklemek mümkün olmaz

{% include links.html %}
