---
title: "PostgreSQL Logları"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 13, 2020
summary: "PostgreSQL Logları"
sidebar: mydoc_sidebar
permalink: mydoc_postgresql_loglari.html
folder: mydoc
---

## PostgreSQL Logları

- PostgreSQL logları varsayılan olarak veri dizininde `/log` dizini altında tutulur
- Log dosyası, *syslog*, *eventlog*, *csvlog* gibi seçenekler kullanılabilir
- Loglamanın detay seviyesi ayarlanabilir
- Sorgu logları ayrıca tutulabilir

### Logların Yazılışı

Loglama ayarları `postgresql.conf` dosyasında *# ERROR REPORTING AND LOGGING* bölümünde bulunur:

```js
log_destination = stderr
logging_collector = on
log_directory = log
log_filename = 'postgresql-%a.log'
log_rotation_age = 1d
```

*Syslog* kullanılacaksa:

```js
log_destination = syslog
syslog_facility = 'LOCAL0'
syslog_ident = 'postgres'
```

### Log Seviyeleri

```sql
DEBUG1...DEBUG5
INFO
NOTICE
WARNING
ERROR
LOG
FATAL
PANIC
```

- Seviyeler ayar dosyasında sınıflandırılmıştır
- Farklı konularda farklı seviyeler (istemci, sunucu v.b.)
- Seçilen seviye ve üzeri seviyelerdeki eylemler loglanır

- Varsayılan loglama seviyeleri:

```sh
client_min_messages = notice
log_min_messages = warning
log_min_error_statement = error
```

### Neler Loglanacak?

Varsayılan olarak detay loglar kapalıdır. Açılmak istenirse birçok seçenek var:

- Bağlantı başlatma ve bitirme logları
- Sunucu ismi
- Lock beklemeleri
- Replikasyon logları

Sunucu performansını değerlendirmek için sorgu logları açılmak istenebilir. Varsayılan olarak kapalıdır.

```yaml
log_statement = 'none'          # Kullanılabilecek değerler: none, ddl, mod, all
```

Belirli bir zamandan uzun süren sorguların loglanması için de şu iki ayar var:

```sql
log_min_duration_statement = -1
log_duration = off
```

Log formatını değiştirmek, başına farklı ekler yapmak da mümkündür. Varsayılanı başında zaman damgası olanı:

```sh
log_line_prefix = '< %m >'
```

### Bazı Başlangıç Hata Logları

```sql
LOG:  could not bind IPv4 socket: Address already in use
HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
FATAL:  could not create TCP/IP listen socket
```

- TCP 5432 portu kullanılamıyor.
- Port başka bir süreç tarafından kullanılıyor.

```sql
LOG:  could not bind IPv4 socket: Permission denied
HINT:  Is another postmaster already running on port 666? If not, wait a few seconds and retry.
FATAL:  could not create TCP/IP listen socket
```

- Belirtilen portu kullanmaya izin yok. (Rezerve port numarası)

```sql
FATAL:  could not create shared memory segment: Invalid argument
DETAIL:  Failed system call was shmget(key=5440001, size=4011376640,03600).
```

- Paylaşılan bellek boyutunun çekirdek limiti miktarı, PostgreSQL’in oluşturmaya çalıştığı çalışma alanı boyutu miktarından az!

```sql
FATAL:  could not create semaphores: No space left on device
DETAIL:  Failed system call was semget(5440126, 17, 03600).
```

- Diskte yer yok.

```sql
ERROR: could not set permissions on directory "/var/lib/pgsql": Permission denied
```

- SELinux (Erişim Denetimi Aracı) erişim izni vermesi için ayarlanmamış

### Bazı İstemci Hataları

```sql
psql: could not connect to server: Connection refused
        Is the server running on host "server.joe.com" and accepting
        TCP/IP connections on port 5432?
```

- PostgreSQL sunucuda TCP/IP bağlantılarına izin verilmesi ayarlanmamış

- TCP/IP izin verilmiş ise, güvenlik duvarı 5432 portunu blokluyor olabilir

```sql
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

- Unix soketi üzerinden çalışan servis yok

### PgAudit ile Audit Loglarının Tutulması

PostgreSQL’in standart sorgu loglamasına ek olarak sadece belirli işlemlere ve nesnelere ait loglarının tutulmasını sağlar.

PostgreSQL deposundan gerekli eklenti kurulur:

```text
# yum install pgaudit_96
```

Ayar dosyasına girerek, eklentimizi ekliyoruz:

```text
$ vi postgresql.conf
  
shared_preload_libraries='pgaudit'
```

Servisi yeniden başlatıp, psql’le bağlandıktan sonra:

```sql
CREATE EXTENSION pgaudit;
```

*pgaudit.log* değiskeni ile okuma loglarının yakalanması:

```sql
ALTER SYSTEM SET pgaudit.log TO 'read';
SELECT pg_reload_conf();
```

- `all` ile bütün logları tutabileceğimiz gibi virgülle `read`, `write` şeklinde alabiliriz.

Sorgumuzu çalıştırıp logumuza bakalım.

```sql
SELECT count(*) FROM doviz;
```

```sql
$ cd /var/lib/pgsql/11/data/log
$ grep AUDIT postgresql-Tue.log | grep READ

< 2018-09-04 18:18:53.594 +03 > LOG:  AUDIT: SESSION,2,1,READ,SELECT,,,SELECT * FROM doviz;,<not logged>
```

{% include links.html %}
