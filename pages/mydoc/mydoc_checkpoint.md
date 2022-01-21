---
title: "Checkpointer"
tags: [PostgreSQL]
keywords: postgres, checkpoint
last_updated: January 21, 2022
sidebar: mydoc_sidebar
permalink: mydoc_checkpoint.html
folder: mydoc
---

# Checkpoint

Checkpoint dirty durumdaki bellekte duran verinin topluca diske yazılma işlemidir.

Bu olay sırasında DB aşağıdaki 3 olayı gerçekleştirir.

1.  **Shared buffer**'daki tüm dirty (değiştirilmiş) blockları bulur.
2.  Tüm bu veriyi dosya sistemi belliğine yazar.
3.  Fiziksel diske yazmak için _**fsync()**_ çalıştırır.

4 şekilde çalışır:

1.  Elle komut olarak çalıştırma.
2.  Başka bir komutun ihtiyaç duymasından dolayı çalışma (`pg_start_backup('<backup_adi>')`, `CREATE DATABASE`, ya da `pg_ctl stop|restart`  gibi.)
3.  Son **_checkpoint'_**ten sonra belirlenmiş zamanın geçmesi  
  *   `Belirli aralık`: **`checkpoint_timeout` (seconds) **ile belirleriz.
4.  Son **_checkpoint'_**ten sonra belirlenmiş miktarda _WAL_ üretilmiş olması. 

  * `Belirli miktar` **`max_wal_size` (GB)** olarak belirlenmektedir. Her bir WAL 16MB olduğundan WAL dosyası sayısı = `max_wal_size` / WAL boyutu olarak düşünebiliriz. 

  * Checkpointlerin WAL miktarına değil de zamana bağlı olarak gerçekleştiriliyor olması istenen bir durumdur. Eğer WAL miktarından dolayı gerçekleşiyorsa `max_wal_size` parametresinin büyütülmesi tavsiye edilmektedir.

Ayrıca `checkpoint_completion_target` (0-1 arasında float)  bir sonraki checkpointin diğeriyle arasındaki mesafenin ne kadar zamanda bitmesi gerektiğini belirler. Yani bir sonraki **CHECKPOINT** 10 dk sonra ise 0.5 değeri checkpoint işlemini 5 dk'ya dağıtarak bitirmesini zorlar. 0.9 %90 zamanında bitirmesini sağlar. Bu sayede checkpointten kaynaklanan io yükselmeleri zamana dağıtılarak performans kazancı sağlanır.


## WAL Mimarisi

```
SELECT pg_walfile_name(pg_current_wal_lsn()), pg_current_wal_lsn();
     pg_walfile_name      | pg_current_wal_lsn
--------------------------+--------------------
 000000010000000300000081 | 3/812FCF08
(1 row)
 
 
select * from pg_walfile_name_offset(pg_current_wal_lsn());
  file_name | file_offset
--------------------------+-------------
 000000010000000000000001 | 8866032
(1 row)
```

### WAL Yapısı

WAL dosyasının adını 3 parçaya ayırarak okumak gerekir. 

```
00000001|00000003|00000081
```

*   1-8: zaman çizgisi (timeline) (restore operasyonlarında anlamlı)
*   9-16: mantıksal WAL dosyalarını göster
*   17-24: fiziksel  WAL'ı ifade eder. PostgreSQL bu kısmı **segment** olarak adlandırır. Her biri varsayılan olarak 16MB boyuttadır. 

Her bir mantıksal WAL, 255 tane dosyadan(fiziksel WAL) oluşan 4080 MB toplamı olan bir dosyalar bütünüdür.

Yukarıdaki `SELECT pg_current_wal_lsn();` sorgusundan dönen _3/812FCF08_ çıktı 2 parçaya ayrılır.

1.  WAL dosyasının mantıksal WAL'daki yerini söyler. 
2.  Bu mantıksal dosyanın başından itibaren ne kadar ilerde (offset) olduğunu gösterir. Bu rakamlar birbirlerinden mantıksal ve fiziksel yeri çıkarabilmektedir. WAL'ın yeri,  wall dosyalarının recovery ve replication işlemlerinde öneminden dolayı kritiktir. 

