---
title: "Failover ve Recovery Olasılıkları"
parent: Yüksek Erişilebilirlik
layout: default
nav_order: 2
---

## Failover ve Recovery Olasılıkları

Replikadan Sistemi çalışır hale getirmek ve eski masterı replikayı takip ettirmek. Eski master'ı kurtarmak veritabanı boyutu çok büyükse anlamlıdır.

**1. Senaryo:**

Aşağıdaki 1. senaryo herşey düzgün giderse ve db1 kısa zaman içerisinde geri döndürülür senaryosu üzerine kuruludur. Streaming replication kurulumda, master fail eder ve replicalarda master ile sync durumda ise replicalardan birini yeni master olarak promote ederiz ve sistem çalışmaya devam eder. Eski master'ı düzelttikten sonra replica olarak sisteme bağladığımızda pg_wal varsa eski master yeni replica olarak sisteme dahil olacaktır.

{% include image.html file="failover_recovery_olasiliklari-1.png" %}

**2. Senaryo:**

Bu senaryoda, replicalar sorun yaşayan master'dan geri kalmışlarsa ve master'ı hızlı bir şekilde ayağa kaldırmak mümkün değilse replicalardan biri yeni master olarak promote edilir. Bu durumda eski ve yeni sistemlerin zaman çizgisi birbirlerinden farklılaşmış olacaktır ve eski master'ı replica olarak sisteme ekleyemeyiz.

Eski master sistem olarak geri döndürdüğümüzde 2 seçeneğimiz vardır. Ya base_backup alarak sıfırdan oluşturacağız ya da pg_rewind eklentisiyle varolan sisteme ekleyeceğiz. Birinci seçenek vt boyutu çok büyük olduğu zaman kullanılabilir olmaktan çıkmaktadır.
  
pg_rewind eklentisi, sadece değişen bloklara bakarak onları senkron ederek yeni master ve eski master'ı replica olarak sync eder. Bunun için pg_control dosyasına bakarak o zaman dilimine ulaşmak için farkı kopyalar.

{% include image.html file="failover_recovery_olasiliklari-2.png" %}

{% include links.html %}
