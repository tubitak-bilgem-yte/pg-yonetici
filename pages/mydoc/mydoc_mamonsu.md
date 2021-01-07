---
title: "Mamonsu ile İzleme"
tags: [PostgreSQL]
keywords: postgres, Mamonsu 
last_updated: January 7, 2021
sidebar: mydoc_sidebar
permalink: mydoc_mamonsu.html
folder: mydoc
---


## Mamonsu ile İzleme

Zabbix sistem monitoring aracına postgresql ve üzerinde çalıştığı sistem hakkında veriler aktararak çalışır. Ayrıca genel tuning ve reporting özellikleri vardır.

Kendi başına çalışan bir araç olduğundan extension yazarak, özel sorgular yazarak monitör ettiği alanlar artırılabilir. Varsayılan kurulumda 200e yakın parametreyi kontrol eder ve zabbix sayesinde bunların trendlerini incelememizi ve bağlantılı parametreleri karşılaştırmamıza yardımcı olur.

Centos için bu [adrese](https://packagecloud.io/postgrespro/mamonsu/install#bash-rpm) giderek, scripti sunucuda çalıştırın. Bu script mamonsu repoyu kuracaktır.
Bundan sonrasında:

```bash
yum install mamonsu
```

diyerek mamonsuyu kurabilirsiniz.

Repo çalışmazsa github repodaki rpm kurulum yöntemini terchi edilir:

```bash
# rpm build araçlarını kuruyoruz.
yum install make rpm-build python2-devel python-setuptools

wget https://github.com/postgrespro/mamonsu/archive/2.3.4.tar.gz

tar xzf 2.3.4.tar.gz && cd mamonsu-2.3.4 && make rpm
 
 
# o sunucuya kuracaksak
rpm -i mamonsu*.rpm
```

Kurduktan sonra:

```bahs
chkconfig mamonsu on
```

Yaparak başlangıçta servis olarak çalışmasını sağlıyorsunuz.

Aşağıdaki klasöre gidin ve o 2 adet scripti indirin. Bunlar ek sorgular gönderecektir.

```bash
cd /etc/mamonsu/plugins 
# transaction zamanlarıyla ilgili temel bilgiler
wget https://raw.githubusercontent.com/edib/mamonsu-plugins/master/postgresql/transactions_time.py
# en büyük tabloları bulmak için
wget https://raw.githubusercontent.com/postgrespro/mamonsu/master/examples/biggest_tables.py
```

klasör içerisinde bu dosyalar oluşacaktır. Sonrasıdan postgres kullanıcısına geçerek aşağıdaki işlemi gerçekleştiriyoruz. Bu işlem mamonsunun kendi dbsine erişerek monitor datasını tutmasını sağlamaktadır.

```bash
createdb mamonsu
createuser mamonsu
#mamonsu db oluşturuyoruz.
mamonsu bootstrap -U postgres -d mamonsu
```

Aşağıdaki dosyanın içeriğini ve alanlarını bu şekilde değiştirin. adres ve client kısmına uygun dnsname ya da ip yi yazmanız gerekmektedir. `vi /etc/mamonsu/agent.conf`

```bash
[zabbix]
; zabbix server address
address = zabbixip
; configured 'Host name' of client in zabbix (aynı olmak zorunda)
client = clientadi 
 
[postgres]
enabled = True
host = auto
user = mamonsu
database = mamonsu
port = 5432
query_timeout = 10
 
[system]
enabled = True
 
[log]
file = /var/log/mamonsu/agent.log
level = INFO
 
[plugins]
directory = /etc/mamonsu/plugins
```

Postgres bağlantısı yukarıdaki gibi çalışması için local sockete trust pg_hba.conf içerisinde trust yetkisi verilmelidir. Commentsiz olan satırların en üstüne konur. `vi /postgresql/config/path/pg_hba.conf`

```bash
local       all         mamonsu        trust
```

zabbix template'inin varolan pluginlere göre oluşturulması gerekir. Bu komut template.xml adında bir dosya üretecektir. Bunu zabbixe template olarak eklemek gerekmektedir.

```bash
mamonsu export template template.xml --add-plugins /etc/mamonsu/plugins
# ya da 
wget https://raw.githubusercontent.com/postgrespro/mamonsu/master/packaging/conf/template.xml
# ya da 
cp /usr/share/mamonsu/template.xml .
```

Bu template.xml zabbixin kullanabileceği template'tir artık.

```bash
# komut satırından 
mamonsu zabbix template export /usr/share/mamonsu/template.xml --url=http://zabbix_url/ --user=Admin --password=zabbix
# ya da dosyayı browser üzerinden upload ederiz.
```

Sonrasında servisi restart ediyoruz.

```bash
/etc/init.d/mamonsu start
```

Zabbixte bu kurulum için host tanımı yapıyoruz. config dosyasında "clientadi" olarak girilmiş değeri hostname olarak bu host tanımında kullanıyoruz. yoksa veri gelmez.

mamonsu bu işlemlerde başka 2 konuda faydalı olacaktır. Parametrelerine de `mamonsu --help` ile bakılabilir.

```bash
# tune edilebilecek alanları tune eder. Dry-run olmadan kullanırsanız gider dosyada değişiklik yapar.
mamonsu tune --dry-run
# system ve db raporu verir.
mamonsu report
```

{% include links.html %}
