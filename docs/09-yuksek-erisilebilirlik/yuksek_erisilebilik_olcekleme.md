---
title: "PostgreSQL Yüksek Erişilebilirlik ve Ölçekleme"
parent: Yüksek Erişilebilirlik
layout: default
nav_order: 1
---

## Yüksek Erişilebilirlik ve Ölçeklemeye Giriş

Temel hedef normalden daha yüksek bir erişilebilirlik sağlamaktır. **Normal** tek bir cihaz/yazılımla hizmeti, **yüksek** ise o cihazda sorun olduğunda başka birinin devreye girmesi ve kesinti olmamasını ifade eder.

**Temel İlkeler**:

- Sistemde tek bir noktadan kaynaklı sorunları ( SPoF, Single Point of Failure ) engelleme
- SPoF’ları engellemek için yapılan geçiş noktalarına da çözüm üretilmesi
- Sorunların gerçekleştikçe farkedilmesi

**Sorun Nedir?**:

- Sistemin yavaş yanıt vermesi
- Sistemin yanıt ver(e)memesi
- Sistemin yanlış çalışması

**Sorunun Kaynakları**:

- Donanım arızası
- Donanım kaynağı eklenmesi
- Sistem kaynağının tükenmesi
- Ağ problemi (omurganın herhangi bir yerinde)
- Sistemin güncellenmesi
- Sisteme çok yük bindiren anlık bir iş yapılması
- İnsan hatası
- Yazılım hatası
- Anlık yük bindiren iş örnekleri: Yedek alma, raporlama sorguları ve benzeri.

### Kabul Edilebilir Kesintiler ve Kayıplar

- Hedefler:
  - Kesinti hiçbir zaman kabul edilemez.
  - Kayıp hiçbir zaman kabul edilemez.
- Gerçekler:
  - Ne zaman, ne kadar servis kesintisi kabul edilebilir?
  - Ne kadar veri kaybı kabul edilebilir?
- Örnekler:
  - TEİAŞ : 31 Mart 2015 günü ülkenin tamamında elektrik kesintisi oldu. Ankara ve İstanbul dahil birçok yerde 12 saati aşan kesinti, ülkede yaşamı durdurdu.
  - İş Bankası: 12 Haziran 2013 öğleden itibaren İş Bankası banka işlemi yapamaz hale geldi. ATM’ler, kredi kartları, banka şubeleri çalışmadı. Gün sonunda o ana kadarki para aktarım işlemleri yapıldı (havale/eft), bir sonraki gün sabah banka hizmet vermeye başladı.
- Her sistem kesintiye uğrayabilir
- Her sistemde veri kaybı yaşanabilir
- Veri kaybedeceğimize kesinti olsun kararı verilebilir (ya da tersi)

**Planlı Kesinti**:

- Yüksek erişilebilir olmayan sistemler için tek çözüm
- Yoğun olmayan bir saatte kesinti (sabahlar olmasın)
- En azından bir gün önceden kesintinin duyurulması
- Kesinti sırasında insancıl bir uyarı gösterilmesi

{% include note.html content="Planlı kesintiler, kullanan kişilere önceden haber verildiği sürece birçok kurumda ciddi bir soruna yol açmayacaktır."%}

Yüksek Erişilebilirlik Hesaplama:

| Erişilebilirlik | Yıllık Kesinti Süresi |
|-------|--------|
| % 99 | 3,65 gün |
| % 99,5 | 1,83 gün |
| % 99,8 | 17,52 saat |
| % 99,9 | 8,76 saat |
| % 99,95 | 4,38 saat |
| % 99,99| 52,56 dakika |

### İdeal Yüksek Erişilebilir Sistem

- Sistemi etkileyecek her yapının aynı kapasitede bir yedeğinin olması (ve otomatik devreye girmesi):
- Çift ağ hattı (biri uydu, biri karasal)
- Çift elektrik hattı (farklı kaynaklardan)
- Çift switch, SAN, sunucu, klima, vs vs…​
- Çift fiziki mekan
- Çiftler arasında otomatik failover

### Yüksek Erişme, Ölçekleme, Yedeklilik?

