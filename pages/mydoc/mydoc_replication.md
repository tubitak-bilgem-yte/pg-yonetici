---
title: "Replication"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 04, 2020
summary: "Replication"
sidebar: mydoc_sidebar
permalink: mydoc_replication.html
folder: mydoc
---

## Replication

Bu bölümde bahsedecedilen ayarlar yerleşik [streaming replication](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION) özelliğinin davranışını kontrol eder. Sunucular primary ya da standby bir sunucu olabilir. Primary sunucular veri gönderebilir, standby ise her zaman replike verilerin alıcılarıdır. Standby sunucular [Cascading replication](https://www.postgresql.org/docs/current/warm-standby.html#CASCADING-REPLICATION) kullanıldığında alıcının (receiver) yanı sıra gönderici (senders) de olabilir. Parametreler esas olarak gönderme ve standby sunucuları içindir, ancak bazı parametreler yalnızca primary sunucuda anlamlıdır. Ayarlar, gerekirse küme genelinde sorunsuz olarak değişebilir.

### Sending Servers

Bu parametreler, replika verileri bir veya daha fazla standby sunucusuna gönderecek herhangi bir sunucuda ayarlanabilir. Primary her zaman gönderen bir sunucudur, bu nedenle bu parametreler her zaman primary üzerinde ayarlanmalıdır. Bu parametrelerin rolü ve anlamı standby, primary olduktan sonra değişmez.

{% include callout.html content="**`max_wal_senders (integer)`**: Standby sunucularından veya streaming tabanlı yedekleme istemcilerinden gelen maksimum eşzamanlı bağlantı sayısını belirtir. Aynı anda çalışan maksimum WAL sender süreci sayısıdır. Öntanımlı değeri 10'dur. 0 değeri replication'un devre dışı bırakıldığı anlamına gelir. Bir streaming istemcisinin bağlantısının aniden kesilmesi zaman aşımına ulaşılıncaya kadar orphan bir bağlantı slotuna sebep olabilir. Bu nedenle bu parametre, bağlantısı kesilen istemcilerin hemen yeniden bağlanabilmesi için beklenen maksimum istemci sayısından fazla ayarlanmalıdır. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Ayrıca, standby sunucularından bağlantılara izin vermek için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır.<br/><br/>

Bir standby sunucu çalıştırırken, bu parametreyi primary sunucudakiyle aynı veya daha yüksek bir değere ayarlamalısınız. Aksi takdirde standby sunucusunda sorgulara izin verilmeyecektir." type="primary" %}

{% include callout.html content="**`max_replication_slots (integer)`**: Sunucunun destekleyebileceği maksimum [replication slotu](https://www.postgresql.org/docs/current/warm-standby.html#STREAMING-REPLICATION-SLOTS) sayısını belirtir. Öntanımlı değeri 10'dur. Bu parametre yalnızca sunucu başlangıcında ayarlanabilir. Mevcut replication slotlarının sayısından daha düşük bir değere ayarlamak, sunucunun başlamasını engeller. Ayrıca, replication slotlarının kullanılmasına izin vermek  için `wal_level` paremetresi `replica` veya daha yüksek seviyede ayarlanmalıdır" type="primary" %}

### Master Server

### Standby Servers

### Subscribers


{% include links.html %}