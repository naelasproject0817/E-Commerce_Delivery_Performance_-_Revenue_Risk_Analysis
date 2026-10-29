# 📦 E-Commerce Delivery Performance & Revenue Risk Analysis

Analisis data pengiriman e-commerce untuk mengidentifikasi akar penyebab keterlambatan pengiriman, membuktikan dampaknya terhadap rating pelanggan dan tingkat retur secara statistik, serta menyusun rekomendasi dan simulasi *what-if* untuk menekan risiko kebocoran pendapatan.

---

## 📌 Problem Statement

Perusahaan mengalami keterlambatan pengiriman dari estimasi yang telah ditentukan, dengan **late rate mencapai 56,32%** dan rata-rata keterlambatan **1,28 hari** dari waktu yang dijanjikan. Tingkat keterlambatan ini **konsisten tinggi sepanjang Januari–Desember 2026** (kisaran 55,2%–57,7%), menunjukkan ini adalah masalah operasional struktural, bukan anomali musiman.

Keterlambatan ini terbukti secara statistik berhubungan signifikan dengan:
- **Penurunan rating pelanggan** (korelasi Pearson r = -0,508, p < 0,001)
- **Peningkatan permintaan retur** (23,34% pada pesanan terlambat vs 9,39% pada pesanan tepat waktu, Chi-Square p < 0,001)

<p align="center">
  <img src="images/01_late_rate_by_month.png" width="48%">
  <img src="images/05_return_rate_by_status.png" width="48%">
</p>

yang berpotensi menggerus pendapatan hingga **$1,17 juta** dari nilai pesanan yang diretur akibat keterlambatan.

Perusahaan perlu mengetahui **root cause** apa yang paling banyak menyumbang keterlambatan ini, agar dapat memprioritaskan perbaikan operasional dan menekan risiko kehilangan pendapatan tersebut.

---

## ❓ Business Questions

1. Bagaimana performa ketepatan waktu pengiriman secara keseluruhan?
2. Apakah keterlambatan meningkatkan risiko retur, dan berapa besar nilai pendapatan yang berisiko?
3. Kondisi cuaca apa yang paling berkontribusi terhadap keterlambatan?
4. Carrier mana yang paling sering terlambat, dan apakah itu berdampak pada rating pelanggan?

---

## 🗂️ Dataset