- Kavramlar birbirine sıkı sıkı bağlı:
  - Yüksek erişme: Bir aksaklığın sistemi etkilememesi
  - Ölçekleme: Mevcut sistem yetmediğinde, bir sunucu daha ekleyip sistemin kapasitesini arttırabilmek.
  - Yedeklilik: Bir sorun olduğunda hazır bir yedeğin elde olması. Kesintide elle de devreye sokulabilir.

Neden Ölçekleme?

- Tek bir sunucuya daha fazla donanım eklenemiyordur
- Tek bir sunucuya daha fazla donanım eklemek için verilecek parayla yeni bir sunucu alınıyordur
- Tüm sistem tek bir sunucuya emanet edilmek istenmiyordur (yedeklilik)
- Performansı iyileştirmek mi ölçeklemek mi?

Kolay ölçeklenebilen sistemlerde, performans iyileştirme ile uğraşmak yerine yeni sistemler eklemek daha masrafsız olabiliyor.

## Yüksek Erişilebilirlik ve Ölçekleme Temelleri

**Yapılması Gereken**:

- Bir sunucuda yazılan verinin diğer sunuculara aktarılması
- Bunun anlık ya da belirli aralıklarla gerçekleştirilmesi
- Bir sunucunun anlık olarak ya da gerektiğinde diğeri yerine işlem yapabilmesi
- İstemcilerin sürekli ya da gerektiğinde diğer sunucuları da sorgulamasının sağlanması
- Birden fazla sunucuda aynı veri değişiyorsa, çakışma olmasının engellenmesi
- Uygulamanın kümede çalışacak biçimde düzenlenmesi
- Saatlerimizi ayarlayalım!

{% include warning.html content="Eğer saatler aynı olmazsa, timestamp karşılaştırmaları hatalı olabilir. Yeni bir veri, eski zannedilip aktarılmayabilir. Daha eski bir veri yenisinin üzerine yazılabilir. NTP servisi ile saatlerin aynılığı sağlanabilir."%}

### Sunucular Arası Veri Aktarımı

- Veri yazan tüm işlemlerin kaydedilmesi
- Diğer sunucuların belirli aralıklarla bu işlemleri sorgulaması
- Diğer sunucuların son sorgulamadan beri kaydedilen işlemleri alması ve kendine uygulaması, eş hale gelmesi
- Sunucular arasında ağ bağlantısı hızlı ve kesintisiz olmalı.
- Sunucu donanımları olabildiğince eş olmalı.

**NOT:**

- Veri yazan işlemler doğrudan SQL sorgusu olarak yazılabileceği gibi, verinin binary farkı olarak da yazılabilir.
- Sunucular arasındaki ağ bağlantılarında kopmalar olursa ya da bant genişliği yeterli gelmezse, veri değiştiren işlemlerin diğer sunuculara aktarımında gecikmeler olur.
- Donanımlar eş olmazsa, bir sunucu diğer bir sunucunun yazdığı hızda veri yazamayabileceğinden, veri değiştiren işlemlerin diğer sunuculara aktarımında gecikmeler olur.
- Birden fazla sunucu veri giriyorsa, veri çakışma riski artar. O sunuculardan okuyan istemcilere eski bilgi gitme riski olur.
- Sunucuya yeni yazılan verilerin kayıtlarının tutulduğu dosyadaki bilgi silinene kadar diğer sunucu gelip o bilgiyi almazsa, kümeden elle müdahale edilene kadar kopabilir.

### Bayat Veri Okuma Riski

#### Veritabanı Katmanı

- Bir sunucuda yazılan veri, diğerine anlık olarak aktarılsa da bir gecikme payı bulunuyor.
- Diğer bir sunucuyu okuyan bir istemci bayat (dirty) bir okuma yapabilir.
- Uygulama açısından bu risk önemsiz olabilir.
- Veritabanı servisine diğer sunuculara da yazıldığından emin ol, ancak ondan sonra uygulamaya "işlem başarılı" döndür denebilir.

#### Uygulama Katmanı

- Verinin uygulama tarafında sürümlendirilmesi.
- Veri tekrar yazılmadan önce, okunan sürümle üzerine yazılmak üzere olan sürümün karşılaştırılması.
- Sürümlerin tutmaması durumunda kullanıcıya çakışma olduğu bilgisini iletip, eski ve yeni bilgiyi göstererek çözüm yöntemi sorması.

{% include note.html content="Veritabanında her bir değiştirilen satır için bir sürüm numarası tutulur ve güncelleme sırasında arttırılır."%}

