---
title: "PostgreSQL Veritabanı Kümesini Yükseltme"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 23, 2020
summary: "Bu bölümde veritabanı verilerinizi bir PostgreSQL sürümünden daha yenisine nasıl yükselteceğiniz anlatılmaktadır."
sidebar: mydoc_sidebar
permalink: mydoc_vt_kumesi_yukseltme.html
folder: mydoc
---

## PostgreSQL Veritabanı Kümesini Yükseltme

Mevcut PostgreSQL sürüm numaraları, ana sürüm ( major ) ve bir küçük ( minor ) sürüm numarasından oluşur. Örneğin 10.1 sürüm numarasında, 10 ana sürüm numarasını 1 ise küçük sürüm numarasıdır, bu ana sürüm 10'un ilk küçük sürümünü ifade eder.

Küçük sürümlerde asla dahili depolama formatında değişiklik olmaz ve her zaman aynı ana sürüm numarasının önceki ve sonraki küçük sürümleriyle uyumludur. Örneğin sürüm 10.1, sürüm 10.0 ve sürüm 10.6 ile uyumludur. Uyumlu sürümler arasında güncelleme yapmak için sunucu kapalıyken yürütülebilir dosyaları değiştirmeniz ve sunucuyu yeniden başlatmanız yeterlidir. Veri dizini değişmeden kalır. Küçük yükseltmeler bu kadar basittir.

