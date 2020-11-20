---
title: "Veritabanı Kümesi Oluşturma"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 20, 2020
summary: "Veritabanı Kümesi Oluşturma"
sidebar: mydoc_sidebar
permalink: mydoc_vt_kumesi_olusturma.html
folder: mydoc
---

## Veritabanı Kümesi Oluşturma

Herhangi bir şey yapmadan önce, diskte bir veritabanı depolama alanını başlatmalısınız. Buna veritabanı kümesi ( database cluster ) diyoruz. (SQL standardı, katalog kümesi terimini kullanır). Bir veritabanı kümesi, çalışan bir veritabanı sunucusunun tek bir örneği ( instance ) tarafından yönetilen bir veritabanları koleksiyonudur. Veritabanı kümesi başlatma sonrasında, yardımcı programlar, kullanıcılar ve üçüncü taraf uygulamalar tarafından kullanılmak üzere varsayılan bir veritabanı anlamına gelen `postgres` adlı bir veritabanı içerecektir. Veritabanı sunucusunun kendisi postgres veritabanının var olmasını gerektirmez, ancak birçok harici yardımcı program var olduğunu varsayar. Başlatma sırasında her kümede oluşturulan başka bir veritabanına `template1` adı verilir. Adından da anlaşılacağı gibi bu, sonradan oluşturulan veritabanları için bir şablon olarak kullanılacaktır; fiili iş için kullanılmamalıdır. (Bir küme içinde yeni veritabanları oluşturma hakkında bilgi için [Veritabanlarını Yönetme]("") bölümüne bakınız.)

Dosya sistemi açısından bir veritabanı kümesi, tüm verilerin depolanacağı tek bir dizindir. Buna veri dizini ( data directory ) veya veri alanı diyoruz. Verilerinizi nerede depolayacağınız tamamen size bağlıdır. Varsayılan yoktur, ancak `/usr/local/pgsql/data` veya `/var/lib/pgsql/data` gibi konumlar popülerdir. Veri dizini kullanılmadan önce, PostgreSQL ile kurulan [`initdb`](https://www.postgresql.org/docs/13/app-initdb.html) programı kullanılarak ilklendirilmelidir.

{% include important.html content="PostgreSQL'in önceden paketlenmiş bir sürümünü kullanıyorsanız, veri dizininin nereye yerleştirileceğine dair belirli bir kuralı olabilir ve ayrıca veri dizinini oluşturmak için bir komut dosyası da sağlayabilir. Bu durumda, `initdb`'yi doğrudan çalıştırmak yerine o betiği kullanmalısınız. Ayrıntılar için paket düzeyindeki belgelere bakınız." %}

Bir veritabanı kümesini manuel olarak başlatmak için, initdb'yi çalıştırın ve veritabanı kümesinin istenen dosya sistemi konumunu `-D` seçeneğiyle belirtin, örneğin:

```bash
$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content="Bu komutu, önceki bölümde anlatılan PostgreSQL kullanıcı hesabında oturum açtığınızda çalıştırmanız gerektiğini unutmayın. Ayrıca, `-D` seçeneğine bir alternatif olarak, ortam değişkeni `PGDATA`'yı ayarlayabilirsiniz." type="info" %}

Alternatif olarak, initdb'yi `pg_ctl` programı aracılığıyla şu şekilde çalıştırabilirsiniz:

```bash
$ pg_ctl -D /usr/local/pgsql/data initdb
```

Belirttiğiniz dizin önceden yoksa, initdb belirtilen dizini yaratmaya çalışır. Ancak, initdb'nin üst dizine yazma izni yoksa bu başarısız olacaktır. PostgreSQL kullanıcısının sadece veri dizinine değil aynı zamanda onun ana dizinine de sahip olması genellikle tavsiye edilir, bu yüzden bu bir problem olmamalıdır. İstenen üst dizin yoksa, root ayrıcalıklarını kullanarak önce onu oluşturmanız ve gerekli yetkileri vermeniz gerekir. Dolayısıyla süreç şöyle olur:

```bash
root# mkdir /usr/local/pgsql
root# chown postgres /usr/local/pgsql
root# su postgres
postgres$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content="initdb, veri dizini varsa ve zaten dosyalar içeriyorsa çalışmayı reddeder. Bu, yanlışlıkla mevcut bir kurulumun üzerine yazılmasını önlemek için alınan bir önlemdir." type="info" %}

Veri dizini, veri tabanında depolanan tüm verileri içerdiğinden, yetkisiz erişime karşı güvence altına alınması önemlidir. Bu nedenle `initdb`, PostgreSQL kullanıcısı ve isteğe bağlı olarak grup dışındaki herkesin erişim izinlerini iptal eder. Grup erişimi etkinleştirildiğinde salt okunurdur. Bu, küme sahibiyle aynı gruptaki ayrıcalıksız bir kullanıcının küme verilerinin yedeğini almasına veya yalnızca okuma erişimi gerektiren diğer işlemleri gerçekleştirmesine olanak tanır.