### Veri Yazma Çakışma Riski

#### Veritabanı Katmanı

- Unique kolonlar için sayı aralığı verilmesi ya da UUID kullanılması
- Birden fazla sunucuda aynı veri güncellenirse, daha güncel verinin daha eski olanın üzerine yazması
- Birden fazla sunucuda aynı veri güncellenirse, servisin durdurulması ve dükkanın elle müdahaleye kadar kapatılması

**NOT**:

- Veri girerken otomatik sayı üretilen unique kolonlar (ör: id) bulunabiliyor. Unique kolonlar için sayı aralığı verilmemesi durumunda, ikinci sunucu birbirinden habersiz aynı sayı aralığını üretebilir. Çakışma olmaması için veri girerken birden fazla sunucunun aynı sayıyı üretmesinin engellenmesi gerekiyor.

- UUID, unique bir değer üretilmesini garantiliyor. Ancak büyük bir değer olduğu için hem çok yer kaplıyor, hem de karşılaştırma nedeniyle veritabanı performansını düşürüyor.

#### Uygulama Katmanı

- Bayat veri okuma riski bölümündeki adımlarının uygulanması
- Unique kolonlar için node_id gibi farklı çözümlere yönelinmesi
- Kayıt hiç güncellenmemesi, ters yeni kayıt girerek toplamda veri silmesi ya da güncellemesi
- Uygulamada çift kayıt kontrolü yapılması ve gerektiğinde iki kaydı kullanıcının birleştirebilmesinin sağlanması

**NOT**:

- Kayıt hiç güncellememe örneği bir muhasebe yazılımı üzerinden verilebilir. Yapılan bir işlemin silinmesi yerine, ters kayıt girişi ile eski değerli aynı miktarda bir işlem yapılarak bu işlem toplam alındığında silinmiş olur.
- Veri teknik olarak çakışmasa da, pratikte aynı içeriğe sahip (farklı id’lerde) iki veri oluşabilir ve o veriye bağlı başka veriler girilebilir.
- Otomatik üretilen unique kolonlar için her bir sunucu için ayrı bir de node_id değeri bir kolon olarak eklenebilir. Uygulama tarafında otomatik üretilen id ve node_id’nin beraber bir bileşik birincil anahtar (composite primary key) olarak kullanılabilir.
- Otomatik üretilen unique kolonlar için alternatif bir yöntem de Hi/Lo algoritması kullanılabilir:
  - Node_id olarak gelen değer hi bit’e (yüksek basamağa) konurken, otomatik üretilen değer lo bit’e (düşük basamağa) konabilir.
  - Veritabanından otomatik istenen değer hi bit’e konup, uygulama kendi sayacını tutup onu da lo bit’e yerleştirebilir.
  - İşi unique değer vermek olan, in-memory çalışan ve kendinden küme desteği olan başka bir küme sisteminden bu değer alınabilir (Redis, Hazelcast, Infinispan, vs).

## Yüksek Erişilebilirlik ve Ölçekleme Modelleri

### Yatay/Dikey Ölçekleme

- Dikey Ölçekleme
  - Performans iyileştirme
  - Daha çok disk/RAM/CPU ekleme

- Yatay Ölçekleme
  - Daha fazla sunucu ekleme
  - Okuma/yazma yükünü sunuculara dağıtma
  - Tüm verileri tüm sunuculara kopyalama

### Aktif ve Pasif Kümeleme

- Aktif-Pasif
  - Bir cihaz istekleri karşılar
  - İkinci bir cihaz ilk cihaz düştüğünde ancak devreye girer
  - Düşen cihaz geri geldiğinde pasif kalır
- Aktif-Aktif
  - İki cihaz da tüm istekleri paylaşarak karşılar
  - Bir cihaz düşerse, tüm yük ikinci cihazın üzerine düşer
  - İstemcinin iki cihazı birden sorgulayabilmesi gerekir

### Tek Primary

#### Standby’ler Oluşturulması

- Primary’da Standby IP’si ve kullanıcısı yetkilendirilir
- Standby salt-okunur çalışacak biçimde ayarlanır
- Veritabanı ilk sync işlemini destekliyorsa, Standby’de Primary’ın IP’si ve erişim bilgileri ayarlanır. Sync işlemi zaman içinde gerçekleşir.
- Veritabanı ilk sync desteklemiyorsa,
  - Elle dump alınır ve restore edilir
  - Standby’in başlangıç pozisyonu, Primary’daki dump anına alınır
  - Primary’ın IP’si ve erişim bilgileri ayarlanır.
