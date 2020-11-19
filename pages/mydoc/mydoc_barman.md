---
title: "Barman"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 19, 2020
summary: "Barman"
sidebar: mydoc_sidebar
permalink: mydoc_barman.html
folder: mydoc
---

## Barman

- Barman, 2ndQuadrant ve topluluk tarafından geliştirilen bir özgür yazılım [](http://www.pgbarman.org/).
- Sıcak yedek sunucular, PITR, tam ve arttırımlı yedekleme, çoklu ana ve yedek sunucu desteği
- Barman katalogu sayesinde yedek ve geri dönüşleri kolay yönetme

### Kurulum ve Ayarlar - Barman

Postgresql deposu kurulup barman indirilip kurulur:

```text
# yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
# yum install barman rsync epel-release -y
```

- Barman’ın otomatik çalışabilmesi için, Postgresql ve Barman sunucularının karşılıklı parolasız SSH yapması gerekiyor.
  - Postgresql sunucusundaki postgres kullanıcısının keyi ile Barman sunucusunda barman kullanıcısına,
  - Barman sunucusundaki barman kullanıcısının keyi ile Postgresql sunucusunda postgres kullanıcısına erişilmelidir.

Ayar dosyasında genel ayarlar aşağıdaki gibi belirtilir:

```text
# vim /etc/barman.conf

compression = gzip
immediate_checkpoint = true
basebackup_retry_times = 3
basebackup_retry_sleep = 30
last_backup_maximum_age = 1 DAYS
reuse_backup = link
archiver = on
```

Ayar dosyasının sonuna, yedekleyeceğimiz Postgresql sunucusu için ayrı bir blok ekliyoruz:

```text
# vim /etc/barman.conf

[postgresql-sunucu]
description = "Ana Postgresql Sunucusu"
ssh_command = ssh postgres@192.168.56.101
conninfo = host=192.168.56.101 user=postgres
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 14 days
wal_retention_policy = main
```

Yedeklenecek Postgresql’de Barman sunucusuna ve kullanıcısına izin verilir:

```text
# vim /var/lib/pgsql/11/data/pg_hba.conf
host     all             postgres             192.168.56.104/32      trust
```

Postgresql’in wal loglarını yazıp Barman sunucusuna da arşivlemesi için PostgreSQL sunucusunda şu ayarlar yapılır:

```text
# vim /var/lib/pgsql/11/data/postgresql.conf
listen_addresses = '*'
wal_level = replica
archive_mode = on
archive_command = 'rsync -a %p barman@192.165.56.104:/var/lib/barman/postgresql-sunucu/incoming/%f'
```

{% include note.html content=" Arşivleme komutunun çalışması için Postgresql sunucu tarafında rsync yoksa kurulması gerekir."%}

Erişimlerin sorunsuz olduğunu kontrol için:

```text
# su - barman
$ barman check postgresql-sunucu
Server postgresql-sunucu:
    PostgreSQL: OK
    is_superuser: OK
    wal_level: OK
    directories: OK
    retention policy settings: OK
    backup maximum age: FAILED (interval provided: 1 day, latest backup age: No available backups)
    compression settings: OK
    failed backups: OK (there are 0 failed backups)
    minimum redundancy requirements: OK (have 0 backups, expected at least 0)
    ssh: OK (PostgreSQL server)
    not in recovery: OK
    archive_mode: OK
    archive_command: OK
    continuous archiving: OK
    archiver errors: OK
```

- `retention_policy` ile 14 günlük geri dönüşe izin verecek kadar base backup ve wal log tutacağımızı belirttik.
- barman `check` komutu çalıştığında herşey OK görünmeli, sadece backup maximum age `FAILED` olabilir çünkü henüz hiç yedek almadık.

### Kullanım

Elle yedekleme başlatma:

```text
$ barman backup postgresql-sunucu
Starting backup using rsync-exclusive method for server postgresql-sunucu in /var/lib/barman/postgresql-sunucu/base/20180322T210335
Backup start at LSN: 0/B000060 (00000001000000000000000B, 00000060)
This is the first backup for server postgresql-sunucu
WAL segments preceding the current backup have been found:
        000000010000000000000005 from server postgresql-sunucu has been removed
        000000010000000000000006 from server postgresql-sunucu has been removed
        000000010000000000000007 from server postgresql-sunucu has been removed
        000000010000000000000008 from server postgresql-sunucu has been removed
        000000010000000000000009 from server postgresql-sunucu has been removed
Starting backup copy via rsync/SSH for 20180322T210335
Copy done (time: 20 seconds)
```

```text
This is the first backup for server postgresql-sunucu
WAL segments preceding the current backup have been found:
        00000001000000000000000A from server postgresql-sunucu has been removed
Asking PostgreSQL server to finalize the backup.
Backup size: 23.4 MiB. Actual size on disk: 23.4 MiB (-0.00% deduplication ratio).
Backup end at LSN: 0/B000168 (00000001000000000000000B, 00000168)
Backup completed (start time: 2018-03-22 21:03:37.777884, elapsed time: 25 seconds)
Processing xlog segments from file archival for postgresql-sunucu
        00000001000000000000000B
        00000001000000000000000B.00000060.backup
```

Otomatik yedekleme için crontab’a:

```text
# vim /etc/cron.d/barman

30 23 * * * barman [ -x /usr/bin/barman ] && /usr/bin/barman backup postgresql-sunucu
*  *  * * * barman [ -x /usr/bin/barman ] && /usr/bin/barman -q cron
```

Mevcut yedekleri listeleme:

```text
$ barman list-backup postgresql-sunucu
postgresql-sunucu 20180322T210335 - Thu Mar 22 21:04:03 2018 - Size: 23.4 MiB - WAL Size: 0 B
```

Belirli bir yedeğin detaylarını görme:

```text
$ barman show-backup postgresql-sunucu 20180322T210515
Backup 20180322T210515:
  Server Name            : postgresql-sunucu
  Status                 : DONE
  PostgreSQL Version     : 100003
  PGDATA directory       : /var/lib/pgsql/11/data

  Base backup information:
    Disk usage           : 23.4 MiB (23.4 MiB with WALs)
    Incremental size     : 20.7 KiB (-99.91%)
    Timeline             : 1
    Begin WAL            : 00000001000000000000000D
    End WAL              : 00000001000000000000000D
    WAL number           : 1
    WAL compression ratio: 99.84%
    Begin time           : 2018-03-22 21:05:16.505850+03:00
    End time             : 2018-03-22 21:05:27.638172+03:00
    Copy time            : 5 seconds + 4 seconds startup
```

```text
    Estimated throughput : 3.9 KiB/s
    Begin Offset         : 40
    End Offset           : 304
    Begin LSN           : 0/D000028
    End LSN             : 0/D000130

  WAL information:
    No of files          : 0
    Disk usage           : 0 B
    Last available       : 00000001000000000000000D

  Catalog information:
    Retention Policy     : VALID
    Previous Backup      : 20180322T210335
    Next Backup          : - (this is the latest base backup)
```

Belirli bir zamandaki postgresql-sunucu yedeğini başka bir PostgreSQL’e geri dönme:

```text
$ barman recover --target-time="2018-03-22 13:00:00+02:00" \
    --remote-ssh-command="ssh postgres@127.0.0.1" postgresql-sunucu \
    20180322T210515 /var/lib/postgresql/11/main/

Starting remote restore for server postgresql-sunucu using backup 20180322T210515
Destination directory: /var/lib/postgresql/11/main/
Doing PITR. Recovery target time: '2018-03-22 13:00:00+02:00'
Copying the base backup.
Copying required WAL segments.
Generating recovery.conf
```

{% include note.html content=" Tüm barman komutları barman kullanıcısı ile çalıştırılır."%}

{% include links.html %}
