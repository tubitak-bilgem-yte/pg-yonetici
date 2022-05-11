---
title: "repmgr"
parent: Yüksek Erişilebilirlik
layout: default
nav_order: 1
---

## repmgr

- 2nd Quadrant geliştiriyor, GPL
- Streaming Replication yönetim aracı
- Konsol uygulaması + (isteğe bağlı) repmgrd servisi + PostgreSQL eklentisi
- Standby klonlama
- Monitoring ve manual/otomatik failover

### Kavramlar

- replication cluster: Streaming replikasyon bağlantısı içinde olan PostgreSQL sunucular kümesi
- node: Kümedeki her bir sunucu
- upstream node: Replike olunan sunucu. Çoğunlukla Primary sunucudur, standby → standby replikasyonu durumunda ise standby sunucu da olabilir.
- failover: Primary sunucu düştüğünde uygun bir stanby sunucunun Primary’lığa terfi ettirilmesi. repmgrd servisi otomatik failoverı destekler.
- switchover: Bakım vb için manuel olarak Primary sunucunun offline’a çekilip stanby sunucunun Primary’lığa terfi ettirilmesi. repmgr komutu ile yapılır.
- fencing: Failover sonrası eski Primary’ın geri geldiğinde Primary olmaya çalışmaması için yapılan şey. Eski Primary uygulamalardan izole edilmesi = fenced off
- witness server: Birden fazla stanby sunucunun olduğu durumda failover’da hangisinin terfi edeceğine dair oy veren sunucu. Replike olmaz, sadece replikasyon meta verisini tutar.

### Parçalar

RepMgr Komutu:

- Standby sunucu kurulumu
- Standby → Primary sunucu haline getirme
- Standby → Primary sunucu arasında geçişler
- Kümede yer alan sunucuların durumunu gösterme

RepMgrd Servisi:

- Replikasyon performansını izleme ve kaydetme
- Primary’a erişilemediğini farkedip otomatik failover gerçekleştirme
- Kümede çeşitli oluşan olaylarda uyarı üretebilme (e-posta, komut çalıştırma, vs)

### Veriler

Tablolar:

- repmgr.events: Olay kayıtları
- repmgr.nodes: Kümedeki her sunucunun bağlantı ve durum bilgisi
- repmgr.monitoring_history: repmgrd’nin topladığı izleme bilgileri

Viewlar:

- repmgr.show_nodes: repmgr.nodes temelli, ek olarak upstream sunucu bilgisini gösterir
- repmgr.replication_status: her bir standby’ın durumunu gösterir

### repmgr Kurulum

Kurulum kümedeki tüm sunucularda ayrı ayrı yapılır. Repmgr paket deposu eklenmesi,

```text
# curl https://dl.2ndquadrant.com/default/release/get/11/rpm | sudo bash
```

PostgreSQL majör sürümüne göre uygun repmgr paketi kurulur:

```text
# yum install repmgr11 --disablerepo=pgdg11
```

Kümedeki tüm sunucuların postgres kullanıcıları arasında SSH ile parolasız bağlantı kurulabilmesi sağlanır. Kümedeki tüm sunucular arası parolasız SSH bağlantısı olmasının gerekmesinin nedeni, kümede ileride başka bir sunucunun primary olabilmesi ve kümedeki her sunucudan kümenin yönetilebilmesi.

Primary sunucuda PostgreSQL’in dışarıya açık olduğu (listen_addresses = '*') ve güvenlik duvarında PostgreSQL portuna izin verildiği kontrol edilir. Primary’da replikasyon ayarları yapılır.

```text
# vim /var/lib/pgsql/11/data/postgresql.conf
max_wal_senders = 10
wal_level = 'replica'
hot_standby = on
archive_mode = on
archive_command = 'test ! -f /srv/pgdata/WAL_Archive/%f && cp %p /srv/pgdata/WAL_Archive/%f'


# mkdir -p /srv/pgdata/WAL_Archive && chown -R postgres:postgres /srv/pgdata/WAL_Archive
```

Standby olacak sunucularda sadece PostgreSQL ve repmgr’nin kurulmuş olması, ama PostgreSQL’in veri dizininin boş olması (initialize edilmemiş olması) gerekir.

Primary sunucuda repmgr kullanıcısı ve veritabanı oluşturulur:

```text
$ createuser -s repmgr
$ createdb repmgr -O repmgr
$ psql -c 'ALTER ROLE repmgr SET search_path TO repmgr, "$user", public;'
```

*pg_hba.conf* ayar dosyasına şu satırlar eklenir:

```text
local   replication   repmgr                              trust
host    replication   repmgr      127.0.0.1/32            trust
host    replication   repmgr      192.168.56.0/24          trust
local   repmgr        repmgr                              trust
host    repmgr        repmgr      127.0.0.1/32            trust
host    repmgr        repmgr      192.168.56.0/24          trust
```

- repmgr kullanıcısının superuser olması zorunlu olmasa da, kolaylık açısından böyle oluşturuyoruz.
- Kümedeki her bir sunucudaki pg_hba.conf’a bu ayarları uygulanmalı (diğer sunucular primary olursa diye).

`/etc/repmgr/11/repmgr.conf` ayar dosyası şöyle düzenlenir:

```text
node_id=1
node_name=node1
conninfo='host=192.168.56.101 user=repmgr dbname=repmgr connect_timeout=2'
data_directory='/var/lib/pgsql/11/data'
```

İlk sunucu primary olarak repmgr’a kaydedilir:

```text
$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf primary register
NOTICE: attempting to install extension "repmgr"
NOTICE: "repmgr" extension successfully installed
NOTICE: primary node record (id: 1) registered
```

- *repmgr.conf*’un dosya sistemi üzerindeki yeri aslında önemli değil. Ancak PostgreSQL data dizininde bulunmaması öneriliyor. Olası bir tekrar oluşturulma durumunda o dizin uçacaktır. repmgr.conf ayar dosyası da o arada kaybedilebilir.
- *node1* yerine istenen isim verilebilir. Ancak "primary", "standby1" gibi isimler iyi bir fikir olmuyor. Bir failover durumunda standby1 isimli bir primary kafa karıştırıcı olabiliyor.

Kümeye sunucunun eklendiği kontrol edilir:

```text
$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf cluster show
 ID | Name  | Role    | Status    | Upstream | Location | Connection string
----+-------+---------+-----------+----------+----------+-----------------------------------------------------------------
 1  | node1 | primary | * running |          | default  | host=192.168.56.101 user=repmgr dbname=repmgr connect_timeout=2
```

repmgr’ın nodes tablosunda kayıt şu şekilde eklenmiş olur:

```text
repmgr=# \x on
Expanded display is on.
repmgr=# SELECT * FROM repmgr.nodes;
-[ RECORD 1 ]----+-------------------------------------------------------
    node_id          | 1
    upstream_node_id |
    active           | t
    node_name        | node1
    type             | primary
    location         | default
    priority         | 100
    conninfo         | host=192.168.56.101 dbname=repmgr user=repmgr connect_timeout=2
    repluser         | repmgr
    slot_name        |
    config_file      | /etc/repmgr/11/repmgr.conf
```

Kümedeki her bir standby sunucu için */etc/repmgr/11/repmgr.conf* ayar dosyası şuna benzer düzenlenir:

```text
node_id=2
node_name=node2
conninfo='host=192.168.56.102 user=repmgr dbname=repmgr connect_timeout=2'
data_directory='/var/lib/pgsql/11/data'
```

Sunucu, master sunucunun verilerini kopyalayarak ilk haline getirilir.

```text
$ /usr/pgsql-11/bin/repmgr -h 192.168.56.101 -U repmgr -d repmgr \
    -f /etc/repmgr/11/repmgr.conf standby clone

NOTICE: destination directory "/var/lib/pgsql/11/data" provided
NOTICE: starting backup (using pg_basebackup)...
HINT: this may take some time; consider using the -c/--fast-checkpoint option
NOTICE: standby clone (using pg_basebackup) complete
NOTICE: you can now start your PostgreSQL server
HINT: for example: pg_ctl -D /var/lib/pgsql/11/data start
HINT: after starting the server, you need to register this standby with "repmgr standby register"
```

Sunucu servisi açılır:

```text
# systemctl start postgresql-11
```

Sunucu standby olarak repmgr’a kaydedilir:

```text
$ /usr/pgsql-11/bin/repmgr -h 192.168.56.101 -U repmgr -d repmgr \
    -f /etc/repmgr/11/repmgr.conf standby register

WARNING: --upstream-node-id not supplied, assuming upstream node is primary (node ID 1)
NOTICE: standby node "node2" (id: 2) successfully registered
```

