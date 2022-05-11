---
title: "Data Aging"
parent: Veritabanı Yönetimi
layout: default
nav_order: 5
--- 

## Data Aging

Data Aging, eski ve kullanılmayan verilerin başka bir storage ortamına taşınması, böylece performans ve storage optimizasyonunun sağlanmasıdır. Çok uzun süreki veri saklamak istediğimizde, o kadar veriyi hem saklamak, hem de yedeklemek için daha çok alana gereksinim duyacağımız ortadadır. Data aging özellikle performans sorununu çözer.

Data aging'de önemli bir nokta, güncel verinin daha iyi disklerde tutulması, ama bir yandan da gerektiğinde sorgulayabilmektir.

### Data Aging altyapısı

Data aging için partitioning ve tablespacelerin beraber kullanımı genellikle önerilir. PostgreSQL'de partitioning, 9.6 sürümü dahil trigger ve constraint yardımı ile yapılmaktadır.

Data aging yapılacağı zaman, iki yol bulunmaktadır:

- Verileri başka tablespacelere taşımak,
- Eski partitionları drop etmek.

Ayrınlı bilgi için [Tablespace](mydoc_tablespace.html) ve [Partitioning](mydoc_partitioning.html) sayfalarına bakın.

{% include links.html %}
