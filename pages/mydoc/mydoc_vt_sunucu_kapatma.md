---
title: "Veritabanı Sunucusunu Kapatma"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 21, 2020
sidebar: mydoc_sidebar
permalink: mydoc_vt_sunucu_kapatma.html
folder: mydoc
---

## Veritabanı Sunucusunu Kapatma

Veritabanı sunucusunu kapatmanın supervisor `postgres` sürecine sinyal gönderme temelli birkaç yolu vardır. PostgreSQL'in önceden paketlenmiş bir sürümünü kullanıyorsanız ve sunucuyu başlatmak için paket yöneticisinin araçlarını kullandıysanız, sunucuyu durdurmak için de sağlanan araçları kullanmalısınız.

Sunucuyu doğrudan yönetirken, postgres sürecinin kapanma türünü farklı sinyaller göndererek kontrol edebilirsiniz:

`SIGTERM`

*Akıllı Kapatma* ( Smart Shutdown ) modudur. SIGTERM'i aldıktan sonra, sunucu yeni bağlantılara izin vermez. Mevcut oturumların çalışmalarını normal şekilde bitirmesine izin verir ve tüm oturumlar sona erdikten sonra kapanır. Sunucu çevrimiçi yedekleme modundaysa, çevrimiçi yedekleme modu artık etkin olmayana kadar bekler. Yedekleme modu etkinken yalnızca süper kullanıcıların yeni bağlantılarına izin verilir. ( süper kullanıcıya, çevrimiçi yedekleme modunu sonlandırması için izin verilir). Sunucu recovery'de SIGTERM aldığında, recovery ve streaming replikasyon işlemi oturumlar sona erdikten sonra durdurulur.

`SIGINT`

*Hızlı Kapatma* ( Fast Shutdown ) modudur. Sunucu yeni bağlantılara izin vermez ve tüm sunucu süreçlerine SIGTERM göndererek mevcut işlemlerini iptal etmelerine ve hemen çıkmalarına ister. Tüm sunucu süreçlerinin bitmesini bekleyerek kapanır. Sunucu çevrimiçi yedekleme modundaysa, yedekleme modu sonlandırılacak ve yedeklemeyi işe yaramaz hale getirecektir.

`SIGQUIT`

*Anında Kapatma* ( Immediate Shutdown ) modudur. Sunucu tüm alt süreçlere SIGQUIT gönderecek ve bunların sona ermesini bekleyecektir. Herhangi biri 5 saniye içinde sona erdirilmezse `SIGKILL` gönderilecektir. Ana sunucu süreci, tüm alt süreçler kapanır kapanmaz bitirilir ve normal veritabanı kapatma işlemi gerçekleşmez. Bu durum bir sonraki açılışta recovery (WAL loglarından yeniden oynatarak) işlemine sebep olur. Bu yöntem sadece acil durumlar için tavsiye edilir.

`pg_ctl` programı bu sinyalleri göndermek için uygun bir arayüz sağlar. Alternatif olarak, sinyali Windows olmayan sistemlerde doğrudan `kill` kullanarak gönderebilirsiniz. Postgres sürecinin PID'si `ps` programı kullanılarak veya veri dizinindeki `postmaster.pid` dosyasından bulunabilir. Örneğin, hızlı bir kapatma yapmak için:

```bash
$ kill -INT `head -1 /usr/local/pgsql/data/postmaster.pid`
```

{% include important.html content=" Sunucuyu kapatmak için `SIGKILL` kullanmamak tavsiye edilir. Böylece, sunucunun paylaşılan belleği ve semaforları serbest bırakması önlenir. SIGKILL sinyali `postgres` sürecini, alt süreçlere iletmesine izin vermeden sonlandıracağından alt süreçleri de tek tek elle öldürmek gerekebilir." %}

{% include tip.html content=" Sadece ilgili oturumu sonlandırmak için, **pg_terminate_backend( )** kullanın veya oturumla ilişkili alt sürece **SIGTERM** sinyali gönderin." %}

{% include links.html %}
