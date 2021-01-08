---
title: "PostgreSQL Yönetici Dokümantasyon Projesi Katkı Rehberi"
tags: [PostgreSQL]
keywords: postgres
last_updated: January 6, 2021
sidebar: mydoc_sidebar
permalink: mydoc_katki_rehberi.html
folder: mydoc
---

## PostgreSQL Dokümantasyon Projesi Katkı Rehberi

### Projeyi Düzenleme

1. Projeyi düzenlemek için öncelikle repository'nin **`fork`** edilmesi gerekmektedir.

2. Repository fork edildikten sonra yapılmak istenilen katkılar aşağıda belirtildiği şekilde yapılır ve açıklayıcı bir commit mesajı ile **`commit`** edilir.

3. Commit edildikten sonra repository'e dönülür ve açıklayıcı bir mesaj ile **`pull request`** açılır.

4. Pull request açıldıktan sonra **proje yöneticileri** duruma göre katkıyı ekler veya geri çevirir.

---

### Yeni Sayfa Ekleme

PostgreSQL dokümantasyon projesi sayfaları markdown formatında `/pages/mydoc` klasöründe bulunmaktadır. Yeni sayfa eklemek isteyen kullanıcılar öncelikle bu klasörde aşağıda belirtilen formatta bir dosya oluşturmalıdır.

Oluşturulan dosyanın isim formatı:

```text
dosya_ismi.md      Örnek: mydoc_dosya_konumlari.md
```

{% include note.html content=" Markdown, okunması ve yazması kolay bir düz metin formatıdır. Projedeki markdown dosyaları HTML'e dönüştürülerek web sayfasına konulmaktadır. Markdown hakkında daha fazla bilgiye [Mastering Markdown](https://guides.github.com/features/mastering-markdown/) sayfasından ulaşabilirsiniz." %}

Eklenecek Markdown dosyası ilgili sayfaya göre aşağıdaki formatta başlamalıdır.

```text
---
title: "Dosya Konumları"
tags: [PostgreSQL]
keywords: postgres
last_updated: December 31, 2020
sidebar: mydoc_sidebar
permalink: mydoc_dosya_konumlari.html
folder: mydoc
---
```

- **title, keywords, tags:** İlgili sayfanın konusuna göre belirlenir.
- **last_updated:** dosyanın yaratıldığı tarih girilir.
- **sidebar:** `mydoc_sidebar` olmak **zorundadır**.
- **permalink:** Markdown dosyasının ismi (.md hariç) sonuna '.html' konarak eklenir.
Örnek:

```text
dosya_konumlari.md ---> dosya_konumlari.html
```

- **folder:** `mydoc` olarak belirtilir.

Eklenecek sayfayı tanımlayan sayfa üst bilgisi bu şekilde eklendikten sonra içerik kısmı Markdown formatında yazılır.

Dosya yukarda belirtilen şekilde uluşturulup kaydedildikten sonra soldaki panele eklenmesi gerekmektedir. Bunun için `_data\sidebars\mydoc_sidebar.yml` dosyasına gidilir. Bu dosyanın özel bir formatı vardır ve bu formata uymak **zorunludur**. Eğer bir indentation hatası bulunursa konsolda hata mesajı verilir ve web sitesi çalışmayı durdurur. Her bir alt bölmeye geçildiğinde üsttekine göre bir boşluk bırakılır ve istenilen sayfa uygun başlık altına eklenir.

```text
  - title: Veritabanı Yapılandırması
    output: web, pdf
    folderitems:

    - title: Parametreleri Ayarlama
      url: /mydoc_paremetreleri_ayarlama.html
      output: web, pdf

    - title: Dosya Konumları
      url: /mydoc_dosya_konumlari.html
      output: web, pdf

    - title: Bağlantılar ve Kimlik Doğrulama
      url: /mydoc_baglantilar_kimlik_dogrulama.html
      output: web, pdf

    - title: Kaynak Tüketimi
      url: /mydoc_kaynak_tuketimi.html
      output: web, pdf

    - title: Write Ahead Log
      url: /mydoc_wal.html
      output: web, pdf

    - title: Replication
      url: /mydoc_replication.html
      output: web, pdf
```

- **title:** Eklenen sayfanın yan panelde görüneceği başlık (title) belirlenir.
- **URL:** Markdown dosyasının ismi (.md hariç) sonuna .html uzantısı, başına `/` eklenerek yazılır.

```text
mydoc_dosya_konumlari.md --> /mydoc_dosya_konumlari.html
```

- **Output** kısmına ise web, pdf yazılılarak dosyada uygun başlık altına eklenir.

```text
- title: Veri Tipleri
  url: /veri_tipleri.html
  output: web, pdf
```

---

### Mevcut Sayfaları Düzenleme

**1.** Düzenlemek istediğiniz sayfada **`Düzeltme Öner`** butonuna tıklanır.
{% include image.html file="contribution-1.png" alt="Düzeltme Öner" %}