- Her bir sunucu için için node_id, node_name ve host değerleri değiştirilir.
- `standby clone` komutu normal çalıştırılmadan önce `--dry-run` parametresi ile verilirse bir sorun olup olmadığı kontrol edilebilir.
- `stanby clone` varsayılan olarak pg_basebackup ile verileri kopyalar. Barman kullanmasını söylemek mümkündür.
- `standby register` komutuna `--upstream-node-id` verilmezse varsayılan olarak master (ID:1) node’dan replike olur, --upstream-node-id verilerek bir standby’dan replike olması da sağlanabilir (`=cascading replication`)

Kümeye sunucunun eklendiği kontrol edilir:

```text
$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf cluster show
 ID | Name  | Role    | Status    | Upstream | Location | Connection string
----+-------+---------+-----------+----------+----------+-----------------------------------------------------------------
 1  | node1 | primary | * running |          | default  | host=192.168.56.101 user=repmgr dbname=repmgr connect_timeout=2
 2  | node2 | standby |   running | node1    | default  | host=192.168.56.102 user=repmgr dbname=repmgr connect_timeout=2
```

Primary ve Standby’da PostgreSQL komutlarıyla da replikasyon durumu görülebilir.

**Primary'de**:

```text
SELECT * FROM pg_stat_replication;
```

**Standby’da**:

```text
SELECT * FROM pg_stat_wal_receiver;
```

### Otomatik Failover

RepMgrd kullanabilmek için öncelikle PostgreSQL’in bu fonksiyonu yükleyebilmesi gerekir. postgresql.conf ayar dosyasına ekliyoruz:

```text
# vim /var/lib/pgsql/11/data/postgresql.conf
shared_preload_libraries = 'repmgr'

# systemctl restart postgresql-11
```

RepMgrd’lerin PostgreSQL servisini yönetebilmesi için postgres kullanıcılarının PostgreSQL servisi açma/kapama gibi komutları parolasız sudo ile çalıştırabilme izninin olması sağlanır.

```text
# vim /etc/sudoers.d/postgres
Defaults:postgres !requiretty
postgres ALL = NOPASSWD: /usr/bin/systemctl stop postgresql-11, /usr/bin/systemctl start postgresql-11, /usr/bin/systemctl restart postgresql-11, /usr/bin/systemctl reload postgresql-11
```

repmgrd’nin otomatik failover yapabilmesi için aşağıdaki gibi ayarlar ayar dosyasına yazılır:

```text
# vim /etc/repmgr/11/repmgr.conf
failover=automatic
promote_command='/usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf standby promote --log-to-file'
follow_command='/usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf standby follow --log-to-file --upstream-node-id=%n'
reconnect_attempts=3
reconnect_interval=4
service_start_command   = 'sudo systemctl start postgresql-11'
service_stop_command    = 'sudo systemctl stop postgresql-11'
service_restart_command = 'sudo systemctl restart postgresql-11'
service_reload_command  = 'sudo systemctl reload postgresql-11'
log_file='/var/log/repmgr/repmgr.log'
log_status_interval=300
pg_bindir='/usr/pgsql-11/bin/'
```

repmgrd servisini başlatıp açılışa da ekliyoruz:

```text
# systemctl start repmgr11.service
# systemctl enable repmgr11.service
```

RepMgrd sunucularda başlatıldığında düşen loglar "cluster events" komutuyla ya da events tablosuna bakarak görülebilir.

```text
repmgr=# SELECT * from repmgr.events ;
 node_id |      event       | successful |        event_timestamp        |                                        details
---------+------------------+------------+-------------------------------+----------------------------------------------------------------------------------------
       1 | cluster_created  | t          | 2018-05-02 14:32:25.490917+03 |
       1 | primary_register | t          | 2018-05-02 14:32:25.496667+03 |
       2 | standby_clone    | t          | 2018-05-02 14:47:38.181777+03 | cloned from host "192.168.56.101", port 5432; backup method: pg_basebackup; --force: N
       2 | standby_register | t          | 2018-05-02 14:55:47.844957+03 | standby registration succeeded
       1 | repmgrd_start    | t          | 2018-05-02 17:20:52.796128+03 | monitoring cluster primary "node1" (node ID: 1)
       2 | repmgrd_start    | t          | 2018-05-02 17:29:43.898148+03 | monitoring connection to upstream node "node1" (node ID: 1)
(8 rows)
```

