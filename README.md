# tokopedia-data-analysis-python
Analisis data e-commerce Tokopedia menggunakan Python di Google Colab untuk mengidentifikasi produk terlaris, penurunan penjualan, tren transaksi unpaid, dan performa harian.
# 🐍 Tokopedia Data Analysis — Python Final Project

Repository ini berisi portofolio proyek akhir (*Final Project*) analisis data menggunakan Python di Google Colab, sebagai bagian dari program Data Analyst di MySkill. Analisis dilakukan pada dataset transaksi e-commerce **Tokopedia** periode tahun 2021 s.d. 2022.

---

## 👥 Anggota Kelompok 4B
* Muhammad Fathi Rizqi
* Ajeng Listya Devani
* Brigita Adven Novani
* Farid Ilham Nugrahatta
* Godefriedo Dimas Putra Brata
* Rifqi Habib
* Anugerah Putra Pratama
* Dini Desita
* Imam Afdol Hakiki Bakri
* Naura Fatin
* Yohanes Dimas Pratama
* Muhammad Hibban
* Siti Aisyah Bintang Larasati

---

## 📊 Pustaka & Penyiapan Data (Libraries & Data Loading)
Proses inisialisasi pustaka pendukung (*libraries*) serta pemuatan seluruh berkas data mentah berformat CSV ke dalam lingkungan kerja Python:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from pandas.tseries.offsets import BDay

# Memuat Dataset Mentah dari Sumber Data
path_od = "[https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/order_detail.csv](https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/order_detail.csv)"
path_pd = "[https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/payment_detail.csv](https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/payment_detail.csv)"
path_cd = "[https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/customer_detail.csv](https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/customer_detail.csv)"
path_sd = "[https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/sku_detail.csv](https://raw.githubusercontent.com/dataskillsboost/FinalProjectDA11/main/sku_detail.csv)"

df_od = pd.read_csv(path_od)
df_pd = pd.read_csv(path_pd)
df_cd = pd.read_csv(path_cd)
df_sd = pd.read_csv(path_sd)

# Menyaring data transaksi valid, kategori Mobiles & Tablets, selama tahun 2022
data1 = pd.DataFrame(
    df_od[(df_od['is_valid'] == 1) &
          (df_od['category'] == 'Mobiles & Tablets') &
          (df_od['order_date'] >= '2022-01-01') &
          (df_od['order_date'] <= '2022-12-31')]
    .groupby(by=["sku_name"])["qty_ordered"]
    .sum()
    .sort_values(ascending=False)
    .head(5)
    .reset_index(name='qty_2022')
)
data1