**2.** Sayfa açıldıktan sonra sağ üstteki kalem ikonuna basılır.
{% include image.html file="contribution-2.png" alt="Kalem" %}

**3.** Metin editörü açıldıktan sonra istenilen değişiklikler yapılır. Bu değişikliklerin önizlemesi üstte bulunan **`Preview Changes`** kısmında görülebilir. Değişiklikler yapıldıktan sonra açıklayıcı bir mesaj yazılır ve **`Propose Changes`** butonuna basılır.
{% include image.html file="contribution-3.png" alt="Preview Changes" %}

**4.** Açılan sayfada **`Create Pull Request`** butonuna basılır.
{% include image.html file="contribution-4.png" alt="Create Pull Request" %}

**5.** Açılan pull request sayfasında **`Create Pull Request`** butonuna basılır ve değişiklikler proje yöneticisine gönderilir.
{% include image.html file="contribution-5.png" alt="Send Request" %}

**6.** Yapılan değişiklikler **proje yöneticisi** tarafından incelenerek kabul edilir veya geri çevrilir.

--- 

## Sayfa İçeriği Formatı

### Uyarı Metni Türleri

- note.html
- tip.html
- warning.html
- important.html

Bu uyarı türleri farklı bir renk, simge ve uyarı kelimesine sahip olmaları dışında aynı işlevi görürler. Ekleyeceğiniz yazılarda istediğiniz şablonunu seçip kullanarak yazılarınızı daha renkli ve dikkat çekici hale getirebilirsiniz. Birden çok paragrafa ihtiyacınız varsa, `<br/><br/>` etiketleri girin.

```liquid
{% raw %}{% include note.html content="Notunuzu buraya ekleyin." %}{% endraw %}
```

```liquid
{% raw %}{% include tip.html content="Buraya ipucunu ekle." %}{% endraw %}
```

```liquid
{% raw %}{% include important.html content="Önemli bilgilerinizi buraya ekleyin." %}{% endraw %}
```

```liquid
{% raw %}{% include warning.html content="Uyarınızı buraya ekleyin." %}{% endraw %}
```

Bu şablonların çıktıları şu şekilde olacaktır:

{% include note.html content="Notunuzu buraya ekleyin." %}

{% include tip.html content="Buraya ipucunu ekle." %}

{% include important.html content="Önemli bilgilerinizi buraya ekleyin." %}

{% include warning.html content="Uyarınızı buraya ekleyin." %}

---

### Açıklama Metinleri

Her biri farklı açıklama metni türüne sahip şablonları ile ekleyeceğiniz yazıları daha renki ve dikkat çekici yapabilirsiniz.

```liquid
{% raw %}{% include callout.html content="This is my **danger** type callout. It has a border on the left whose color you define by passing a type parameter." type="danger" %}{% endraw %}
```

```liquid
{% raw %}{% include callout.html content="This is my **default** type callout. It has a border on the left whose color you define by passing a type parameter." type="default" %}{% endraw %}
```

```liquid
{% raw %}{% include callout.html content="This is my **primary** type callout. It has a border on the left whose color you define by passing a type parameter." type="primary" %}{% endraw %}
```

```liquid
{% raw %}{% include callout.html content="This is my **success** type callout. It has a border on the left whose color you define by passing a type parameter." type="success" %}{% endraw %}
```

```liquid
{% raw %}{% include callout.html content="This is my **info** type callout. It has a border on the left whose color you define by passing a type parameter." type="info" %}{% endraw %}
```

```liquid
{% raw %}{% include callout.html content="This is my **warning** type callout. It has a border on the left whose color you define by passing a type parameter." type="warning" %}{% endraw %}
```

Bu şablonların çıktıları şu şekilde olacaktır:

{% include callout.html content="This is my **danger** type callout. It has a border on the left whose color you define by passing a type parameter." type="danger" %}

{% include callout.html content="This is my **default** type callout. It has a border on the left whose color you define by passing a type parameter." type="default" %}

{% include callout.html content="This is my **primary** type callout. It has a border on the left whose color you define by passing a type parameter." type="primary" %}

{% include callout.html content="This is my **success** type callout. It has a border on the left whose color you define by passing a type parameter." type="success" %}

{% include callout.html content="This is my **info** type callout. It has a border on the left whose color you define by passing a type parameter." type="info" %}

{% include callout.html content="This is my **warning** type callout. It has a border on the left whose color you define by passing a type parameter." type="warning" %}

---

### Resim Ekleme

Dokümantasyon projesinde yer alan sayfalara eklenecek resimler **`images/`** klasörü altında toplanmaktadır. Yazdığınız yazılara ekleyeceğiniz resimleri uygun bir isimle bu klasöre ekledikten sonra aşağıdaki formatta kullanabilirsiniz.

```liquid
{% raw %}{% include image.html file="contribution-postgresql.png" alt="PostgreSQL" caption="Bu bir örnek resim açıklaması" %}{% endraw %}
```

