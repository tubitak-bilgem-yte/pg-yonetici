---
title: "MVCC ve VACUUM"
layout: default
parent: PostgreSQL Nasıl Çalışır?
nav_order: 5
---

## PostgreSQL MVCC ve VACUUM

### MVCC

MVCC = MultiVersion Concurrency Control (Çoklu Sürüm Eşzamanlılık kontrolü) PostgreSQL’in sorgu tutarlılığı ve kilitleme optimizasyonu çözümüdür. Bir satır değiştiren sorgu geldiğinde satırın yeni bir sürümünü çıkıp onun üzerinde çalışır. Eski sorgular satırın eski sürümünden okumaya devam eder. MVCC yapısında Okuma işlemi yazma işlemini bloklamaz, yazma işlemi de okuma işlemini bloklamaz böylece bir transaction başka bir transactiondan etkilenmeden çalışmış olur. Update işlemlerinde satırın ilk sürümü ile iş bittiğinde satır expired olarak işaretlenir ama silinmez. Bu kirli satırları tekrar PostgreSQL’in kullanabilmesi için VACUUM çalışmalıdır.

### VACUUM

MVCC ile saklanan eski sürüm satırları tekrar kullanılabilir hale getirir. Ayrıca `ANALYZE` ile birlikte sorgu planlayıcısının kullandığı istatistiklerin güncel olmasını sağlar. Bu işlemin düzenli olarak yapılması gerekir.

{% include links.html %}