```text
$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf cluster events --event=repmgrd_start
 Node ID | Name | Event         | OK | Timestamp           | Details
---------+------+---------------+----+---------------------+-----------------------------------------------------------
 2       | node2  | repmgrd_start | t  | 2018-05-02 17:29:43 | monitoring connection to upstream node "node1" (node ID: 1)
 1       | node1  | repmgrd_start | t  | 2018-05-02 17:20:52 | monitoring cluster primary "node1" (node ID: 1)
```

Mevcut master’ı kapatıp olacaklara bakalım.

```text
[root@node1 ~]# systemctl stop postgresql-11
```

RepMgrd 4 saniyede bir 3 kez master’a bağlanma girişiminde bulunur, sonrasında master’ı failed olarak işaretler ve bir standby sunucuyu terfi ettirir.

```text
[root@node2 ~]# su - postgres
Last login: Wed May  2 17:51:02 +03 2018 on pts/0
-bash-4.2$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf cluster events
 Node ID | Name | Event                      | OK | Timestamp           | Details
---------+------+----------------------------+----+---------------------+-----------------------------------------------------------------------
 2       | node2  | repmgrd_reload             | t  | 2018-05-02 18:06:28 | monitoring cluster primary "node2" (node ID: 2)
 2       | node2  | repmgrd_failover_promote   | t  | 2018-05-02 18:06:28 | node 2 promoted to primary; old primary 1 marked as failed
 2       | node2  | standby_promote            | t  | 2018-05-02 18:06:28 | server "node2" (ID: 2) was successfully promoted to primary
```

```text
-bash-4.2$ /usr/pgsql-11/bin/repmgr -f /etc/repmgr/11/repmgr.conf cluster show
 ID | Name | Role    | Status    | Upstream | Location | Connection string
----+------+---------+-----------+----------+----------+-----------------------------------------------------------------
 1  | pg1  | primary | - failed  |          | default  | host=192.168.56.101 user=repmgr dbname=repmgr connect_timeout=2
 2  | pg2  | primary | * running |          | default  | host=192.168.56.102 user=repmgr dbname=repmgr connect_timeout=2
```

- Eski master tekrar geldiğinde otomatik olarak yeni master’dan replike olmaya başlamaz.
- repmgrd tek başına geri gelen master sorunsalını çözmek için yeterli değildir, çift master oluşur.
- Uygulamaların doğru master’a bağlanmaları için RepMgrd’ye terfi komutu olarak pgbouncer’ı da ayarlayan bir komut girilmesi bu konuda çözüm olarak uygulanır (=fencing)

### İzleme

repmgrd sadece izleme amaçlı da kullanılabilir. İzleme tablosunun kullanılması için aşağıdaki ayar yapılmalıdır:

```text
# vim /etc/repmgr/11/repmgr.conf
monitoring_history=true
```

Sonrasında replikasyon durumu, gecikmesi vb bilgiler repmgrd üzerinden gözlenebilir.

```text
repmgr=# select * from repmgr.replication_status;
    -[ RECORD 1 ]-------------+------------------------------
    primary_node_id           | 1
    standby_node_id           | 2
    standby_name              | node2
    node_type                 | standby
    active                    | t
    last_monitor_time         | 2017-08-24 16:28:41.260478+09
    last_wal_primary_location | 0/6D57A00
    last_wal_standby_location | 0/5000000
    replication_lag           | 29 MB
    replication_time_lag      | 00:00:11.736163
    apply_lag                 | 15 MB
    communication_time_lag    | 00:00:01.365643
```

### repmgrd - Witness Server

- Birden fazla standby sunucu olduğu durumda ağ bölünmeleri gibi sorunların yaşanmasına karşılık şahitlik eden bir sunucu önlemi alınmıştır.
- Bir failover yaşandığında standby sunucudaki RepMgrd, kendiyle aynı ağda şahit sunucuyu görmüyorsa ağın kalanından koptuğunu varsayar ve kendini terfi ettirmez.
- Eğer stanby sunucu şahit sunucuyu görüyorsa terfi işlemi yapılıp sunucu master olur.
- Şahit sunucu kullanılmazsa split-brain’i önlemek için replication cluster’ın en az 3 elemanının olması (bölünme olursa 2 eleman kalan tarafta master seçimi olması) gerekir.

{% include links.html %}
