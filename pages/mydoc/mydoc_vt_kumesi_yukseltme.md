---
title: "PostgreSQL Veritabanı Kümesini Yükseltme"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 24, 2020
sidebar: mydoc_sidebar
permalink: mydoc_vt_kumesi_yukseltme.html
folder: mydoc
---

## PostgreSQL Veritabanı Kümesini Yükseltme

Mevcut PostgreSQL sürüm numaraları ana sürüm ( major ) ve küçük ( minor ) sürüm numarasından oluşur. Örneğin 10.1 sürüm numarasında, 10 ana sürüm numarasını 1 ise küçük sürüm numarasıdır. 10.1, ana sürüm 10'un ilk küçük sürümünü ifade eder.

Küçük sürümlerde asla dahili depolama formatında değişiklik olmaz ve her zaman aynı ana sürüm numarasının önceki ve sonraki küçük sürümleriyle uyumludur. Örneğin sürüm 10.1, sürüm 10.0 ve sürüm 10.6 ile uyumludur. Uyumlu sürümler arasında güncelleme yapmak için sunucu kapalıyken yürütülebilir ( executable ) dosyaları değiştirerek sunucuyu yeniden başlatmanız yeterlidir. Veri dizini değişmeden kalır. Küçük yükseltmeler bu kadar basittir.