{% include image.html file="contribution-postgresql.png" alt="PostgreSQL" caption="Bu bir örnek resim açıklaması" %}

---

### Kod Örnekleri

    ```sql
    CREATE TABLE measurement (
        city_id         int not null,
        logdate         date not null,
        peaktemp        int,
        unitsales       int
    ) PARTITION BY RANGE (logdate);
    ```

**Sonuç:**

```sql
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);
```

Desteklenen dillerin listesi için [bkz](https://github.com/rouge-ruby/rouge/wiki/list-of-supported-languages-and-lexers)

---

### Bağlantılar

**Dış bağlantı oluşmak için:**

```text
[PostgreSQL Server Administration](https://www.postgresql.org/docs/13/admin.html)
```

[PostgreSQL Server Administration](https://www.postgresql.org/docs/13/admin.html)

**Mevcut sayfalara bağlantı oluşturmak için:**

```text
[Sorguların İşlenmesi](mydoc_query_processing.html)
```

[Sorguların İşlenmesi](mydoc_query_processing.html)

{% include callout.html content=" Dosya adını değiştirirseniz, tüm bağlantılarınızı güncellemeniz gerekir." type="warning" %}

---

### Tablo Oluşturma

Eklenecek yazılarda tablo oluşturmak için Multimarkdown sözdizimini kullanabilirsiniz. Aşağıda bir örnek gösterilmektedir:

```text
| Dosya | Tanım |
|-------|------|
| **PG_VERSION** | PostgreSQL'in majör sürüm numarasını içeren dosya |
| **pg_hba.conf** | PosgreSQL istemci kimlik doğrulaması kontrolü dosyası |
| **pg_ident.conf** | PostgreSQL user name mapping kontrol dosyası |
| **postgresql.conf** | PostgreSQL yapılandırma parametrelerini ayarlama dosyası |
| **postgresql.auto.conf** | ALTER SYSTEM ile ayarlanan yapılandırma parametrelerini depolamak için kullanılan dosya |
| **postmaster.opts** | Sunucunun en son başlatıldığı komut satırı seçeneklerini kaydeden dosya |
```

| Dosya | Tanım |
|-------|------|
| **PG_VERSION** | PostgreSQL'in majör sürüm numarasını içeren dosya |
| **pg_hba.conf** | PosgreSQL istemci kimlik doğrulaması kontrolü dosyası |
| **pg_ident.conf** | PostgreSQL user name mapping kontrol dosyası |
| **postgresql.conf** | PostgreSQL yapılandırma parametrelerini ayarlama dosyası |
| **postgresql.auto.conf** | ALTER SYSTEM ile ayarlanan yapılandırma parametrelerini depolamak için kullanılan dosya |
| **postmaster.opts** | Sunucunun en son başlatıldığı komut satırı seçeneklerini kaydeden dosya |

---

### Yazı Vurgulama

```text
*Bu yazı italik yazılır*

_Bu yazı da italik yazılır_

**Bu yazı kalın yazılır**

__Bu yazı da kalın yazılır__

`Vurgulu yazı`

**`Vurgulu yazı`**
```

*Bu yazı italik yazılır*

_Bu yazı da italik yazılır_

**Bu yazı kalın yazılır**

__Bu yazı da kalın yazılır__

`Vurgulu yazı`

**`Vurgulu yazı`**

---

### Paragraflar

Markdown'da paragraf yaratmak için metinler arasında bir ya da daha fazla boş satır bırakın.

```text
Örnek:
Markdown kullanması çok basit bir metin formatıdır.

Sanırım gelecekte yazacağım bütün dokümanlarda Markdown kullanacağım.
```

---

### Başlıklar

```text
# En büyük başlık (HTML'de <h1>)
## Biraz daha küçük (HTML'de <h2>)
###### En küçük başlık (HTML'de <h6>)
```

```markdown
Çıktı şu şekilde gözükecek:
# En büyük başlık
## Biraz daha küçük
###### En küçük başlık
```

---

### Listeler

#### Sırasız

```text
* Dosya 1
* Dosya 2
  * Dosya 2a
  * Dosya 2b
```

Çıktı şu şekilde gözükecek:

* Dosya 1
* Dosya 2
  * Dosya 2a
  * Dosya 2b

#### Sıralı

```text
1. Dosya 1
2. Dosya 2
3. Dosya 3
   1. Dosya 3a
   2. Dosya 3b
```

Çıktı şu şekilde gözükecek:

1. Dosya 1
2. Dosya 2
3. Dosya 3
   1. Dosya 3a
   2. Dosya 3b

---

### Alıntılar

```text
Örnek:
Atatürk'ün dediği gibi:

> Hayatı ve özgürlüğü için ölümü 
>
> göze alan bir millet asla yenilmez.
```

Çıktı şu şekilde gözükecek:

Atatürk'ün dediği gibi:

> Hayatı ve özgürlüğü için ölümü
>
> göze alan bir millet asla yenilmez.

{% include links.html %}