Mevcut bir kümede grup erişiminin etkinleştirilmesi veya devre dışı bırakılmasının, kümenin kapatılmasını ve PostgreSQL'i yeniden başlatmadan önce tüm dizinlerde ve dosyalarda uygun modun ayarlanmasını gerektirdiğini unutmayın. Aksi takdirde, veri dizininde karışık modlar bulunabilir. Yalnızca sahip tarafından erişime izin veren kümeler için uygun modlar dizinlerde `0700` ve dosyalarda `0600` şeklindedir. Grup tarafından okumaya da izin veren kümeler için uygun modlar, dizinlerde `0750` ve dosyalarda `0640`'tır.

Bununla birlikte, dizin içerikleri güvendeyken, varsayılan istemci kimlik doğrulama kurulumu, herhangi bir yerel kullanıcının veritabanına bağlanmasına ve hatta veritabanı süper kullanıcısı olmasına izin verir. Diğer yerel kullanıcılara güvenmiyorsanız, veritabanı süper kullanıcısına bir parola atamak için initdb'nin `-W`, `--pwprompt` veya `--pwfile` seçeneklerinden birini kullanmanızı öneririz. Ayrıca, varsayılan `trust` kimlik doğrulama modunun kullanılmaması için `-A md5` veya `-A password` belirtin; veya oluşturulan `pg_hba.conf` dosyasını initdb'yi çalıştırdıktan sonra değiştirin. (Diğer makul yaklaşımlar, bağlantıları kısıtlamak için `` kimlik doğrulaması veya dosya sistemi izinlerini kullanmayı içerir. Daha fazla bilgi için [İstemci Kimlik Doğrulaması]("") bölümüne bakın.)

`initdb` ayrıca veritabanı kümesi için varsayılan yerel (locale) ayarı da başlatır. Normalde, ortamdaki yerel ayarları alır ve bunları başlatılmış veritabanına uygular. Veritabanı için farklı bir yerel ayar belirtmek mümkündür; bununla ilgili daha fazla bilgi [Locale Support]("") bölümünde bulunabilir. Belirli bir veritabanı kümesinde kullanılan varsayılan sıralama düzeni, initdb tarafından belirlenir ve farklı sıralama düzeni kullanarak yeni veritabanları oluştururken initdb'nin oluşturduğu şablon veritabanlarında kullanılan sıralama düzeni, drop ve recreate edilmeden değiştirilemez. `C` veya `POSIX` dışındaki yerel ayarları kullanmanın da bir performans etkisi vardır. Bu nedenle ilk seferde bu seçimi doğru yapmak önemlidir.

`initdb` ayrıca veritabanı kümesi için varsayılan karakter kümesi kodlamasını da ayarlar. Normalde bu, yerel ayara uyacak şekilde seçilmelidir. Ayrıntılar için [Localization]("") bölümüne bakınız.

`Non-C` ve `non-POSIX` yerel ayarları, karakter kümesi sıralaması için işletim sisteminin collation kütüphanesini kullanır. Bu, indekslerde saklanan anahtarların sırasını kontrol eder. Bu nedenle bir küme, anlık görüntü geri yüklemesi ( snapshot restore ), binary streaming replication, farklı bir işletim sistemi veya bir işletim sistemi yükseltmesi yoluyla uyumlu olmayan bir collation kütüphanesi sürümüne geçemez.

## İkincil Dosya Sistemlerinin Kullanımı

Çoğu kurulum, veritabanı kümelerini makinenin "root" birimi dışındaki dosya sistemlerinde oluşturur. Bunu yapmayı seçerseniz, ikincil birimin en üstteki dizinini (mount point) veri dizini olarak kullanmayı denemeniz tavsiye edilmez. En iyi uygulama, PostgreSQL kullanıcısının sahip olduğu bağlama noktası dizini ( mount-point directory ) içinde bir dizin oluşturmak ve ardından bunun içinde veri dizinini oluşturmaktır. Bu, özellikle `pg_upgrade` gibi işlemler için izin sorunlarını önler ve ayrıca ikincil birim çevrimdışı duruma getirilirse temiz hatalar sağlar.

## Dosya Sistemleri

Genellikle, POSIX semantiğine sahip herhangi bir dosya sistemi PostgreSQL için kullanılabilir. Kullanıcılar; satıcı desteği, performans ve aşinalık gibi çeşitli nedenlerle farklı dosya sistemlerini tercih eder. Deneyimler bize diğer tüm şeyler eşit olduğunda durumda, yalnızca dosya sistemlerini değiştirerek veya küçük dosya sistemi yapılandırması değişiklikleri yaparak büyük performans veya davranış değişiklikleri beklenmemesi gerektiğini göstermiştir.

### NFS

PostgreSQL veri dizinini depolamak için bir NFS dosya sistemi kullanmak mümkündür. PostgreSQL, NFS dosya sistemleri için özel bir şey yapmaz, yani NFS'nin tam olarak yerel olarak bağlı sürücüler gibi davrandığını varsayar. PostgreSQL, dosya kilitleme ( file locking ) gibi NFS üzerinde standart olmayan davranışa sahip olduğu bilinen herhangi bir işlevselliği kullanmaz.

{% include links.html %}