| | |
|---|---|
| **Sumber** | [E-Commerce Delivery, Logistics & Shipping Analytics Dataset (2026)](https://www.kaggle.com/datasets/datascikhan/e-commerce-delivery-and-shipping-data-2026) — Kaggle |
| **Ukuran** | 50.000 baris × 30 kolom (37 kolom setelah feature engineering) |
| **Periode** | Januari – Desember 2026 |
| **Cakupan** | 17 negara pelanggan, 10 gudang, 8 perusahaan carrier |

> ⚠️ **Catatan penting:** Dataset ini bersifat **sintetis/simulasi**, bukan transaksi nyata. Seluruh insight pada project ini ditulis sebagai temuan dalam dataset ini, bukan generalisasi terhadap industri e-commerce secara umum.

### Keterbatasan Data yang Teridentifikasi
- `customer_id` **tidak konsisten** mewakili satu pelanggan (99,4% ID muncul di >1 negara berbeda) → tidak digunakan untuk analisis tingkat pelanggan.
- `shipping_cost_usd` memiliki batas atas (*cap*) di $500 (658 baris) → kemungkinan nilai buatan, bukan biaya asli.
- `return_reason` kosong untuk pesanan yang tidak diretur (82,75% baris) → diisi `"Not Returned"`.
- Kategori `Failed Delivery` pada `delivery_status` hanya berisi 3 dari 50.000 pesanan → tidak dijadikan basis kesimpulan karena ukuran sampelnya terlalu kecil.

---

## 🔍 Metodologi

Alur analisis mengikuti pendekatan end-to-end data analytics:

```
Business Understanding → Data Understanding → Data Cleaning & Preprocessing
    → Exploratory Data Analysis (EDA) → Statistical Testing
    → Root Cause Analysis → What-If Simulation → Dashboard & Recommendation
```

**Tools yang digunakan:** Python (pandas, numpy, seaborn, matplotlib, scipy), Power BI, Google Colab.

### 1. Data Cleaning & Preprocessing
- Perbaikan tipe data (`order_date` → datetime, `late_delivery`/`return_requested` → boolean, kolom kategorikal → category).
- Validasi duplikat (baris utuh & kombinasi kunci bisnis) — 0 duplikat ditemukan.
- Validasi konsistensi `customer_id` dan `warehouse_id`.
- Feature engineering: `order_month`, `route`, `value_at_risk`, `shipping_cost_ratio`, `delay_bucket`, `order_value_segment`, `order_value_capped`, `distance_group`.

### 2. Exploratory Data Analysis (EDA)
- Distribusi nilai pesanan, rating pelanggan, late rate, dan return rate.
- Analisis bivariat: keterlambatan vs rating, keterlambatan vs carrier, keterlambatan vs cuaca.
- Tren late rate bulanan (Januari–Desember) untuk menguji apakah masalah ini musiman atau struktural.

### 3. Statistical Testing (α = 0,05)

| Uji | Tujuan | Hasil | Kesimpulan |
|---|---|---|---|
| **Korelasi Pearson** | Hubungan lama keterlambatan dengan rating pelanggan | r = -0,508, p < 0,001 | Hubungan negatif signifikan |
| **Independent T-Test** | Perbedaan rata-rata rating: pesanan terlambat (2,73) vs tepat waktu (4,21) | t = -249,05, p < 0,001 | Perbedaan signifikan & besar secara praktis |
| **Chi-Square Test** | Hubungan status keterlambatan dengan permintaan retur | χ² = 1.676,28, df = 1, p < 0,001 | Keterlambatan & retur berhubungan signifikan, bukan kebetulan |

<p align="center">
  <img src="images/02_rating_late_vs_ontime.png" width="60%">
</p>
<p align="center"><i>Distribusi rating pelanggan: pesanan terlambat (kiri) vs tepat waktu (kanan)</i></p>

### 4. Root Cause Analysis
Dua faktor utama yang diidentifikasi sebagai pendorong keterlambatan:

- **Jarak pengiriman** — late rate melonjak tajam di atas 3.000 km, dari ~50% menjadi 75–83%.
- **Performa carrier** — matrix *carrier × kategori jarak* mengonfirmasi SwiftShip unggul **konsisten di setiap kategori jarak** (bukan kebetulan mendapat rute pendek), sementara GlobalExpress konsisten terburuk di semua kategori.
- **Cuaca ekstrem** (Snow, Storm) menaikkan late rate signifikan, sementara estimasi waktu pengiriman sistem tetap kaku tanpa penyesuaian kondisi cuaca.

<p align="center">
  <img src="images/03_late_rate_by_carrier.png" width="48%">
  <img src="images/04_late_rate_by_distance.png" width="48%">
</p>
<p align="center">
  <img src="images/06_matrix_carrier_distance.png" width="75%">
</p>
<p align="center"><i>SwiftShip (hijau) konsisten unggul di setiap kategori jarak, GlobalExpress (merah) konsisten terburuk — membuktikan ini bukan kebetulan rute.</i></p>

### 5. What-If Simulation — Dynamic SLA
Simulasi penambahan buffer waktu janji pengiriman (+2 hari) khusus untuk pesanan berjarak >3.000 km:

| Metrik | Sebelum | Sesudah Simulasi |
|---|---|---|
| Late rate segmen >3.000 km | 77,65% | 32,94% |
| **Late rate keseluruhan** | **56,32%** | **≈ 41,37%** |
| Pesanan berubah status (telat → tepat waktu) | — | ≈ 7.475 pesanan/tahun |

> Simulasi ini **tidak mempercepat waktu pengiriman aktual** — yang dikoreksi adalah estimasi janji yang semula tidak realistis untuk jarak jauh, sehingga ekspektasi pelanggan menjadi lebih akurat sejak awal.

<p align="center">
  <img src="images/07_whatif_dynamic_sla.png" width="75%">
</p>

---

## 📊 Dashboard

Dashboard interaktif dibuat di **Power BI**, terdiri dari 2 halaman:
- **Halaman 1 — Overview Performance:** KPI utama, tren bulanan, gap janji vs aktual, alasan retur, dampak cuaca.
- **Halaman 2 — Root Cause:** performa carrier vs rating, late rate per jarak, return rate per status keterlambatan, matrix carrier × jarak.

*(Tambahkan screenshot atau link Power BI publish di sini)*

---

## 💡 Key Findings

1. Late rate **56,32%** konsisten sepanjang tahun (selisih min-max hanya 2,5 poin persentase) — masalah struktural, bukan musiman.
2. Keterlambatan terbukti signifikan menurunkan rating pelanggan dan meningkatkan permintaan retur (keduanya diuji secara statistik).
3. 9,3% pesanan bernilai tinggi (>$500) menyumbang **49,3%** dari total pendapatan — keterlambatan pada segmen kecil ini berdampak tidak proporsional terhadap pendapatan berisiko.
4. "Late Delivery" hanya menjadi alasan retur eksplisit pada 12,38% kasus (terendah dari 6 kategori) — mengindikasikan keterlambatan kemungkinan memengaruhi retur secara tidak langsung, lewat penurunan kepuasan, bukan selalu menjadi alasan retur yang dicatat.
5. Jarak pengiriman dan performa carrier adalah dua *root cause* utama; cuaca ekstrem menjadi faktor pendukung yang signifikan.

---

## ✅ Recommendations

1. **Audit & evaluasi kontrak carrier berisiko** (GlobalExpress, PrimeDelivery, BlueRoute) — bandingkan performa pada rute setara sebelum mengambil keputusan kontrak.
2. **Terapkan Dynamic SLA** dengan buffer waktu otomatis: +2 hari untuk jarak >3.000 km, +3 hari saat kondisi cuaca ekstrem.
3. Jadikan baseline late rate hasil simulasi (**≈41,4%**) sebagai target SOP tahap awal (*Quick Win*), dilanjutkan realokasi volume carrier pada tahap berikutnya.
4. **Prioritaskan penanganan pesanan bernilai tinggi** (>$500) dengan SLA dan packaging khusus.
5. **Perbaiki kualitas kemasan dan produk di gudang** — Product Defect dan Damaged Package menyumbang gabungan ~43% dari seluruh alasan retur.

---

## 🧰 Tech Stack

- **Python** — pandas, numpy, scipy (statistical testing), seaborn, matplotlib
- **Power BI** — dashboard interaktif & DAX measures
- **Google Colab** — environment analisis

---

## 📁 Repository Structure

```
├── Naela_Arbach_Final_Project_E_Commerce.ipynb   # Notebook analisis lengkap (cleaning → EDA → stat test → simulasi)
├── E-commerce_Delivery_Shipping_Data_2026.csv     # Dataset mentah
├── cleaned_ecommerce_delivery.csv                 # Dataset hasil cleaning & feature engineering
├── Late_Rate_pesanan_terlambat.pptx               # Slide deck presentasi
├── images/                                        # Chart hasil analisis (dipakai di README ini)
└── README.md
```

---

## 👤 Author

**Naela Arbach**
Career transition ke Data Analytics — latar belakang operasional ticketing & akuntansi.

---

> ⚠️ Dataset yang digunakan bersifat sintetis dan dibuat untuk tujuan pembelajaran/portofolio. Seluruh angka dan simulasi pada repository ini merefleksikan pola dalam dataset tersebut, bukan kondisi operasional perusahaan e-commerce nyata.
