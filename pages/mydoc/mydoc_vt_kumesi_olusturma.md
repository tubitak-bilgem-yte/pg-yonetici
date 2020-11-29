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

Veritabı kümesi oluşturma işlemine ilk olarak disk üzerinde bir veritabanı depolama alanı oluşturularak başlanır. PostgreSQL'de buna veritabanı kümesi ( database cluster ) denilirken SQL standardı, katalog kümesi terimini kullanır. Veritabanı kümesi, çalışan bir veritabanı sunucusunun tek bir ana programı ( instance ) tarafından yönetilen veritabanları koleksiyonudur. Veritabanı kümesi başlatıldıktan sonra yardımcı programlar, kullanıcılar ve üçüncü taraf uygulamalar tarafından kullanılmak üzere varsayılan `postgres` isimli bir veritabanı beraberinde gelir. Veritabanı sunucusu, postgres veritabanının var olmasını gerektirmezken birçok harici yardımcı program var olduğunu varsayar. Başlatma sırasında her kümede oluşturulan bir başka veritabanına da `template1` veritabanıdır. İsminden de anlaşılacağı üzere `template1` oluşturulacak veritabanları için şablon olarak kullanılacaktır, operasyonel işler için kullanılmamalıdır. Bir veritabanı kümesinde yeni veritabanları oluşturma konusu için [Veritabanlarını Yönetme]("") bölümüne bakınız.

Dosya sistemi açısından bir veritabanı kümesi tüm verilerin depolanacağı tek bir dizindir. Buna veri dizini ( *data directory* ) veya veri alanı denilir. Verilerin nerede depolayacağı tamamen size bağlıdır. Varsayılan olmamakla birlikte `/usr/local/pgsql/data` veya `/var/lib/pgsql/data` gibi konumlar popülerdir. Veri dizini ve küme alt yapısı, PostgreSQL ile gelen [`initdb`](https://www.postgresql.org/docs/current/app-initdb.html) programı kullanılarak oluşturulur. (initialize)

{% include important.html content="Önceden paketlenmiş bir PostgreSQL sürümünü kullanıyorsanız, bu sürüm veri dizininin nereye yerleştirileceğine dair belirli bir kurala sahip olabilir. Bunun yanında veri dizinini oluşturmak için komut dosyası da sağlayabilir. Bu durumda `initdb`'yi doğrudan çalıştırmak yerine sağlanan betiği kullanmalısınız. Ayrıntılar için paket düzeyindeki belgelere bakınız." %}

Veritabanı kümesini manuel olarak başlatmak için initdb'yi çalıştırın ve veritabanı kümesi konumunu `-D` parametresiyle belirtin,

```bash
$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content="Bu komut önceki bölümde anlatılan PostgreSQL kullanıcısı hesabında çalıştırdığınızdan emin olun. Ayrıca, `-D` parametresine alternatif olarak `PGDATA` ortam değişkeni ayarlanabilir." type="info" %}

Alternatif olarak, initdb'yi `pg_ctl` programını kullanarak şu şekilde çalıştırabilirsiniz:

```bash
$ pg_ctl -D /usr/local/pgsql/data initdb
```

Verilen dizin yoksa, initdb belirtilen dizini yaratmaya çalışır. Fakat, initdb'nin üst dizine yazma izni yoksa bu işlem başarısız olacaktır. PostgreSQL kullanıcısının sadece veri dizinine değil aynı zamanda onun ana dizinine de sahip olması genellikle tavsiye edilir. İstenen üst dizin yoksa, root ayrıcalıklarını kullanarak önce onu oluşturmanız ve gerekli yetkileri vermeniz gerekir. Dolayısıyla süreç şu şekilde olur:

```bash
root# mkdir /usr/local/pgsql
root# chown postgres /usr/local/pgsql
root# su postgres
postgres$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content="initdb, veri dizinin bulunması ve dosyalar içermesi durumunda çalışmaz. Bu, yanlışlıkla mevcut bir kurulumun üzerine yazılmasını önlemek için alınan bir önlemdir." type="info" %}

Veri dizini, veri tabanında depolanan tüm verileri içerdiğinden yetkisiz erişime karşı korunmalıdır. Bu nedenle `initdb`, PostgreSQL kullanıcısı dışındaki herkesin erişim izinlerini iptal eder. İsteğe bağlı olarak grup erişimi izinleri iptal edilmeyebilir . Grup erişimi etkinleştirildiğinde salt okunurdur. Böylece, küme sahibiyle aynı gruptaki ayrıcalıksız bir kullanıcı küme verilerinin yedeğini alabilir veya yalnızca okuma erişimi gerektiren işlemleri gerçekleştirebilir.

