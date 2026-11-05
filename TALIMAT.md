# TALIMAT · SparkSQL Window Functions demosu

Kokpitten (cmlos) yazıldı, 2026-11-05. Bu dizindeki oturumun başlangıç talimatı. Oku, belirsiz bir şey varsa kullanıcıya sor, sonra uygula.

## Bağlam

- Kararlar: https://github.com/cemalcici/cmlos/blob/main/baglam/sparksql-window-functions-dersi.md (yerelde `~/Documents/cmlos/baglam/sparksql-window-functions-dersi.md`). Önce bunu oku; buradaki her karar oradan gelir.
- Proje: https://github.com/cemalcici/cmlos/blob/main/projeler/sparksql-window-dersi.md
- Kaneo görevi: **SWF-1** "Demo ortamını ve notebook'ları hazırla" (cmlos-work). Başlarken Yapılıyor'a çek, bitince Bitti'ye çek ve kısa yorum yaz. Görev açma/düzenleme yapma; yeni iş çıkarsa kullanıcıya "kokpitte görev açılmalı" de.
- Ders: VBO Data Engineering Bootcamp 8. hafta, **bugün (2026-11-05) akşam**, online. Demo ~30 dk, eğitmen canlı yapar; katılımcılar sonra repo ile tekrar eder.

## Amaç

Katılımcı sunumda window function ailelerini (sıralama, değer erişimi, aggregate + frame) gördü. Demoda bir Olist problemini **doğru yöntemi seçerek** çözer. Sunumdaki mini senaryolar (sınav notları, koşu, adım sayısı, fatura) burada tekrar edilmez; demo yalnızca Olist.

İş sorusu: *"Müşteri ikinci kez geliyor mu, ne zaman geliyor? Her kategoride hangi satıcılar öne çıkıyor?"*

## Referanslar (salt okunur, kopyala ama göreli path ile bağlama)

- Ortam kalıbı: `~/Documents/projects/vbo-deb-trino/00_spark_homework_solution/` (compose, live/solution notebook çifti, `nbconvert` doğrulaması, README dili).
- Olist indirme ve README kalıbı: `~/Documents/projects/vbo-deb-trino/trino-federated-demo/` (Kaggle `olistbr/brazilian-ecommerce`, veri repoya konmaz, CC BY-NC-SA 4.0).
- Ders kalıbı: `~/Documents/projects/vbo-deb-trino/PLAN.md`.

## Üretilecekler

1. **`docker-compose.yml`**: yalnızca `spark` servisi, imaj `vbo-pyspark:4.1.3_python-3.12_java21` (bu makinede build edilmiş; build bölümü yedek olarak kalsın). JupyterLab 8888, Spark UI 4040. RustFS ve Postgres yok.
2. **Veri yükleme**: Kaggle'dan indirme adımı + notebook'un ilk hücresinde CSV'leri `createOrReplaceTempView` ile view'a çevirme. Gerekenler: orders, customers, order_items, products, product_category_name_translation, order_payments.
3. **`notebooks/01_window_functions_solution.ipynb`**: 5 perde, sorgular `spark.sql("""...""")` + `.show()`.
4. **`notebooks/01_window_functions_live.ipynb`**: aynı başlıklar, boş kod hücreleri (canlıda dolu defterden yapıştırılır).
5. **`README.md`**: katılımcı için kurulum ve çalıştırma, Türkçe; teknik terimler İngilizce.

## Perdeler (problem sırasıyla, fonksiyon ailesi sırasıyla değil)

Her perdenin ritmi notebook'ta markdown olarak yazılsın: **iş sorusu** → "hangi fonksiyon, partition ne, order ne?" (eğitmen sohbete sorar, bekler) → sorgu → **sağlama** (sonucu doğrulayan küçük bir kontrol ya da sayı).

1. **Müşterinin kaçıncı siparişi** (`ROW_NUMBER`). Yanlış seçim anı: önce `PARTITION BY customer_id` → herkes 1 çıkar (Olist'te `customer_id` sipariş başına üretilir). Sonra `customer_unique_id` ile düzelt. İş cevabı: tekrar gelen müşteri oranı (~%3 görünüyor, doğrula). Düşük oran boş sorgu gibi değil, iş cevabı olarak sunulsun.
2. **İki sipariş arası kaç gün** (`LAG`). Yalnız tekrar gelen müşteriler; medyan/ortalama gün.
3. **Aylık ciro: running total ve 3 aylık hareketli ortalama** (`SUM/AVG OVER` + `ROWS BETWEEN`). `ROWS` ile `RANGE` farkı bir cümlelik notla.
4. **Kategori içinde ilk 3 satıcı** (`DENSE_RANK`). Yanlış seçim anı: önce `ROW_NUMBER` → eşit cirolu satıcı listeden düşer, sonra `DENSE_RANK`. **Gerçek veride eşitlik olduğunu doğrula**; yoksa ciroyu yuvarlayarak (ör. bin BRL) eşitlik üret ve bunu notebook'ta açıkça söyle.
5. **Harcamaya göre müşteri çeyrekleri** (`NTILE(4)`) ve kapanış: dört perdenin iş cevaplarını birleştiren kısa özet hücresi.

Perde sayısı ve sırası sunumun "demo yol haritası" slaytında kullanılacak; değiştirirsen kullanıcıya söyle.

## Doğrula

- **`QUALIFY`**: Spark 4.1.3'te desteklenip desteklenmediğini bu ortamda dene. Sonucu README'ye ya da perde 4'e bir not olarak yaz; sunum oturumu bu sonucu kullanacak (kullanıcıya da bildir).
- Spark UI'da window sorgusunun planında `Window` ve `Exchange` (shuffle) operatörleri görünüyor mu? Görünüyorsa perde 1'e "Spark UI'da bak" notu koy.

## Bitiş ölçütü

- `docker compose up -d` ile ortam kalkıyor.
- `docker compose exec spark jupyter nbconvert --to notebook --execute 01_window_functions_solution.ipynb --output /tmp/run.ipynb` hatasız bitiyor; her perdenin sağlaması `assert` ile.
- Live notebook başlıklarla hazır.
- SWF-1 Bitti'ye çekildi, yorumda QUALIFY sonucu ve eşit ciro durumu yazılı.

## Kapsam dışı

`PERCENT_RANK`, `CUME_DIST`; RustFS/Postgres; mini senaryoların Spark eki; yedek ekran görüntüleri.

## Repo

Dizin henüz git reposu değil. `git init` ve ilk commit bu oturumda yapılır; GitHub reposu (kişisel hesap) açılmadan önce kullanıcıya sor. Bu dosyayı (`TALIMAT.md`) ilk commit'e dahil et.
