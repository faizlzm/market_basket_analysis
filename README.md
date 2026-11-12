# Market Basket Analysis: Pola Belanja Groceries dengan Apriori

Analisis 14.963 keranjang belanja dari toko groceries (2014–2015) dengan algoritma Apriori di Python dan dashboard Tableau, untuk menemukan produk yang sering dibeli bersamaan dan menurunkannya menjadi ide penataan rak serta promo.

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Tujuan Analisis](#tujuan-analisis)
- [Struktur Repositori](#struktur-repositori)
- [Dataset](#dataset)
- [Metodologi](#metodologi)
- [Temuan Utama](#temuan-utama)
- [Rekomendasi](#rekomendasi)
- [Keterbatasan](#keterbatasan)
- [Cara Menjalankan](#cara-menjalankan)
- [Penulis](#penulis)

---

## Latar Belakang

Data transaksi kasir menyimpan petunjuk tentang produk pelengkap yang bisa dimanfaatkan untuk memperbesar nilai keranjang belanja. Market Basket Analysis (MBA) menjawab pertanyaan sederhana: *ketika pelanggan membeli produk A, seberapa besar kemungkinan mereka juga membeli produk B?* Hasilnya bisa dipakai untuk menata rak, merancang paket, dan menguji promo silang.

## Tujuan Analisis

1. Memahami karakteristik keranjang belanja, produk terlaris, dan pola waktu belanja.
2. Menemukan pasangan produk yang dibeli bersamaan lebih sering daripada kebetulan.
3. Menyusun rekomendasi penataan rak dan promo berbasis pola tersebut.

## Struktur Repositori

```text
market_basket_analysis/
├── groceries_dataset.csv                       # Dataset mentah (38.765 baris, 3 kolom)
├── market_basket_analysis_main.ipynb           # Preprocessing, EDA, Apriori, dan association rules
├── MARKET BASKET ANALYSIS.twbx                 # Dashboard Tableau (9 worksheet)
├── ahmad_faiz_ali_azmi_market_basket_analysis.pdf   # Laporan presentasi (19 slide)
└── README.md
```

Worksheet Tableau: Kombinasi Produk Terbaik, Peta Peluang Cross-Selling, Rata-rata Keranjang, Top 10 Produk, Total Pelanggan, Total Produk, Total Transaksi, Transaksi per Hari, dan Tren Bulanan.

## Dataset

Riwayat transaksi *point of sale* sebuah toko groceries.

- **Jumlah baris:** 38.765 (satu baris per item yang dibeli)
- **Jumlah kolom:** 3
- **Periode:** 1 Januari 2014 – 30 Desember 2015
- **Cakupan:** 3.898 member dan 167 produk unik

| Kolom | Keterangan |
| --- | --- |
| `Member_number` | ID unik member |
| `Date` | Tanggal pembelian (format `DD-MM-YYYY`) |
| `itemDescription` | Nama produk yang dibeli |

Dataset tidak memiliki nomor struk, sehingga **satu keranjang didefinisikan sebagai semua item yang dibeli satu member pada tanggal yang sama** (`Transaction_ID = Member_number + Date`).

## Metodologi

### 1. Data Preprocessing

| Pemeriksaan | Hasil | Tindakan |
| --- | --- | --- |
| Missing value | 0 | Tidak perlu penanganan |
| Tipe data | `Member_number` berupa angka, `Date` berupa teks | Dikonversi ke string dan datetime |
| Konsistensi teks | `cream cheese ` (358 baris) dan `roll products ` (82 baris) memiliki spasi di akhir | `strip` dan `lower`; jumlah produk tetap 167 |
| Duplikat | 759 baris (item sama dibeli lebih dari sekali pada hari yang sama) | **Dihapus**, karena MBA hanya melihat ada atau tidaknya produk di keranjang |
| Outlier | Keranjang dengan ≥ 5 item dan member dengan 10–11 kunjungan | **Dipertahankan** karena mencerminkan belanja mingguan atau bulanan yang wajar |

Hasil: **38.006 baris**, 3.898 member, **14.963 keranjang unik**.

### 2. Feature Engineering

`Transaction_ID` (member + tanggal), `Basket_Size` (jumlah produk berbeda per keranjang), serta `Day_Name`, `Month`, dan `Year`.

### 3. Exploratory Data Analysis

Distribusi dan uji normalitas D'Agostino-Pearson untuk ukuran keranjang dan frekuensi belanja, 10 produk terlaris, serta pola harian dan bulanan.

### 4. Market Basket Analysis

- Keranjang di-*one-hot encode* dengan `TransactionEncoder`.
- Apriori dengan **`min_support = 0,2%`**: dipilih karena produk terlaris pun hanya muncul di 15,8% keranjang, sehingga pasangan produk wajar berada di kisaran 0,2%–2%. Hasilnya 330 itemset (126 produk tunggal dan 204 pasangan).
- Association rules difilter dengan **lift > 1**.

| Metrik | Arti |
| --- | --- |
| Support | Seberapa sering kombinasi muncul di seluruh keranjang |
| Confidence | Jika membeli A, peluang membeli B |
| Lift | Rasio terhadap kebetulan; > 1 berarti dibeli bersama lebih sering dari yang diharapkan |

## Temuan Utama

> Angka dicocokkan ulang dengan menjalankan ulang analisis dari `groceries_dataset.csv`.

### 1. Keranjang kecil, pelanggan berkunjung 2–5 kali

- Rata-rata 2,54 item per keranjang (median 2). **67%** keranjang berisi tepat 2 item dan **85%** berisi 2–3 item; hanya 746 keranjang (5%) berisi ≥ 5 item.
- Frekuensi belanja per member: rata-rata 3,84 kunjungan (median 4, maksimum 11).
- Kedua distribusi **tidak normal** (p < 0,05); Apriori memang tidak mensyaratkan asumsi distribusi.

### 2. Kebutuhan dapur harian mendominasi

| Produk | Keranjang | Penetrasi |
| --- | ---: | ---: |
| Whole milk | 2.363 | 15,8% |
| Other vegetables | 1.827 | 12,2% |
| Rolls/buns | 1.646 | 11,0% |
| Soda | 1.453 | 9,7% |
| Yogurt | 1.285 | 8,6% |

### 3. Kunjungan relatif merata sepanjang minggu

Jumlah keranjang per hari berkisar **2.067–2.188**, dengan puncak Kamis (2.188) dan Sabtu (2.161), dan terendah Senin. Selisih antar hari hanya sekitar 6%.

Per bulan: 539–711 keranjang, dengan puncak Mei 2014 (711). **Tahun 2015 lebih rendah dari 2014** (6.982 vs 7.981 keranjang, turun 12,5%).

### 4. Aturan asosiasi: ada, tetapi lemah

Apriori menghasilkan **36 aturan dengan lift > 1**, yang merupakan **18 pasangan produk** (setiap pasangan muncul dua arah). Support semua aturan di bawah 0,6%.

| Pasangan | Support | Confidence | Lift | Keranjang bersama |
| --- | ---: | ---: | ---: | ---: |
| Curd → Sausage | 0,29% | 8,7% | 1,45 | 44 |
| Brown bread → Canned beer | 0,24% | 6,4% | 1,36 | 36 |
| Frozen vegetables → Sausage | 0,21% | 7,4% | 1,23 | 31 |
| Bottled beer → Sausage | 0,33% | 7,4% | 1,22 | 50 |
| Frankfurter → Other vegetables | 0,51% | 13,6% | 1,12 | 77 |

- **Sausage** menjadi poros: muncul di 3 dari 4 pasangan teratas, padahal hanya ada di sekitar 6% keranjang. Ia produk yang sering dipasangkan, bukan yang terlaris.
- Lift tertinggi hanya 1,45, jadi asosiasinya **moderat**.

### 5. Produk terlaris tidak saling melengkapi

Whole milk dan other vegetables tidak membentuk pasangan kuat. Pasangan whole milk dengan produk lain berlift sekitar 1 atau di bawahnya (maksimum 1,01), dan pasangan whole milk–other vegetables justru berlift **0,77** (222 keranjang bersama dibanding 289 yang diharapkan jika independen). Popularitas keduanya berdiri sendiri-sendiri.

## Rekomendasi

1. **Dekatkan pasangan berlift tinggi.** Tempatkan curd dan sausage di chiller yang bersebelahan, serta frankfurter di dekat area sayuran segar, sebagai uji coba tata letak.
2. **Uji brown bread dan canned beer** dengan display bersama.
3. **Kupon silang bersyarat.** Pembeli curd mendapat voucer diskon sausage pada kunjungan berikutnya.
4. **Jadikan semua ini uji coba (A/B), bukan klaim pasti,** karena lift dan support rendah. Ukur kenaikan penjualan pasangan produk sebelum diperluas.
5. **Jangan bundling whole milk dengan other vegetables.** Datanya tidak mendukung.

## Penulis

**Ahmad Faiz Ali Azmi**
Lulusan Teknik Informatika, Universitas Brawijaya. Peserta Offline Bootcamp Data Analyst Batch 4 di dibimbing.id.

GitHub: [@faizlzm](https://github.com/faizlzm)
