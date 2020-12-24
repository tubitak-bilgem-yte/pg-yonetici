---
title: "SSH Tünelleri ile Güvenli TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres, ssh, tcp
last_updated: December 23, 2020
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_ssh.html
folder: mydoc
---

## SSH Tünelleri ile Güvenli TCP / IP Bağlantıları

İstemciler ile PostgreSQL sunucusu arasındaki bağlantısı şifrelemek için SSH kullanılabilir.

SSH sunucusunun PostgreSQL sunucusuyla aynı makinede düzgün çalıştığından ve bir kullanıcı olarak `ssh` ile oturum açabildiğinizden emin olduktan sonra uzak sunucu ile güvenli tünel kurabilirsiniz. Güvenli tünel, yerel bir bağlantı noktasında dinleme yapar ve tüm trafiği uzak makinedeki bir bağlantı noktasına iletir. Aşağıdaki komut, istemci makineden uzak `bilgemyte.com` makinesine güvenli bir tünel oluşturur:

```bash
ssh -L 63333:localhost:5432 tubitak@bilgemyte.com
```

{% include callout.html content="`-L` argumanındaki 63333, tünelin local port numarasıdır. Bu, kullanılmayan herhangi bir bağlantı noktası olabilir. Daha sonraki ad veya IP adresi, uzak bağlantı adresidir. Burada varsayılan değer olan localhost'tur. 5432, tünelin uzak ucudur. Bu örnekte, veritabanı sunucusunun kullandığı port numarasıdır. Oluşturduğumuz tüneli kullanarak veritabanı sunucusuna bağlanmak için yerel makine üzerindeki 63333 numaralı porta bağlanırsınız:" type="info" %}

```bash
psql -h localhost -p 63333 postgres
```

Veritabanı sunucusu, `tubitak` kullanıcısını, `bilgemyte.com`'daki localhost bağlama adresine bağlanan bir bilgisayar gibi görünecek ve ilgili bağlantı adresine yapılan bağlantılarda bu kullanıcı için yapılandırılmış kimlik doğrulama prosedürünü kullanacaktır.

Tünel kurulumunun başarılı olduğunu anlamak için `ssh` ile `tubitak@bilgemyte.com` üzerinden yapacağınız isteğe izin verilmelidir.

Bağlantı noktası yönlendirmeyi şu şekilde de kurabilirdiniz:

```bash
ssh -L 63333:bilgemyte.com:5432 tubitak@bilgemyte.com
```

{% include callout.html content="Bu kurulumda veritabanı sunucusu, bağlantının varsayılan `listen_addresses = 'localhost'` ayarıyla açılmayan `bilgemyte.com` bağlama adresine geldiğini görecektir. Bu pek istenilen bir durum değildir." type="info" %}

Veritabanı sunucusuna direkt ulaşmanız gereken durumlarda kurulum şu şekilde olabilir:

```bash
ssh -L 63333:db.bilgemyte.com:5432 tubitak@shell.bilgemyte.com
```

{% include note.html content=" Bu kullanımda `shell.bilgemyte.com`'dan `db.bilgemyte.com`'a olan bağlantı SSH tüneli tarafından şifrelenmez."%}

{% include links.html %}
