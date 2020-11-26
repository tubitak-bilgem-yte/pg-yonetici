---
title: "SSH Tünelleri ile Güvenli TCP / IP Bağlantıları"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 25, 2020
summary: "SSH Tünelleri ile Güvenli TCP / IP Bağlantıları"
sidebar: mydoc_sidebar
permalink: mydoc_tcp_ip_ssh.html
folder: mydoc
---

## SSH Tünelleri ile Güvenli TCP / IP Bağlantıları

İstemciler ve bir PostgreSQL sunucusu arasındaki ağ bağlantısını şifrelemek için SSH kullanılabilir. Uygun şekilde kullanıldığında, SSL kabiliyeti olmayan istemciler için bile yeterince güvenli bir ağ bağlantısı sağlar.

Öncelikle, SSH sunucusunun PostgreSQL sunucusuyla aynı makinede düzgün çalıştığından ve bir kullanıcı olarak `ssh` ile oturum açabildiğinizden emin olun. Sonrasında uzak sunucu ile bir güvenli tünel kurabilirsiniz. Güvenli tünel, yerel bir bağlantı noktasında dinleme yapar ve tüm trafiği uzak makinedeki bir bağlantı noktasına iletir. Uzak bağlantı noktasına gönderilen trafik, localhost adresine veya farklı bağlantı adresine ulaşabilir ve bu yerel makinenizden geliyormuş gibi görünmez. Aşağıdaki komut, istemci makineden uzak `bilgemyte.com` makinesine güvenli bir tünel oluşturur:

```bash
ssh -L 63333:localhost:5432 tubitak@bilgemyte.com
```

{% include callout.html content="`-L` argumanındaki 63333, tünelin yerel bağlantı noktası ( local port ) numarasıdır. Bu, kullanılmayan herhangi bir bağlantı noktası olabilir. Daha sonraki ad veya IP adresi, uzak bağlantı adresidir. Burada varsayılan değer olan localhost'tur. 5432, tünelin uzak ucudur. Bu örnekte, veritabanı sunucunuzun kullandığı bağlantı noktası numarasıdır. Oluşturduğumuz tüneli kullanarak veritabanı sunucusuna bağlanmak için yerel makine üzerindeki 63333 numaralı bağlantı noktasına bağlanırsınız:" type="info" %}

```bash
psql -h localhost -p 63333 postgres
```

Veritabanı sunucusu, `tubitak` kullanıcısını `bilgemyte.com`'dan localhost bağlama adresine bağlanan bir bilgisayar gibi görünecek ve ilgili bağlantı adresine yapılan bağlantılarda bu kullanıcı için yapılandırılmış kimlik doğrulama prosedürünü kullanacaktır.

{% include note.html content=" SSH sunucusu ile PostgreSQL sunucusu arasında bağlantı şifrelenmemiş olduğundan, sunucu bağlantının SSL şifreli olduğunu düşünmeyecektir. Bu durum aynı makinede oldukları için herhangi bir ekstra güvenlik riski oluşturmaz."%}

Tünel kurulumunun başarılı olduğunu anlamak için `ssh` ile `tubitak@bilgemyte.com` üzerinden yapacağınız istek sanki terminal oturumu oluşturmak için `ssh` kullanmayı denemişsiniz gibi izin verilmesi gerekir.

Bağlantı noktası yönlendirmeyi şu şekilde de kurabilirdiniz:

```bash
ssh -L 63333:bilgemyte.com:5432 tubitak@bilgemyte.com
```

{% include callout.html content="Bu kurulumda veritabanı sunucusu, bağlantının varsayılan `listen_addresses = 'localhost'` ayarıyla açılmayan `bilgemyte.com` bağlama adresine geldiğini görecektir. Bu pek istenilen bir durum değildir." type="info" %}

Veritabanı sunucusuna direkt ulaşmanız gereken durumlarda kurulum şöyle olmalıdır:

```bash
ssh -L 63333:db.bilgemyte.com:5432 tubitak@shell.bilgemyte.com
```

{% include note.html content=" Bu kullanımda `shell.bilgemyte.com`'dan `db.bilgemyte.com`'a olan bağlantı SSH tüneli tarafından şifrelenmez. Daha fazla ayar ve bilgi için SSH belgelerine bakın."%}

{% include links.html %}
