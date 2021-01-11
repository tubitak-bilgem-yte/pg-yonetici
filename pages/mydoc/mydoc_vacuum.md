---
title: "Vacuum"
tags: [PostgreSQL]
keywords: postgres, Vacuum 
last_updated: January 7, 2021
sidebar: mydoc_sidebar
permalink: mydoc_vacuum.html
folder: mydoc
---

## Postgresql Vacuum İşlemi

Bir veri tabanında insert update delete operasyonları veriyi güncelleyen operasyon türleridir. Postgresql veri tabanında DELETE ve UPDATE operasyonları sırasında davranış şekli şu şekildedir;

Bir page yani veri bloğuna delete veya update operasyonu geldiği zaman o page’de bulunan ilgili tuple mantıksal olarak silinir. Update operasyonu verinin mantıksal olarak silinip yeni versiyonun başka bir tuple’a insert edilmesinden ibarettir. Var olan tuple’ın fiziksel olarak güncellenmesi gibi bir durum söz konusu değildir. Mantıksal olarak silinmesinden kasıt verinin hala ilgili page’de bulunuyor olması; ancak bir pointer ile verinin kullanılmaz olarak işaretlenmesidir.

Kullanılamaz olarak işaretlenen tuple’lara dead tuple denir. Dead Tuple’ların disk üzerinden kaldırılması için Vacuum işlemi kullanılır. Yani vacuum işlemi disk üzerindeki page’lerin temizlenmesi demektir. Bu noktada Vacuum işlemi disk üzerinde herhangi bir boş alan yaratmaz. Sadece dead tuple’ları yeniden kullanılabilir olarak işaretler.

## Vacuum vs Vacuum Full

Vacuum işlemi tablo üzerinde lock oluşturmaz ancak Vacuum Full işlemi tablolar üzerinde lock oluşturur. Vacuum process’i paralel olarak da çalıştırılabilir. Vacuum full işlemi sırasında dead tuple’lardan arındırılmış halde tablonun yeni bir kopyası oluşturulur. Tabloya lock koyulma sebebi budur. Tablo üzerinde varsa indexler de kopyalanan yeni tablo üzerinde yeniden oluşturulur. Dolayısı ile indexler yeniden oluşturulduğu için VACUUM işleminden sonra tekrar analiz yapılmaz.

Vacuum işlemi sequential scan sırasında okunan tuple sayısını azaltacağı için okuma işlemlerinde de hızlanmaya sebep olacaktır. Sequential scan sıralı okuma işlemidir ve dead tuple’ları atlamaz. Dolayısı ile dead tuple’ların temizlenmesi daha az satır okunması anlamına gelir.

Çok fazla güncellenen tablolarda vacuum işleminin düzenli olarak yapılması gerekir. Vacuum işlemi veri tabanı boyutunu azaltır ve performansı arttırır.

Autovacuum özelliği ile bu işlemlerin manuel olarak yapılmasına gerek kalmamıştır. Autovacuum işlemi sırasında hem vacuum hem de analiz işlemi yapılacağından veri tabanı performansını arttıtır.

**Kaynak:**

[1]. [veritabaniegitimleri.com: Postgresql Vacuum İşlemi Nedir?](http://www.veritabaniegitimleri.com/2020/04/24/postgresql-vacuum-islemi-nedir/)

{% include links.html %}