Her biri 16MB dir. Dolunca yenisine geçer. Ellede tetiklenebilir. _"select pg_switch_wal()"_ yazılmakta olan segmenti geçerek diğer wal segment dosyasına geçer.

### wal, xlog ya da transaction loglarının görevleri

*   recovery (Sunucu ya da servis aniden kapanmışsa db dosyalarını kullanılabilir duruma getirmek için master servisi wal dosyalarını okuyarak db'yi stabil hale getirir.)
*   sunucu başlangıcında (wal'ın kaldığı yeri db tutarlılığı ile kontrol eder.)
*   replikasyonda (streaming replication wal dosyalarını aktararak yapılır.)
*   incremental yedekleme (repmgr vb backup yazılımları doğrudan wal dosyalarını aktarır ve wal içerisindeki bir transactiona a dönebilir. )  

### WAL ile  config parametreleri

```sh
root@aa4bb340333d:/# grep wal /var/lib/postgresql/data/postgresql.conf
#wal_level = minimal # minimal, replica, or logical
 -- minimal sadece servis çakılırsa ve acil kapatmada kullanılabilir
 -- replica read only replika sunucuyu beslemek ya da pitr yapma işine yarar.
 -- logical ise logical replikasyon da kullanılabilir.
#wal_sync_method = fsync # the default is the first option
 -- fsync olmazsa diske yazmadan ok dönebilir. Bu vt bozulması riski oluşturur.
 -- fsync off olursa "wal_sync_method" parametresi geçersiz olur.
 -- fdatasync eski sürümlerde sadece
#wal_compression = off # enable compression of full-page writes
#wal_log_hints = off # also do full page writes of non-critical updates
#wal_buffers = -1 # min 32kB, -1 sets based on shared_buffers
 -- -1 shared buffera göre oto yapsın
#wal_writer_delay = 200ms # 1-10000 milliseconds
 -- ne kadar sıklıkla wal diske flash edilsin.
#wal_writer_flush_after = 1MB # measured in pages, 0 disables
#max_wal_size = 1GB
#min_wal_size = 80MB
#max_wal_senders = 0 # max number of walsender processes
#wal_keep_segments = 0 # in logfile segments, 16MB each; 0 disables
#wal_sender_timeout = 60s # in milliseconds; 0 disables
#wal_receiver_status_interval = 10s # send replies at least this often
#wal_receiver_timeout = 60s # time that receiver waits for
#wal_retrieve_retry_interval = 5s # time to wait before retrying to
```

Checkpoint_timeout 5 dk olarak düşünelim. Bu durumda 5 dakikada ne kadar wal üretildiğini 3 şekilde öğrenebiliriz. 

* `pg_current_wal_lsn()` ile belli zaman aralığındaki WAL pozisyonlarına bakarız. 
* postgresql.conf içerisinde `log_checkpoints=on`  ile checkpoint loglamayı açarak buradan ürütilen bilgiye bakarız.
* `pg_stat_bgwriter` view istatistiklerinden çıkarabiliriz. `select * from pg_stat_bgwrite;`

[https://www.postgresql.org/docs/current/static/functions-admin.html](https://www.postgresql.org/docs/current/static/functions-admin.html)

```sh
postgres=# SELECT pg_current_wal_insert_lsn();
 
pg_current_wal_insert_lsn
---------------------------------
 3D/B4020A58
(1 row)
 
... after 5 minutes ...
 
postgres=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------------
 3E/2203E0F8
(1 row)
 
postgres=# SELECT pg_wal_lsn_diff('3E/2203E0F8', '3D/B4020A58');
 pg_xlog_location_diff
-----------------------
            1845614240
(1 row)
```

Yukarıdaki sonuçlardan 5dk. da 1.84GB wal log üretildiğini görebiliriz. 


**Kaynaklar**

[https://www.postgresql.org/docs/current/functions-admin.html](https://www.postgresql.org/docs/current/functions-admin.html)

[http://eulerto.blogspot.com.tr/2011/11/understanding-wal-nomenclature.html](http://eulerto.blogspot.com.tr/2011/11/understanding-wal-nomenclature.html)
