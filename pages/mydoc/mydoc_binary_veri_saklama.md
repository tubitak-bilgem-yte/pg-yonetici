---
title: "Binary Veri Saklama"
tags: [PostgreSQL]
keywords: postgres, binary
last_updated: December 31, 2020
sidebar: mydoc_sidebar
permalink: mydoc_binary_veri_saklama.html
folder: mydoc
---

## Binary Veri Saklama

PostgreSQL'de (ve aslında veritabanlarında) binary veri saklama ile ilgili iki genel yaklaşım vardır:

- Veriyi veritabanında saklamak: Bu görüşü savunanların en önemli dayanağı, transaction bütünlüğüdür. Bir dosyayı veritabanına yazarken dosya ve ilgili bilgileri aynı anda yazıldığından veri kaybı olmayacaktır. Bu yöntemin dezavantajı ise yedeklemedir. Veritabanı boyutunu ciddi derecede arttıracak ve yedeği de büyütecek bir yöntemdir.
- Dosyaları dosya sisteminde, bilgilerini veritabanında saklamak: Bu görüşü savunanların dayanağı da veri boyutunun daha az olmasıdır. Üstteki yöntemin aksine, dosya ve dosya bilgileri ayrı ayrı işlemlerde yazılacağı için, iki işlemden birisinde olacak bir hata, aradaki ilişkinin bozulmasına neden olabilecektir.

### Binary Veri Saklama Önerileri

1 MB boyutlarında milyon tane seviyesinde belge saklanması planlanan YTE isimli projemiz olduğunu varsayalım. Üstteki iki yöntemi de incelediğimizde, projenin bütünlüğü için veriyi veritabanında saklamanın en iyi yöntem olduğunu düşünülür. Ancak burada karşımıza önemli bir yedekleme sorunu çıkar.

Bu aşamada şu şekilde bir yol izlenebilir: Dosyaları farklı bir PostgreSQL sunucusunda saklamak ve FDW yardımıyla ana veritabanı sunucusuna bu veritabanını bağlamak.

Özetle yapı şu şekilde olur:

- YTE veritabanı
- Binary data veritabanı

Binary data veritabanını `postgres_fdw` eklentisi ile bağlayacağız.

Bu yapı ile aşağıdaki dezavantajlar kaldırılır:

- Binary data ve ilişkili veriler ayrı bir veritabanında olacağı için YTE veritabanı sunucusuna ek yük gelmeyecektir.
- Yeni veritabanının yedeğini ayrı bir politika ile alabileceğiz (daha az sıklıkta yedekleme gibi).
- Binary dataların getireceği RAM yükünü ayrı bir sunucuya vereceğiz.
- Uygulama etkilenmeyecek, sanki aynı veritabanındaymış gibi sorgu yapacaktır.

{% include links.html %}