Mevcut bir kümede grup erişiminin etkinleştirilmesi veya devre dışı bırakılma işlemi, kümenin kapatılmasını ve PostgreSQL'i yeniden başlatmadan önce tüm dizinlerde ve dosyalarda uygun modun ayarlanmasını gerektirir. Aksi takdirde, veri dizininde karışık modlar bulunabilir. Sadece küme sahibine erişime izin verirken gerekli modlar dizinlerde `0700` ve dosyalarda `0600` şeklindedir. Kümelerin grup tarafından okunmasına izin vermek için uygun modlar, dizinlerde `0750` ve dosyalarda `0640`'tır.

Dizin içeriklerini güvene almış olsak da, varsayılan istemci kimlik doğrulama kurulumu yerel kullanıcıların veritabanına bağlanmasına, hatta veritabanı süper kullanıcısı olmasına izin verir. Yerel kullanıcılara güvenmiyorsanız, veritabanı süper kullanıcısına parola atamak için initdb'nin `-W`, `--pwprompt` veya `--pwfile` parametreleri kullanılır. Ayrıca, varsayılan `trust` kimlik doğrulama modunu `-A md5` veya `-A password` ile değiştirin veya `pg_hba.conf` dosyasını üzerinden belirleyin. Bağlantı kısıtlamak için diğer yaklaşımlar ise `peer` kimlik doğrulaması ve dosya sistemi izinleri kullanmaktır. Daha fazla bilgi için [İstemci Kimlik Doğrulaması]("") bölümüne bakın.

`initdb` veritabanı kümesi için varsayılan yerel (locale) ayarı da başlatabilir. Normalde, yerel ayarları ortamdan alır ve bunları başlatılmış veritabanına uygular. Veritabanı için farklı bir yerel ayar belirtilebilir; bununla ilgili daha fazla bilgi için [Locale Support]("") bölümüne. Veritabanı kümesinde kullanılan varsayılan sıralama düzeni initdb tarafından belirlenir ve farklı sıralama düzeni kullanarak yeni veritabanları oluştururken initdb'nin oluşturduğu şablon veritabanlarında kullanılan sıralama düzenidir. drop ve recreate edilmeden değiştirilemez. `C` veya `POSIX` dışındaki yerel ayarları kullanmanın da bir performans etkisi vardır. Bu nedenle ilk seferde bu seçimi doğru yapmak önemlidir.

`initdb` ayrıca veritabanı kümesi için varsayılan karakter seti kodlamasını da ayarlar. Bu, yerel ayarlarla eşleşecek şekilde seçilmelidir. Ayrıntılar için [Localization]("") bölümüne bakın.

Non-C ve non-POSIX yerel ayarları karakter seti sıralaması için işletim sisteminin collation kütüphanesini kullanır. Bu, indekslerde saklanan anahtarların sırasını kontrol eder. Bu nedenle bir küme , anlık görüntü geri yüklemesi ( snapshot restore ), binary streaming replication, farklı işletim sistemi veya işletim sistemi yükseltmesi yoluyla farklı collation kütüphanesine geçemez.

## İkincil Dosya Sistemlerinin Kullanımı

Çoğu kurulum, veritabanı kümelerini, makinenin "root" birimi dışındaki dosya sistemlerinde oluşturur. Böyle bir tercihte ikincil birimin en üst dizinini (mount point) veri dizini olarak kullanmayı denemeniz pek tavsiye edilmez. Uygun tercih, PostgreSQL kullanıcısının sahip olduğu bağlama noktası dizini ( mount-point directory ) içinde oluşturacağınız dizinde veri dizinini oluşturmaktır. Bu şekilde `pg_upgrade` gibi işlemlerde izin sorunları önlenir.

## Dosya Sistemleri

POSIX semantiğine sahip her dosya sistemi PostgreSQL için kullanılabilir. Kullanıcılar; satıcı desteği, performans ve aşinalık gibi çeşitli nedenlerle farklı dosya sistemlerini tercih eder. Diğer tüm şeyler eşit durumda yalnızca dosya sistemlerini değiştirmekten veya dosya sistemi konfigürasyon değişiklikleri yaparak büyük performans veya davranış değişiklikleri beklenmemelidir.

### NFS

PostgreSQL veri dizinini depolamak için NFS dosya sistemi kullanılabilir. PostgreSQL NFS'nin yerel olarak bağlı sürücüler gibi davrandığını varsayar ve NFS dosya sistemleri için özel bir şey yapmaz. PostgreSQL, dosya kilitleme ( file locking ) gibi NFS üzerinde standart dışı davranışa sahip olduğu bilinen herhangi bir özelliği kullanmaz.

{% include links.html %}
