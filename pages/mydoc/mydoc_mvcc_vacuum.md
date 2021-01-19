---
title: "PostgreSQL MVCC ve VACUUM"
tags: [PostgreSQL]
keywords: postgres
last_updated: November 13, 2020
summary: "PostgreSQL MVCC ve VACUUM"
sidebar: mydoc_sidebar
permalink: mydoc_mvcc_vacuum.html
folder: mydoc
---

## PostgreSQL MVCC ve VACUUM

### MVCC

MVCC = MultiVersion Concurrency Control (Çoklu Sürüm Eşzamanlılık kontrolü) PostgreSQL’in sorgu tutarlılığı ve kilitleme optimizasyonu çözümüdür. Bir satır değiştiren sorgu geldiğinde satırın yeni bir sürümünü çıkıp onun üzerinde çalışır. Eski sorgular satırın eski sürümünden okumaya devam eder. MVCC yapısında Okuma işlemi yazma işlemini bloklamaz, yazma işlemi de okuma işlemini bloklamaz böylece bir transaction başka bir transactiondan etkilenmeden çalışmış olur. Update işlemlerinde satırın ilk sürümü ile iş bittiğinde satır expired olarak işaretlenir ama silinmez. Bu kirli satırları tekrar PostgreSQL’in kullanabilmesi için VACUUM çalışmalıdır.

### VACUUM

MVCC ile saklanan eski sürüm satırları tekrar kullanılabilir hale getirir. Ayrıca `ANALYZE` ile birlikte sorgu planlayıcısının kullandığı istatistiklerin güncel olmasını sağlar. Bu işlemin düzenli olarak yapılması gerekir.

{% include links.html %}