PostgreSQL'in ana sürümlerinde dahili veri depolama formatı değişebilir ve bu durum yükseltmeleri karmaşık hale getirir. Verileri yeni bir ana sürüme taşımanın klasik yöntemi, veritabanının dump'ını almak ve yeniden yüklemektir, ancak bu biraz yavaş olabilir. Daha hızlı bir yöntem [`pg_upgrade`](https://www.postgresql.org/docs/13/pgupgrade.html)'dir. Aşağıda bahsedildiği gibi replikasyon yöntemleri de uygulanabilir. (PostgreSQL'in önceden paketlenmiş bir sürümleri ana sürüm yükseltmelerine yardımcı olacak komut dosyaları sağlayabilir.)

Yeni ana sürümler, kullanıcı tarafından görülebilen bazı uyumsuzlukları ortaya çıkarabileceğinden uygulama tarafında değişiklikler gerekebilir. Kullanıcı tarafından görülebilen tüm değişiklikler [sürüm notlarında](https://www.postgresql.org/docs/current/release.html) listelenmiştir. 'Migration' başlıklı bölüme özellikle dikkat edin. Birkaç ana sürümde arasında yükseltme yapıyorsanız araya giren her sürümün, sürüm notlarını okumanız sağlıklı bir yükseltme için önerilir.

Tamamen geçiş yapmadan önce istemci uygulamalarını yeni sürümde test etmek için eski ve yeni sürümlerin eşzamanlı kurulumlarını yapılabilir. Bir PostgreSQL ana sürüm yükseltmesini test ederken aşağıdaki olası değişiklik kategorilerini göz önünde bulundurun:

{% include callout.html content=" **`Administration`** : Her ana sürümde, yöneticilerin sunucuyu izleme ve kontrol etme yetenekleri sıklıkla değişir ve iyileştirilir." type="primary" %}

{% include callout.html content=" **`SQL`** : Yeni SQL komut yeteneklerini içerir. Sürüm notlarında özellikle belirtilmediği sürece davranışta değişiklik olmaz." type="primary" %}

{% include callout.html content=" **`Library API`** : libpq gibi kütüphaneler sürüm notlarında belirtilmediği sürece yalnızca yeni işlevler ekler." type="primary" %}

{% include callout.html content=" **`System Catalogs`** : Sistem kataloğu değişiklikleri genellikle veritabanı yönetim araçlarını etkiler." type="primary" %}

{% include callout.html content=" **`Server C-language API`** : Backend işlevi API'sindeki değişiklikleri içerir. Bu değişiklikler backend işlevlerine başvuran kodu etkiler." type="primary" %}

### pg_dumpall Yaklaşımı ile Yükseltme

Yükseltme yöntemlerinden biri, PostgreSQL ana sürümünde verilerin dump'ın alıp ve başka sürümde yeniden yüklemektir. Bu işlem için [`pg_dumpall`](mydoc_postgresql_yedekleme.html#pg_dumpall) logical yedekleme aracı kullanılır. PostgreSQL'in uyumsuz bir sürümüne sahip bir veri dizinini kullanmanızı engelleyen kontroller vardır. Bu nedenle bir veri dizininde yanlış sunucu sürümünü başlatmaya çalışmak büyük bir zarara sebep olmaz.

PostgreSQL'in yeni sürümündeki [`pg_dump`](mydoc_postgresql_yedekleme.html#pg_dump-ile-yedekleme) ve `pg_dumpall` programlarını kullanmanız önerilir. Dump programlarının mevcut sürümleri, herhangi bir sunucu sürümünden 7.0 sürüm kadar geriden veri okuyabilir.

{% include warning.html content=" Aşağıdaki talimatlar mevcut kurulumunuzun `/usr/local/pgsql` dizini altında olduğunu ve veri alanının `/usr/local/pgsql/data` içinde olduğunu varsayar. Gerekli durumlarda yollarınızı uygun şekilde değiştirin."%}

**1.** Yedekleme yapıyorsanız, veritabanınızın değişiklikler olmadığından emin olun. Bu durum yedeklemenin bütünlüğünü etkilemez, ancak değiştirilen veriler yedeklemeye dahil edilmeyecektir. Çözüm olarak, sizin dışınızdaki erişimleri engellemek için `/usr/local/pgsql/data/pg_hba.conf` dosyasındaki izinler düzenlenebilir.

Veritabanı kurulumunu yedeklemek için şunu yazın:

```bash
pg_dumpall > outputfile
```

Yedeklemeyi yapmak için, şu anda çalıştırmakta olduğunuz sürümde `pg_dumpall` komutunu kullanabilirsiniz. En iyi sonuçlar için PostgreSQL'in en son sürümündeki `pg_dumpall` aracını kullanmayınız önerilir. Güncel sürüm eski sürümlere göre hata düzeltmeleri ve iyileştirmeler içerir.

**2.** Eski sunucuyu kapatın:

```shell
pg_ctl stop
```

**3.** Yedeklemeden geri yükleme yapıyorsanız, sürüme özgü değilse eski kurulum dizinini yeniden adlandırın veya silin. Sorun yaşarsanız ve eskisine geri dönmeniz gerekirse diye, dizini silmek yerine yeniden adlandırmak önerilir. Dizinin önemli miktarda disk alanı kaplayabileceğini unutmayın. Dizini yeniden adlandırmak için aşağıdaki gibi bir komut kullanın.

```shell
mv /usr/local/pgsql /usr/local/pgsql.old
```

**4.** PostgreSQL'in yeni sürümünü kurun.

**5.** Gerekirse yeni bir veritabanı kümesi oluşturun.

```shell
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
```

**6.** Önceki `pg_hba.conf` ve `postgresql.conf` değişikliklerinizi geri yükleyin.

**7.** Veritabanı sunucusunu başlatın:

```bash
/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
```

**8.** Son olarak, verilerinizi yedeklemeden geri yükleyin:

```bash
/usr/local/pgsql/bin/psql -d postgres -f outputfile
```

yeni psql'i kullanarak:

Hem eski hem de yeni sunucular farklı bağlantı noktalarında paralel olarak çalıştırılarak en az kesinti süresi elde edilebilir. Bu durumda kullanım:

```bash
pg_dumpall -p 5432 | psql -d postgres -p 5433
```

### pg_upgrade Yaklaşımı ile Yükseltme

`pg_upgrade` modülü, bir kurulumun ana PostgreSQL sürümünden diğerine yerinde geçişi için kullanılır. Yükseltmeler özellikle `--link` modunda dakikalar içinde gerçekleştirilir. Bu yöntem de sunucuyu başlatma / durdurma, initdb'yi çalıştırma gibi yukarıda bahsettiğimiz pg_dumpall'a benzer adımlar gerektirir. [pg_upgrade](https://www.postgresql.org/docs/current/pgupgrade.html) dokümantasyonunda gerekli adımlar özetlenmiştir.

### Replikasyon Yaklaşımı ile Yükseltme

Güncellenmiş PostgreSQL sürümünden, logical replikasyon yöntemi ile bir standby sunucu oluşturulabilir. Logical replikasyon, PostgreSQL'in farklı major sürümleri arasında replikasyonu destekler. Standby aynı bilgisayarda veya farklı bir bilgisayarda olabilir. Primary sunucu ile senkronize olduktan sonra (PostgreSQL'in eski sürümünü çalıştıran) ana sunucular değiştirilerek standby ana sunucu yapılabilir ve alt sürümdeki veritabanı kapatılabilir. Böyle bir değişim, yükseltme işleminin birkaç saniyelik kesinti süresinde başarılı şekilde tamamlanmasını sağlar.

{% include tip.html content=" Bu yükseltme yaklaşımı yerleşik logical replikasyon olanaklarının yanı sıra pglogical, Slony, Londiste ve Bucardo gibi harici logical replikasyon araçları kullanılarak da gerçekleştirilebilir."%}

### Minör Sürüm Yükseltme

Sadece çalışabilir dosyalar değişir ve PostgreSQL yeniden başlatılır. Paket yönetim sistemi ile:

```shell
yum update postgresql11*
systemctl restart postgresql-11
```

### Majör Sürüm Yükseltme

```shell
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Yeni PostgreSQL sürümü kurulur:

```shell
yum install postgresql12-server postgresql12 postgresql12-contrib
```

Yeni sürümün ilklendirmesi yapılır:

```shell
/usr/pgsql-12/bin/postgresql-12-setup initdb
```

Eski PostgreSQL sürümü durdurulur ve açılıştan kaldırılır:

```shell
systemctl stop postgresql-11
systemctl disable postgresql-11
```

Eski sürümün veritabanı yeni sürümün veritabanına aktarılır:

```shell
su - postgres
$ /usr/pgsql-12/bin/pg_upgrade -b /usr/pgsql-11/bin \
    -B /usr/pgsql-12/bin -d /var/lib/pgsql/11/data \
    -D /var/lib/pgsql/12/data
$ exit
```

Öntanımlıları değiştirilen veritabanı ayar dosyaları (*postgresql.conf*, *pg_hba.conf*, vs) karşılaştırılarak elle yeni sürüme aktarılır. Yeni sürüm PostgreSQL başlatılır ve servis açılışta otomatik çalışacak biçimde ayarlanır:

```shell
systemctl start postgresql-12
systemctl enable postgresql-12
```

Veritabanının aktarımı sonrasında sistemin istatistiklerinin oluşturulması için `ANALYZE` çalıştırılır:

```shell
su - postgres
$ ./analyze_new_cluster.sh
$ exit
```

Eski veri dizini ve eski PostgreSQL paketleri kaldırılabilir (ya da yedek olarak saklanabilir):

```shell
rm -rf /var/lib/pgsql/11/data/*
yum remove postgresql11-server postgresql11 postgresql11-contrib
```

{% include links.html %}
