---
title: "Prometheus ile İzleme"
parent: Veritabanı İzleme ve Loglama
layout: default
nav_order: 5
--- 

## Prometheus ile İzleme

Prometheus, altyapıda çalışan servislerin çalışma zamanındaki metriklerini toplayan ve bunları bir zaman serisi veritabanında saklayan güçlü ve açık kaynaklı bir izleme sistemidir. Tek tek sistem olarak kendi üzerinde bir verileri görme ve grafik yapma özelliği olmasının yanı sıra, grafana gibi araçlarla çok boyutlu bir veri modeli, esnek bir sorgu dili ve çeşitli görselleştirme olanakları sunar. Ayrıca bir çok programlama dilinde istemci kütüphanesi mevcut olup, uygulama içerisinden prometheusa veri gönderebilmektedir.

Go dilinde yazılmıştır ve ayar dosyaları olarak yaml kullanılmaktadır.

Varsayılan olarak, Prometheus yalnızca kendi hakkındaki metrikleri dışa aktarır (örneğin, aldığı isteklerin sayısı, hafıza tüketimi vb.). Ancak izlemek istediğimiz diğer servisler için exporter adında bir mimarisi bulunmaktadır. Bu mimari sayesinde, ek metrikler üreten isteğe bağlı programları yükleyerek Prometheus'u büyük ölçüde genişletebilirsiniz.

Exporters - hem Prometheus ekibinin hem de katkıda bulunan topluluğun ürettiği, altyapıdan, veritabanlarından ve web sunucularından mesajlaşma sistemlerine, API'lerden ve daha fazlasına kadar her şey hakkında bilgi sağlayan exporter mevcuttur.

Bunlardan biri de postgresql_exporter modülüdür.

Prometheus'ta her exporter kendi başına bir servis olarak çalışır.

prometheus servisi [buradan](https://prometheus.io/download/) indirilip kurulabilir.

```bash
wget https://github.com/prometheus/prometheus/releases/download/v{prometheus_version}/prometheus-{prometheus_version}.linux-amd64.tar.gz

tar xzf prometheus-{prometheus_version}.linux-amd64.tar.gz

cd prometheus-{prometheus_version}.linux-amd64
```

promotheus.yml'yi açıp PostgreSQL izlemek için kullanacağımızdan varsayılan dokümanın altına aşağıdaki satırı ekliyoruz.

````bash
vi promotheus.yml 

  - job_name: postgres
    scrape_interval: 10s
    scrape_timeout: 10s
    static_configs:
    - targets: ['0.0.0.0:9091']
```

{% include links.html %} 