- İki veritabanı eş hale geldiğinde, Standby düzenli olarak Primary’dan yeni değişen kayıtları alarak kendisine uygular.
- İstenildiği kadar Standby üretilebilir.

#### Standby’in Kullanım Alanları

- Pasif olarak bekletilebilir. Primary’da (aktifte) kesinti olduğunda:
  - Salt-okunur olarak uygulamanın çalışmasını sağlayabilir.
  - Elle ya da otomatik Primary yapılabilir
- Üzerinde okuma sorguları çalıştırılabilir.
- Üzerinden yedek alınabilir.
- Sunucu güncellemelerinin ilk uygulaması yapılabilir.
- Veritabanına ağır yük getiren ama nadir yapılan sorgular gerçekleştirilebilir.

**Not:**

- Primary’a gelen tüm okuma sorguları Standby’e kaydırılması durumunda, Primary üzerinde ciddi bir yük dengelenmiş olacaktır.
- Standby’in kendisi de yedek olmasına karşın, her zaman için "soğuk" alınan bir yedeğe ihtiyaç vardır. Primary’da uygulamasal yapılan bir hatayı Standby birebir uygulayacaktır.
- Güncellemeler ne kadar olsa, sorun çıkarma riski olan uygulamalardır. Önce Standby üzerinde yapılan bir güncelleme, sorun çıkması durumunda kesintiyi en aza indirger.

#### Okuma Sorgularının Dağıtılması

- Veritabanı destekliyorsa, Primary ve Standby’lere gelen okuma sorguları otomatik olarak (diğer) Standby’lere dağıtılır
- Veritabanı desteklemiyorsa,
  - 3.parti bir proxy servisi araya yerleştirilir. Tüm sorgular onun üzerinden geçer, ayrıştırmayı gerçekleştirir.
  - Uygulamada iki ayrı veritabanı bağlantısı kurulur, okuma sorguları ile yazma sorguları uygulamada ayrıştırılarak diğerine yollanır.

**NOT**: Uygulama katmanında sorgu ayrıştırma işlemininin elle yapılması gerekmiyor. Programlama dillerinin ORM katmanı bu işlevi gerçekleştirebiliyor.

#### Standby’lerin Güncel Olma Garantisi

- Primary’dan Standby’lere veri aktarımı hızlı da olsa, anlık olarak aktarılmamış olabilir.
- Primary düşerse veri kaybedilebilir.
- Primary, en az bir Standby’e veri yazılmadıkça uygulamaya "yazma işlemi başarılı" demeyecek biçimde ayarlanabilir.
- Dezavantajı, uygulamanın yazma yanıtı alması yavaşlayacaktır.

**NOT**: 1 değil, n sayıda Standby veriyi almış olsun diye de ayarlanabiliyor. Her eklenen Standby yazma yanıtını biraz daha geciktirebilir.

#### Otomatik Failover - Quorum

- Standby’ler Primary’ı düzenli kontrol eder
- Primary düşünce,
  - Standby’ler toplanır
  - Aralarında en güncel veriye sahip olan Primary olur
  - Quorum salt çoğunluk ile toplanır ve bir Primary seçilir
  - Primary geri gelse de artık o bir Standby’dir
  - Quorum’da salt çoğunluk yoksa, Primary seçilmez ve servis yazmayı durdurur
- Veri kaybı riski!

**NOT**: Primary, "mutlaka en az bir Standby’e veri yazılmış olsun" olarak ayarlanmadıysa, kapanan Primary’da henüz Standby’lerin herhangi birisine aktarılmamış veriler olabilir. Otomatik bir failover, bu verilerin kaybolmasına yol açar. Quorum’un salt çoğunlukla toplanmasının nedeni split brain durumunu engellemek.

#### Otomatik Failover - Split-brain Sorunu

- Pasif olan Standby Primary’a ulaşamıyor (ağ sorunu)
- Primary aslında çalışıyor
- Standby kendini Primary ilan ediyor
- Primary bir taraftan Primary’lık yapmaya devam ediyor