# Visualisasi Grafik Batang
sort_data1 = data1.sort_values(by='qty_2022', ascending=True)
plt.figure(figsize=(10, 6))
plt.barh(sort_data1['sku_name'], sort_data1['qty_2022'], color='skyblue')
plt.xlabel('Kuantitas Terjual')
plt.ylabel('Nama Produk')
plt.title('Top 5 Produk Kategori Mobiles & Tablets Sepanjang 2022')
plt.grid(axis='x', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

# Kuantitas Penjualan Kategori Others Tahun 2021
data2_2021 = df_od[(df_od['is_valid'] == 1) &
                   (df_od['category'] == 'Others') &
                   (df_od['order_date'] >= '2021-01-01') &
                   (df_od['order_date'] <= '2021-12-31')] \
            .groupby(by=["sku_name"])["qty_ordered"].sum() \
            .reset_index(name='qty_2021')

# Kuantitas Penjualan Kategori Others Tahun 2022
data2_2022 = df_od[(df_od['is_valid'] == 1) &
                   (df_od['category'] == 'Others') &
                   (df_od['order_date'] >= '2022-01-01') &
                   (df_od['order_date'] <= '2022-12-31')] \
            .groupby(by=["sku_name"])["qty_ordered"].sum() \
            .reset_index(name='qty_2022')

# Menggabungkan data penjualan 2021 & 2022 serta menghitung selisih penurunan
data2_merge = pd.merge(data2_2021, data2_2022, on='sku_name', how='outer').fillna(0)
data2_merge['qty_decrease'] = data2_merge['qty_2021'] - data2_merge['qty_2022']
data2_top20 = data2_merge.sort_values(ascending=False, by='qty_decrease').head(20)
data2_top20

# Menyaring data transaksi Checkout namun Belum Bayar (Unpaid) di tahun 2022
unpaid_2022 = df_od[(df_od['is_valid'] == 0) &
                    (df_od['is_gross'] == 1) &
                    (df_od['is_net'] == 0) &
                    (df_od['order_date'] >= '2022-01-01') &
                    (df_od['order_date'] <= '2022-12-31')]

total_unique_users = unpaid_2022['customer_id'].nunique()
print(f"Total Unique Customers Unpaid 2022: {total_unique_users}")

# Membuat kolom pembantu nama hari dan tipe hari (Weekend / Weekday)
df_od['order_date'] = pd.to_datetime(df_od['order_date'])
df_od['day_name'] = df_od['order_date'].dt.day_name()
df_od['day_type'] = np.where(df_od['day_name'].isin(['Saturday', 'Sunday']), 'Weekend', 'Weekday')

# Menyaring total penjualan harian valid periode 1 Oktober - 31 Desember 2022
daily_revenue = df_od[(df_od['is_valid'] == 1) &
                      (df_od['order_date'] >= '2022-10-01') &
                      (df_od['order_date'] <= '2022-12-31')] \
                .groupby(['order_date', 'day_type'])['before_discount'].sum().reset_index()

# Menghitung rata-rata pendapatan harian berdasarkan tipe hari
avg_revenue_type = daily_revenue.groupby('day_type')['before_discount'].mean().reset_index(name='avg_revenue')

# Membuat DataFrame visualisasi untuk menyamakan format tampilan data harian
df_avg_3months = pd.DataFrame({
    'Tipe Hari': ['Weekend (Sabtu - Minggu)', 'Weekday (Senin - Jumat)'],
    'Rata-Rata Penjualan Harian (Okt-Des 2022)': [
        f"{avg_revenue_type.loc[avg_revenue_type['day_type']=='Weekend', 'avg_revenue'].values[0]:,.2f}",
        f"{avg_revenue_type.loc[avg_revenue_type['day_type']=='Weekday', 'avg_revenue'].values[0]:,.2f}"
    ]
})
df_avg_3months

# Visualisasi Grafik Perbandingan Batang
plt.figure(figsize=(8, 5))
colors = ['#ff9999', '#66b3ff']
plt.bar(avg_revenue_type['day_type'], avg_revenue_type['avg_revenue'], color=colors, width=0.4)
plt.title('Perbandingan Rata-Rata Penjualan Harian (Oktober - Desember 2022)')
plt.ylabel('Rata-Rata Penjualan')
plt.xlabel('Tipe Hari')
plt.grid(axis='y', linestyle='--', alpha=0.5)
plt.show()

## 💡 Rekomendasi Strategis Bisnis

Berdasarkan hasil analisis data transaksi di atas, berikut adalah beberapa rekomendasi strategis yang dapat diterapkan untuk meningkatkan performa bisnis:

1. **Optimalisasi High Season & Forecasting (Kategori Mobiles & Tablets):**
   * Memaksimalkan perencanaan stok (*inventory*) dan promosi pada produk-produk TOP 5 terlaris (seperti seri IDROID dan Apple iPhone) terutama menjelang kuartal akhir (Q4) guna menangkap lonjakan pasar secara maksimal tanpa kendala kehabisan barang.

2. **Taktik Cross-Selling untuk Mengatasi Penurunan Kategori "Others":**
   * Mengingat adanya penurunan tajam pada beberapa SKU di kategori *Others*, tim marketing dapat memasangkan produk bervolume rendah tersebut dengan kategori yang selalu ramai (seperti *Fashion* atau *Mobiles*) lewat promo kombo/bundling paket hemat untuk mendongkrak penjualan silang.

3. **Pemberian Voucher Onboarding & Pengingat Sistem untuk Menekan Angka Unpaid:**
   * Menyediakan insentif berupa voucer diskon khusus dengan masa kedaluwarsa ketat (misal: berlaku maksimal 7 hari setelah klaim) untuk mendorong pelanggan menyelesaikan pembayaran. Selain itu, mengaktifkan notifikasi otomatis (*push notification*) atau email pengingat jika ada transaksi yang tertinggal di keranjang belanja (*checkout unpaid*).

4. **Kampanye Kilat Khusus Weekend (Weekend Flash Sale):**
   * Analisis menunjukkan adanya perbedaan rata-rata penjualan harian antara hari kerja dan akhir pekan. Untuk mendongkrak performa di akhir pekan, disarankan membuat kampanye khusus seperti *Flash Sale Weekend* atau promo gratis ongkir tanpa minimum belanja khusus di hari Sabtu dan Minggu.