PostgreSQL'in ana sürümlerinde dahili veri depolama formatı değişebilir ve bu durum yükseltmeleri karmaşık hale getirir. Verileri yeni bir ana sürüme taşımanın geleneksel yöntemi, veritabanının dump'ını almak ve yeniden yüklemektir, ancak bu yavaş olabilir. Daha hızlı bir yöntem [`pg_upgrade`](https://www.postgresql.org/docs/13/pgupgrade.html)'dir. Aşağıda bahsedildiği gibi replication yöntemleri de mevcuttur. (PostgreSQL'in önceden paketlenmiş bir sürümünü kullanıyorsanız, büyük sürüm yükseltmelerine yardımcı olacak komut dosyaları sağlayabilir. Ayrıntılar için paket düzeyindeki belgelere bakın.)

Yeni ana sürümler ayrıca kullanıcı tarafından görülebilen bazı uyumsuzlukları ortaya çıkarabileceğinden uygulama tarafında değişiklikler gerekebilir. Kullanıcı tarafından görülebilen tüm değişiklikler sürüm notlarında [Appendix E](https://www.postgresql.org/docs/current/release.html) listelenmiştir; 'Migration' başlıklı bölüme özellikle dikkat edin. Birkaç ana sürümde arasında yükseltme yapıyorsanız, araya giren her sürüm için sürüm notlarını okuduğunuzdan emin olun.

Dikkatli kullanıcılar, tamamen geçiş yapmadan önce istemci uygulamalarını yeni sürümde test etmek isteyeceklerdir, bu nedenle eski ve yeni sürümlerin eşzamanlı kurulumlarını yapmak genellikle iyi bir fikirdir. Bir PostgreSQL ana sürüm yükseltmesini test ederken aşağıdaki olası değişiklik kategorilerini göz önünde bulundurun:

`Administration`

Yöneticilerin sunucuyu izleme ve kontrol etme yetenekleri, her ana sürümde sıklıkla değişir ve iyileştirilir.

`SQL`

Yeni SQL komut yeteneklerini içerir ve sürüm notlarında özellikle belirtilmediği sürece davranışta değişiklik içermez.

`Library API`

libpq gibi kitaplıklar, sürüm notlarında belirtilmediği sürece yalnızca yeni işlevler ekler.

`System Catalogs`

Sistem kataloğu değişiklikleri genellikle yalnızca veritabanı yönetim araçlarını etkiler.

`Server C-language API`

C programlama dilinde yazılan arka uç ( backend ) işlevi API'sindeki değişiklikleri içerir. Bu tür değişiklikler sunucunun derinliklerinde arka uç işlevlerine başvuran kodu etkiler.

### pg_dumpall Yaklaşımı ile Yükseltme

Yükseltme yöntemlerinden biri, PostgreSQL'in bir ana sürümünde verilerin dump'ın alıp ve başka bir sürümde yeniden yüklemektir. Bu işlem için `pg_dumpall` gibi bir mantıksal yedekleme aracı ( logical backup tool ) kullanmanız gerekir; dosya sistemi düzeyinde yedekleme yöntemleri çalışmayacaktır. (PostgreSQL'in uyumsuz bir sürümüne sahip bir veri dizinini kullanmanızı engelleyen yerinde kontroller vardır, bu nedenle bir veri dizininde yanlış sunucu sürümünü başlatmaya çalışmak büyük bir zarar sebep olmaz.)

Bu programlarda yapılmış olabilecek geliştirmelerden faydalanmak için, PostgreSQL'in yeni sürümünden `pg_dump` ve `pg_dumpall` programlarını kullanmanız önerilir. Dump programlarının mevcut sürümleri, herhangi bir sunucu sürümünden 7.0 sürüm kadar geriden veri okuyabilir.

Bu talimatlar, mevcut kurulumunuzun `/usr/local/pgsql` dizini altında olduğunu ve veri alanının `/usr/local/pgsql/data` içinde olduğunu varsayar. Gerekli durumlarda yollarınızı uygun şekilde değiştirin.

**1.** Yedekleme yapıyorsanız, veritabanınızın güncellenmediğinden emin olun. Bu, yedeklemenin bütünlüğünü etkilemez, ancak değiştirilen veriler yedeklemeye dahil edilmeyecektir. Gerekirse, sizin dışınızda herkesin erişimini engellemek için `/usr/local/pgsql/data/pg_hba.conf` dosyasındaki izinleri düzenleyin. Erişim kontrolü hakkında ek bilgi için [SQL Dump]("") bölümüne bakın.

Veritabanı kurulumunuzu yedeklemek için şunu yazın:

```bash
pg_dumpall > outputfile
```

Yedeklemeyi yapmak için, şu anda çalıştırmakta olduğunuz sürümden `pg_dumpall` komutunu kullanabilirsiniz; Daha fazla ayrıntı için [SQL Dump]("") bölümüne bakın. Ancak en iyi sonuçlar için PostgreSQL'in en son srümündeki `pg_dumpall` komutunu kullanmayınız önerilir, çünkü bu sürüm eski sürümlere göre hata düzeltmeleri ve iyileştirmeler içerecektir. Henüz yeni sürümü yüklemediğiniz için bu tavsiye kendine özgü görünse de, yeni sürümü eski sürüme paralel olarak yüklemeyi planlıyorsanız bu tavsiyeye uymanız önerilir. Bu durumda kurulumu normal şekilde tamamlayabilir ve verileri daha sonra aktarabilirsiniz. Bu aynı zamanda kesinti süresini de azaltacaktır.

**2.** Eski sunucuyu kapatın:

```shell
pg_ctl stop
```

PostgreSQL'in önyükleme ( boot time ) sırasında başlatıldığı sistemlerde muhtemelen aynı şeyi yapacak bir başlangıç ​​dosyası vardır. Örneğin, bir Red Hat Linux sisteminde bunun çalıştığı görülebilir:

```shell
/etc/rc.d/init.d/postgresql stop
```

Ayrıntılı bilgi için **Sunucu Kurulumu ve Çalışması** bölümüne bakın.

**3.** Yedeklemeden geri yüklüyorsanız, sürüme özgü değilse eski kurulum dizinini yeniden adlandırın veya silin. Sorun yaşarsanız ve eskisine geri dönmeniz gerekirse diye, dizini silmek yerine yeniden adlandırmak önerilir. Dizinin önemli miktarda disk alanı kaplayabileceğini unutmayın. Dizini yeniden adlandırmak için aşağıdaki gibi bir komut kullanın (Dizini tek bir birim olarak taşıdığınızdan emin olun, böylece göreli yollar değişmeden kalır.) :

```shell
mv /usr/local/pgsql /usr/local/pgsql.old
```

**4.** PostgreSQL'in yeni sürümünü kurun.

**5.** Gerekirse yeni bir veritabanı kümesi oluşturun. Özel veritabanı kullanıcı hesabına (yükseltme yapıyorsanız zaten sahip olduğunuz) oturum açarken bu komutları yürütmeniz gerektiğini unutmayın.

```shell
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
```

**6.** Önceki `pg_hba.conf` ve `postgresql.conf` değişikliklerinizi geri yükleyin.

**7.** Veritabanı sunucusunu, yine özel veritabanı kullanıcı hesabını kullanarak başlatın:

```bash
/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
```

**8.** Son olarak, verilerinizi yedeklemeden geri yükleyin:

```bash
/usr/local/pgsql/bin/psql -d postgres -f outputfile
```

yeni psql kullanarak:

En az kesinti süresi için yeni sunucu farklı bir dizinde ve hem eski hem de yeni sunucuları farklı bağlantı noktalarında paralel olarak çalıştırarak elde edilebilir. O zaman şöyle bir şey kullanabilirsiniz:

```bash
pg_dumpall -p 5432 | psql -d postgres -p 5433
```

### pg_upgrade Yaklaşımı ile Yükseltme

[`pg_upgrade`](https://www.postgresql.org/docs/current/pgupgrade.html) modülü, bir kurulumun ana PostgreSQL sürümünden diğerine yerinde geçişi için kullanılır. Yükseltmeler, özellikle `--link` modunda dakikalar içinde gerçekleştirilebilir. Sunucuyu başlatma / durdurma, initdb'yi çalıştırma gibi yukarıdaki bahsettiğimiz pg_dumpall'a benzer adımlar gerektirir. [pg_upgrade](https://www.postgresql.org/docs/current/pgupgrade.html) dokümantasyonu gerekli adımları özetler.

### Replication Yaklaşımı ile Yükseltme

PostgreSQL'in güncellenmiş sürümü ile bir standby sunucu oluşturmak için logical replication yöntemlerini kullanmak da mümkündür. Bu mümkündür, çünkü mantıksal çoğaltma ( logical replication ), PostgreSQL'in farklı ana sürümleri arasında eşlemeyi destekler. Standby aynı bilgisayarda veya farklı bir bilgisayarda olabilir. Ana ( Master ) sunucu ile senkronize olduktan sonra (PostgreSQL'in eski sürümünü çalıştıran) ana sunucuları değiştirebilir ve standby'ı ana sunucu yapabilir ve eski veritabanı örneğini kapatabilirsiniz. Böyle bir değişim, yükseltme işleminin yalnızca birkaç saniyelik kesinti süresinde başarılı şekilde tamamlanmasını sağlar.

Bu yükseltme yaklaşımı yerleşik logical replication olanaklarının yanı sıra pglogical, Slony, Londiste ve Bucardo gibi harici logical replication sistemleri kullanılarak gerçekleştirilebilir.

{% include links.html %}
