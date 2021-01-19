---
title: "PostgreSQL Otomatik Failover Çözümleri"
keywords: postgresql
last_updated: January 4, 2021
sidebar: mydoc_sidebar
permalink: mydoc_postgresql_ha.html
folder: mydoc
---

## PostgreSQL Otomatik Failover Çözümleri

Sağlıklı bir veritabanı için düzenli yedekler ve sorunsuz çalışan replikasyonların yanında, veritabanının yaşanacak felaket senaryolarına karşı veritabanının devamlılığı için hazırlıklı olması oldukça önemlidir. Herhangi bir anda sunucunun veya PostgreSQL servisinin kapanması ve gelen taleplerin primary sunucuya ulaşmaması durumunda okuma ve daha da önemlisi yazma işlemlerinin standby düğümler üzerinden devam etmesi gerekir. Bu tip bir yapı Autofailover yapısının mevcut olmasıyla mümkündür.

{% include image.html file="postgresql_ha-1.png" alt="PostgreSQL Otomatik Failover"%}

Autofailover yapısını PostgreSQL üzerine entegre etmek için ücretsiz olarak kullanabileceğiniz açık kaynak çözümlerinden bazıları şunlardır:

- PostgreSQL Automatic Failover [ClusterLabs](https://www.clusterlabs.org/)
- Replication Manager (repmgr) [2ndQuadrant](https://repmgr.org/)
- Patroni [Zalando](https://github.com/zalando/patroni)

Bu araçlar sağlıklı bir cluster yönetimi için otomatik failover, monitoring, replication gibi imkanlar sunar.

### PostgreSQL Automatic Failover (PAF)

PAF, ClusterLabs tarafından geliştirilmiş ve PostgreSQL Lisansı altında lisanslanan açık kaynak bir High Availability (HA) yönetim çözümüdür. Pacemaker ve Corosync tabanlıdır. Sistemdeki hizmetlerde arızayı algılayabilme ve arızalı kaynaktaki yükü başka bir düğüme devretme kabiliyetine sahiptir. PAF, failover sırasında Standby sunucuların yeni Primary sunucuya bağlanması için Standby sunucuları yeniden başlatmak yerine IP address failover kullanır. Çok az manuel müdahaleye ihtiyaç duyarak kaynakların genel sağlığını yönetir.

{% include image.html file="postgresql_ha-2.png" alt="PostgreSQL Otomatik Failover"%}

PAF bileşenlerinde Pacemaker, kaynakla 'Resource Agent'ı kullanarak etkileşime girerek hizmet kaynağını yönetir. Resource agent kaynağın nasıl davranması gerektiğini kontrol etmek ve sonuçlarını Pacemaker'a bildirmekle yükümlüdür. Resource agent'lar uygulamakla sorumlu oldukları işlemleri (start, stop, promote vb), nasıl davranmaları gerektiği ve Pacemaker'ı sonuçlar hakkında bilgilendirmeyi içeren OCF standartlarına uyar. PAF, Perl'de yazılmış PostgreSQL için bir OCF kaynak aracısıdır.

Pacemaker+Corosync yapısı cluster'daki düğümler arasında iletişim kurarak PostgreSQL'in çalışmasını izler. Örneğin, primary sunucudaki bir arıza durumunda Pacemaker önce kurtarmaya çalışır eğer kurtarma şansı yoksa mevcut standby sunucular arasından en uygun olanı yeni primary olarak seçer. Özetle, **pacemaker PAF'a cluster davranışlarını kontrol etme yeteneği sağlarken corosync sunucuların bir cluster olarak iletişim kurmasını sağlar.**

**Kurulum Gereksinimleri**:

- PAF, PostgreSQL sürüm 9.3 ve üstünü destekler.
- CentOS 6 ve 7 üzerinde çeşitli senaryolarda kapsamlı bir şekilde test edilmiştir.
- PAF, primary/standby düğümler oluşturma veya kurulumundan sorumlu değildir. PAF kullanılmadan önce streaming replication hazır olmalıdır.
- PAF, Postgres'in herhangi bir yapılandırmasını düzenlemez. Bu sebeple: standby sunucun hot standby olması ve `standby_mode = on`, `recovery_target_timeline = 'latest'`, `primary_conninfo = <application_name>` gibi paremetrelerin *recovery.conf* dosyasında ayarlanması gerekir.

PAF Yapılandırma parametreleri [](https://clusterlabs.github.io/PAF/configuration.html).

**PAF Artıları**:

- Konfigrasyon ayarları ve PostgreSQL kurulumu konusunda kullanıcıyı serbest bırakır.
- Standby ve Primary sunucularda oluşan hataları tespit edebilir ve primary sunucudaki olası hatada standby sunucular arasından yeni bir primary seçebilir.
- Quorum davranışı PAF'da uygulanabilir.
- Network izolasyonu senaryoları ve kaynaklar için sağlanan start, stop, promote, monitor işlemleriyle tam bir HA yönetim çözümü sunar.
- Bir düğümden herhangi bir düğümün yönetimini sağlayabilen dağıtık bir çözümdür.

**PAF Eksileri**:

- PAF, bir standby'ın recovery konfigrasyon dosyasında bilinmeyen veya var olmayan bir düğümle yanlış yapılandırılıp yapılandırılmadığını algılamaz.
- Pacemaker+Corosync bileşenlerinin UDP üzerinden iletişimi için fazladan bir port (Varsayılan 5405) açılır.
- NAT tabanlı yapılandırmayı desteklemez.
- pg_rewind desteği yoktur.

### Replication Manager (repmgr)

repmgr, 2ndQuadrant tarafından geliştirilmiş zengin özelliklere sahip ve veritabanı yöneticisinin işini kolaylaştıran açık kaynaklı bir cluster yönetim araç paketidir. İki ana işlevi vardır: (streaming) replica cluster'lar kurmak ve yönetmek, manuel/otomatik failover ve monitoring. Bu işlemleri repmgr ve repmgrd araçları ile yapar.

**repmgr**:

- Kullanıcı için çeşitli yönetim görevlerini gerçekleştirmeyi sağlayan bir komut satırı yardımcı programıdır.
- Standby sunucular kurmayı, standby sunucuyu primary'e yükseltmeyi, switchover operasyonları yapmayı ve PostgreSQL cluster'ın durumunu izlemeyi sağlar.

**repmgrd**:

- PostgreSQL cluster'ını aktif olarak izler ve durumuna göre gerekli eylemleri gerçekleştirir.
- Primary düğümdeki arıza durumunda en uygun standby'ı otomatik olarak  yeni primary olarak seçer.
- Replication performansını izleme ve ilgili verileri depolama seçeneği sunar.
- Kullanıcı tarafından sağlanan scriptleri kullanarak kayıtlı olaylar için bildirim sağlar.

repmrg, PostgreSQL cluster içindeki replicaları yönetiminden ve standby sunucuların replikasyon ayarlarından sorumludur. Bunun için, ilk kurulumdan sonra repmgr yapılandırma dosyasında (repmgr.conf) gerekli değişiklikleri yapmanız (her sunucu için) ve `repmgr primary/standby register` komutuyla kaydetmeniz gerekir.

Replication Manager, PostgreSQL uzantılarını kullanarak cluster'la ilgili bilgileri depolamak için cluster veritabanında kendi şemasını oluşturur. Uzantının yüklenmesi ve şemanın oluşturulması repmgr kullanılarak primary sunucunun kaydedilmesi sırasında gerçekleşir. Tüm kurulum gereksinimleri tamamlandığında repmgr yardımcı programı kullanılarak promote, monitoring, switchover  vb. manuel yönetim işlemleri yapılabilir. Switchover operasyonları için düğümler arasında şifresiz SSH'nin kurulması gerek.

repmgrd, replikasyon cluster'ındaki her düğümde çalışan bir yönetim ve izleme arka plan programıdır. Failover, standby'ların güncelleştirme ve yeni Primary öğeyi takip etme işlemlerini otomatikleştir. Standby sunucunun durumu hakkında monitoring işlemleri yapabilir. repmgrd'yi kullanabilmek için PostgreSQL sunucusunu başlatırken 'repmgr' kütüphanesi postgresql.conf dosyasındaki `shared_preload_libraries` yapılandırma parametresinde belirtilmelidir. Ayrıca repmgr.conf dosyasında  `failover=automatic` ve monitoring'i aktif etmek için `monitoring_history=yes` olarak ayarlanmalıdır. Gerekli ayarlamalar yapıldıktan sonra repmgrd cluster'ı aktif olarak izlemeye başlar. Primary düğümde herhangi bir hata varsa, birden çok kez yeniden bağlanmaya çalışır. Primary bağlantı girişimleri başarısız olduğunda en uygun Standby repmgrd tarafından yeni Primary olarak seçilir.

repmgr ayrıca olay bildirimlerini de destekler. Önceden tanımlanmış olayların her oluşumu repmgr.events tablosunda saklanır. repmgr, olay bildirimlerini kullanıcı tarafından tanımlanan bir programa veya script dosyasına iletebilir böylece e-posta gönderme veya herhangi bir uyarıyı tetiklemek işlemlerinde kullanılabilir. Bu işlem *repmgr.conf* dosyasında `event_notification_command` parametresi ile yapılandırılır.

{% include image.html file="postgresql_ha-3.png" alt="PostgreSQL Otomatik Failover"%}

repmgr [split brain](https://repmgr.org/docs/5.0/repmgrd-network-split.html) senaryolarını *repmgr.conf* dosyasındaki `location` paremetresiyle ele alır. Her düğüm hangi verimerkezinde bulunduğunu location paremetresi ile vermelidir. Failover durumda, repmgrd geçerli primary düğüm ile aynı konumda herhangi bir sunucu görünür olup olmadığını kontrol eder. Değilse, repmgrd bir ağ kesintisi olduğunu varsayar ve başka bir konumda herhangi bir düğümü promote etmez (primary görünür hale dönene kadar [degraded monitoring](https://repmgr.org/docs/5.0/repmgrd-degraded-monitoring.html) moduna girer).

repmgr, birden fazla standby sunucunun var olduğu failover durumların yeni bir primary sunucunun belirlenmesine yardımcı olmak için "witness server" özelliğini sağlar. [Witness server](https://repmgr.org/docs/5.0/repmgr-concepts.html#WITNESS-SERVER), repmgr meta veri şemasının bir kopyasını içermesine rağmen replication cluster'ın bir parçası değildir. Witness server'ın amacı, replication cluster'daki  sunucuların birden fazla konuma ayrıldığı failover senaryosunda belirleyici rolü oynamaktır. Konumlar arasındaki bağlantının kesilmesi durumunda, witness server konumdaki bir sunucunun primary sunucuya yükseltilip yükseltilmeyeceğine karar verecektir; böylece izole edilmiş bir konumun ağ kesintisini Primary'nin(uzak) başarısızlığı olarak yorumladığı ve Standby'ın (yerel) promote edildiği 'split-brain' durumunu önlemektir.

**Kurulum Gereksinimleri**:

- repmgr, bir veritabanı ve süper kullanıcı ayrıcalıklarına sahip kullanıcı gerektirir.
- PostgreSQL veri dizininin dışında bulunan yapılandırma dosyalarını kopyalamak ve/veya switchover işlevini test için her iki sunucu arasında parolasız SSH bağlantısının sağlanması ve rsync yüklü olması gerekir.
- start, stop, reload ve restart işlemleri için kullanılan pg_ctl (varsayılan olarak repmgr tarafından kullanılır) dışında hizmet tabanlı komutlar kullanmayı düşünüyorsanız repmgr yapılandırma dosyasında (repmgr.conf) bunları belirlemelisiniz.
- repmgr yapılandırma dosyasında gerekli olan temel yapılandırma parametreleri:

{% include callout.html content=" **`node_id (int)`**: Düğümü tanımlayan sıfırdan büyük benzersiz bir tam sayı." type="primary" %}

{% include callout.html content=" **`node_name (string)`**: Karışıklığı önlemek için sunucunun hostname'i veya sunucuyla açık bir şekilde ilişkili başka bir tanımlayıcıyı kullanan rasgele (ancak benzersiz) bir string." type="primary" %}

{% include callout.html content=" **`conninfo (string)`**: Veritabanı bağlantı bilgileri." type="primary" %}

{% include callout.html content=" **`data_directory (string)`**: Düğümün veri dizini. Bu, veri dizini belirlemenin başka bir yolu olmadığı ve PostgreSQL instance'ının çalışmadığı durumlarda repmgr tarafından kullanılır." type="primary" %}

**repmgr Artıları**:

- Repmgr, primary ve standby düğümler kurmaya ve replicaları yapılandırmaya yardımcı programlar sağlar.
- İletişim için ekstra port kullanmaz.
- Kullanıcı için olay  bildirimlerini destekler.
- Primary sunucu arızası durumunda otomatik failover gerçekleştirir.

**repmgr Eksileri**:

- repmgr, standby sunucunun konfigrasyonlarının bilinmeyen veya var olmayan bir düğümle yanlış yapılandırılıp yapılandırılmadığını algılamaz. Primary düğümüne bağlanmadan çalışıyor olsa bile düğüm standby olarak görülecektir.
- PostgreSQL hizmetinin düştüğü bir düğümden başka bir düğümün durumunu alamaz. Bu nedenle, dağıtılmış bir kontrol çözümü sağlamaz.
- Düğümlerin sağlığını iyileştirmekle ilgilenmez.

{% include note.html content=" repmgr, kaynakları yönetebilme kabiliyetine sahip olmadığından tam teşekküllü yüksek kullanılabilirlik yönetimi aracı değildir. Kaynağın uygun durumda olduğundan emin olmak için manuel müdahale gerektirir." %}

### Patroni

Patroni, PostgreSQL HA cluster yapısının kurulumu ve yönetimi için kullanılan python ile yazılmış open source bir araçtır. PostgreSQL cluster kurulumu (bootstrap), cluster replikosyonu ve otomatik failover işlemlerini yerine getirebilir.

- Patroni otomatik failover yapısını sağlamak için bir Distributed Configuration Store (DCS) aracına ihtiyaç duyar. ZooKeeper, etcd, Consul, Kubernetes gibi DCS çözümlerini destekler.
- Patroni, streaming replication dahil olmak üzere PostgreSQL HA kümelerinin uçtan uca kurulum işlemlerini gerçekleştirebilir. Standby düğümü oluşturmanın çeşitli yollarını destekler ve ihtiyaçlarınıza göre özelleştirilebilen bir şablon gibi çalışır. Örneğin replikasyon cluster'ına yeni bir düğüm ekleme işleminde verileri nereden eşitleyeceğinize (hangi düğümden) karar verebilirsiniz.
- REST API'ler (düğümlerin durumu için) ve patronictl adlı komut satırı yardımcı programı aracılığıyla işlevselliğini kullanıcıya sunar.
- Yük dengelemesini gerçekleştirmek için healt check API'lerini kullanarak HAProxy ile entegrasyonu destekler.
- Kullanıcıların bakım faaliyetleri için pause/resume işlevselliğini sağlar.
- Bazı düğümlerin (yalnızca raporlama için olmasını istediğiniz düğümler) master olmasını engelleyebilirsiniz.
- Olay bildirimi desteği sunar.
- [Watchdog](https://patroni.readthedocs.io/en/latest/watchdog.html) özelliği ile daha da tutarlı bir HA framework haline getirilebilir.

{% include image.html file="postgresql_ha-4.png" alt="Patroni Yapısı"%}

Gerekli kurulumlar yapıldıktan sonra bootstrap işlemi için gerekli tüm yapılandırmaların yaml yapılandırma dosyasında belirtilmelidir. Patroni'nin çalıştırılmasıyla başlatılan ilk düğümün DCS'den lider key'ini alarak primary olarak çalışması sağlanır. Cluster kurulumunuz tamamlandığında, Patroni cluster'ı aktif olarak izler ve sağlıklı bir durumda olmasını sağlar. Primary düğüm lider key'ini 30 saniye (varsayılan) bir günceller. Primary düğüm lider key'ini yenilemediğinde, Patroni bir seçimi tetikler ve lider key'ini alan düğüm yeni primary olarak devam eder. Lider key'i DCS aracılığıyla elde edilir ve yalnızca key'i tutan düğüm primary olabilir. Primary düğüm lider key'ini tutmadığı an Patroni tarafından  standby olarak seviyesi düşürülür. Herhangi bir zamanda, sistemde çalışan yalnızca bir primary olabilir. Patroni, bu yapıyla Split Brain senaryolarına karşı çözüm getirmiştir.

**Kurulum Gereksinimleri**:

- Patroni python 2.7 ve üstü ile çalışır.
- DCS ve özel python modülü kurulmalıdır. Test amacıyla DCS, PostgreSQL çalıştıran aynı düğümlere kurulabilir ama production ortamında DCS ayrı düğümlere kurulmalıdır.
- Yaml yapılandırma dosyası aşağıdaki yapılandırma parametreleriyle mevcut olmalıdır:

{% include callout.html content=" **`Global/Universal`**: Bu alan küme için benzersiz olması gereken hostname, kümenin adı (scope) ve DCS'de yapılandırmayı saklama yolu (namespace) gibi yapılandırmaları içerir." type="primary" %}

{% include callout.html content=" **`Log`**: Patroni spesifik log ayarları; log level, format, file_num, file_size vb." type="primary" %}

{% include callout.html content=" **`Bootstrap yapılandırması`**: DCS'ye yazılacak bir küme için genel yapılandırmadır. Standby oluşturma yöntemleri, initdb parametreleri, script dosyaları vb. içerir. Bu parametreler Patroni API'leri yardımıyla veya doğrudan DCS'den değiştirilebilir." type="primary" %}

{% include callout.html content=" **`PostgreSQL`**: Bu bölüm authentication, veri için dizin yolları, listen address gibi PostgreSQL'e özgü parametreleri içerir." type="primary" %}

{% include callout.html content=" **`REST API`**: Bu bölüm REST API'lerle ilgili; listen address, authentication ve SSL gibi Patroni'ye özgü yapılandırmaları içerir." type="primary" %}

{% include callout.html content=" **`Consul, Etcd, Kubernetes, ZooKeeper, Watchdog, Exhibitor`**: Bu alanlar ilgili DCS ye özgü ayarları içerir." type="primary" %}

**Patroni Eksileri**:

- Patroni, standby sunucunun konfigrasyonlarının bilinmeyen veya var olmayan bir düğümle yanlış yapılandırılıp yapılandırılmadığını algılamaz. Düğüm, primary düğümüne bağlanmadan çalışıyor olsa bile standby olarak görülecektir.
- Kullanıcının DCS aracının kurulumunu, yönetimini ve yükseltmesini yönetmesi gerekir.
- Bileşenler iletişimi için birden fazla bağlantı noktasının açık olmasını gerektirir. (Patroni için REST API portu, DCS için minimum 2 port)

### PostgreSQL HA Framework Testleri: PAF vs. repmgr vs. Patroni

PostgreSQL HA yönetimi için kullanılan PostgreSQL Automatic Failover (PAF), Replication Manager (repmgr) ve Patroni araçları  üzerinde yapılmış birkaç testi sizlerle paylaşmak isterim. Bu testler uygulama çalışır durumda ve PostgreSQL veritabanına veri insert edilirken gerçekleştirilmiştir. Uygulama PostgreSQL Java JDBC Driver kullanılarak yazılmıştır.

**Standby Sunucu Testleri**:

{% include image.html file="postgresql_ha-8.png" alt="Standby Sunucu Testleri"%}

**Master/Primary Sunucu Testleri**:

{% include image.html file="postgresql_ha-9.png" alt="Master/Primary Sunucu Testleri"%}

**Network İzolasyon Testleri**:

{% include image.html file="postgresql_ha-10.png" alt="Network İzolasyon Testleri"%}

**Sonuç**:

Dünya bulut teknolojilerini benimsemek için çok hızlı ilerliyorken corosync + pacemaker, repmgr gibi bazı eski HA çözümleri bu çağ için yeterince güncel değil. Şöyle ki, bu çözümler  fail olmuş düğümü cluster'a  otomatik olarak eklemek, ölçeklenebilirlik ve Split Brain konularında esnek ve kullanışlı değiller. Patroni, buluta özgü özellikler, failover ve failback işlemleri için gelişmiş seçeneklerle PostgreSQL için en uygun HA çözümdür. REST APIs, HaProxy integration, Watchdog support, callbacks gibi zengin özelliklere sahip yönetimi ve DCS'yi seçme ve standby oluşturma esnekliği  Patroni'yi PostgreSQL HA yönetimi için en uygun çözüm haline getirmiştir.

**Kaynak**:

[ClusterLabs/PAF](https://github.com/ClusterLabs/PAF)

[repmgr 5.0.0 Documentation](https://repmgr.org/docs/5.0/index.html)

[ScaleGrid](https://scalegrid.io/blog/whats-the-best-postgresql-high-availability-framework-paf-vs-repmgr-vs-patroni-infographic/)

{% include links.html %}