#### Otomatik Failover - Proxy

- Sorguların üzerinden geçtiği proxy, Primary’ın düştüğünü farkeder
- Standby’lerden birinin Primary olmasını sağlar
- Tüm yazma sorgularını ona yönlendirir
- Veri kaybı riski!
- Proxy’nin kendi bir SPoF riski! Çoğaltılmalı.

**NOT**: Proxy’nin kendisinin çoğaltılması için bir aktif-pasif modeli ona uygulanabilir.

#### Otomatik Failover - Standby

- Primary’ın üzerinde çift IP bulunur.
- Uygulama bu IP’lerden birinden sorgular
- Standby’lerden biri Primary’ı düzenli kontrol eder
- Primary’ın indiğini farkeden Standby,
  - Sunucuya erişebiliyorsa uygulamanın sorguladığı IP’yi kapatır
  - IP’yi kendi üzerine çeker
  - Kendini Primary olarak ayarlar
- Primary’ın geri gelmemesi gerekir
- Veri kaybı riski!
- Splitbrain riski!

#### Otomatik Failover - Standby (Verisiz)

- Primary’ın üzerinde çift IP bulunur
- Uygulama bu IP’lerden birinden sorgular
- Data dizini storage’da bulunur
- Standby’da bekleyen sunucu Primary’ı düzenli kontrol eder
- Primary’ın indiğini farkeden standby sunucu,
  - Sunucuya erişebiliyorsa uygulamanın sorguladığı IP’yi kapatır, data dizinini umount eder
  - IP’yi kendi üzerine çeker
  - Data dizinini kendine bağlar
  - Kendini Primary olarak ayarlar
- Primary’ın geri gelmemesi gerekir
- Veri kaybı riski!
- Split-brain riski!

#### Gecikmeli Standby - Yedeklilik

- Kasıtlı olarak işlemleri x saat geriden takip eder
- Uygulamasal hatalara karşı bir ek yedek katmanı
- Son olarak dump yedeğinden daha günceldir
- Gecikmeli Standby üzerine canlı sistemde son yapılan işlemler uygulanarak hızla canlı sistem haline getirilebilir.

**NOT**: Dump’tan geri dönmek büyük veritabanlarında çok uzun sürebilir. Gecikmeli bir Standby’den dönmek (aradaki hatalı işlem ayıklanarak) çok daha kısa sürer.

### Çok Primary

#### Herkes Primary

- Primary’lar sürekli birbirleriyle iletişim halinde olur
- Birinin yazdığı diğerine anında aktarılır
- Tüm Primary’lar tüm veriye sahip olur.
- Tüm Primary’lar hem yazma hem okuma işlemleri yapar
- Primary sayısı arttıkça ciddi bir ağ trafiği oluşacağından, güçlü bir ağ altyapısına ihtiyaç duyar

#### Sharding Primary’lar + Standby’leri

- Çok Primary olur ama Primary’lar aralarında veri bölüşür
- Her Primary’ın kendi Standby’leri olur
- Her bölüm (shard) kendi Primary-Standby kümesi olur
- Her bir bölümün disk ihtiyacı kendi verisi kadar olur

**NOT**: Her bir Primary-Standby kümesinde, Primary-Standby kümelerinin tüm pratikleri uygulanabilir. Standby’lerin bulunması zorunlu değil. Ancak o zaman her bir bölümlenmiş Primary SPoF olur.

#### Genel Araçlar

**Haproxy**: Yük dengeleme

**Keepalived**: Birden fazla sunucu arası ortak IP gezmesi (VRRP)

**uCARP**: Birden fazla sunucu arası ortak IP gezmesi (CARP)

**Pacemaker + Corosync**: Quorum, servislerin tek bir sunucuda çalışması, vs

**DRDB**: Blok düzeyinde dosya sisteminin diskin mirrorlanması. Ağdan çalışan bir RAID-1 gibi.

### Senaryoların Test Edilmesi

- Yüksek erişilme senaryolarına nadiren ihtiyaç duyulur
- "Bir kere" çalışmaması felaket için yeterlidir
- Kesinti yaratmadan senaryoların gerçek anlamda test edilmesi mümkün değil
- Belirli aralıklarla planlı kesinti oluşturularak sistemin çalıştığı kontrol edilmeli

{% include links.html %}
