# SparkSQL Window Functions · Olist demosu

*VBO Data Engineering Bootcamp · 8. hafta*

Sunumda window function ailelerini gördük: sıralama, değer erişimi, aggregate + frame. Bu demoda bir e-ticaret verisinde, her soru için **doğru fonksiyonu, partition'ı ve sıralamayı seçerek** cevap arıyoruz:

> *Müşteri ikinci kez geliyor mu, ne zaman geliyor? Her kategoride hangi satıcılar öne çıkıyor?*

| Perde | Soru | Fonksiyon |
|---|---|---|
| 1 | Müşterinin kaçıncı siparişi? | `ROW_NUMBER` |
| 2 | İki sipariş arası kaç gün? | `LAG` |
| 3 | Aylık ciro: running total, 3 aylık ortalama | `SUM/AVG OVER` + `ROWS BETWEEN` |
| 4 | Kategori içinde ilk 3 satıcı | `DENSE_RANK` |
| 5 | Harcamaya göre müşteri çeyrekleri | `NTILE(4)` |

## Gereksinimler

- Docker Desktop (veya Docker Engine + Compose v2), Docker'a en az 4 GB RAM.
- Python 3.9+ (yalnızca veriyi Kaggle'dan indirmek için).
- Boş portlar: 8888 (JupyterLab), 4040 (Spark UI).

## Kurulum

### 1. Kaggle API token'ı

1. [kaggle.com](https://www.kaggle.com) → profil → **Settings** → **API** → **Create New Token**.
2. İnen `kaggle.json` dosyasını yerine koy:

   ```bash
   mkdir -p ~/.kaggle
   mv ~/Downloads/kaggle.json ~/.kaggle/
   chmod 600 ~/.kaggle/kaggle.json
   ```

   Windows'ta konum `C:\Users\<kullanıcı>\.kaggle\kaggle.json`.

### 2. Veriyi indir

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (~43 MB zip):

```bash
pip install kaggle          # veya: pipx install kaggle / uv tool install kaggle
kaggle datasets download olistbr/brazilian-ecommerce --unzip -p notebooks/datasets/olist
```

`notebooks/datasets/olist/` altında 9 CSV oluşur; notebook bunlardan 6'sını kullanır: orders, customers, order_items, products, product_category_name_translation, order_payments.

> Veri **CC BY-NC-SA 4.0** lisanslıdır, bu yüzden repoya konmaz (`notebooks/datasets/` `.gitignore`'da).

### 3. Ortamı başlat

```bash
docker compose up -d
```

Compose `vbo-pyspark:4.1.3_python-3.12_java21` imajını kullanır (Spark 4.1.3, Python 3.12, Java 21). İmaj makinende yoksa `image/` altındaki Dockerfile'dan kur:

```bash
docker compose up -d --build     # ilk build ~25-30 dk, sonrakiler saniyeler
```

| Adres | Ne |
|---|---|
| http://localhost:8888 | JupyterLab (token yok, yalnızca bu makineden erişilir) |
| http://localhost:4040 | Spark UI (notebook'ta SparkSession açıldıktan sonra) |

## Çalıştırma

JupyterLab'de `01_window_functions_solution.ipynb` → **Run → Run All Cells**. Her perde aynı ritimde ilerler: iş sorusu → hangi fonksiyon, `PARTITION BY`, `ORDER BY`? → sorgu → **sağlama** (sonucu bağımsız bir yoldan doğrulayan `assert`).

`01_window_functions_live.ipynb` aynı başlıklarla, boş kod hücreleriyle gelir (yalnızca kurulum hücresi dolu). Canlı derste kodlar dolu defterden hücre hücre yapıştırılır; kendin tekrar ederken önce kendin yazmayı dene.

Terminalden baştan sona çalıştırmak için (~30 sn):

```bash
docker compose exec spark jupyter nbconvert --to notebook --execute \
  01_window_functions_solution.ipynb --output /tmp/run.ipynb
```

Hatasız biterse tüm sağlamalar geçmiştir.

## Bilmekte fayda var

- **`customer_id` ≠ müşteri.** Olist `customer_id`'yi her sipariş için yeniden üretir; gerçek kişi `customer_unique_id`. Perde 1 bu tuzağı bilerek gösterir.
- **Eşit ciro.** Gerçek veride bir kategorinin ilk 3'ünde kuruşu kuruşuna eşit ciro yok. Perde 4 ciroyu yüz BRL'ye yuvarlar (`ROUND(revenue, -2)`); `ROW_NUMBER` ile `DENSE_RANK` farkı bu eşitlikten görünür.
- **`QUALIFY` yok.** Databricks, Snowflake ve BigQuery'deki `QUALIFY rnk <= 3` açık kaynak Spark 4.1.3'te `PARSE_SYNTAX_ERROR` verir. Window sonucunu filtrelemek için alt sorgu/CTE + `WHERE` kullanılır (Perde 4'teki hücre bunu doğrular).
- **Window = shuffle.** `PARTITION BY` planda `Exchange hashpartitioning(...)` + `Window` operatörüne dönüşür. Spark UI → **SQL / DataFrame** sekmesinde görebilirsin. `PARTITION BY` olmayan window (Perde 3 ve 5) tüm veriyi tek partition'a toplar.

## Kapatma

```bash
docker compose down
```

## Atıf

Veri: Olist, *Brazilian E-Commerce Public Dataset by Olist*, Kaggle, [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
