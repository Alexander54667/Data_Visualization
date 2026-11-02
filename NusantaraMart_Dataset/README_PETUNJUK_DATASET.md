# Paket Dataset Praktikum NusantaraMart — Sintetis

Dataset ini sepenuhnya sintetis, bukan data transaksi perusahaan sungguhan. Didesain untuk mendampingi 50 soal visualisasi data. Periode transaksi dan pemasaran 1 Januari 2024 sampai 31 Desember 2025.

## File dan hubungan
- customers.csv: satu baris per pelanggan (customer_id unik); tanggal pendaftaran, kota, provinsi, usia, gender, acquisition_channel.
- products.csv: satu baris per produk (product_id unik), category, brand, cost_price (harga pokok per unit).
- transactions.csv: satu baris per *item pesanan* (order_item_id unik). Satu order_id dapat muncul berkali-kali. Kolom order_id, order_item_id, customer_id, product_id, order_date, quantity, unit_price, discount, shipping_cost, payment_method, order_status.
- marketing.csv: satu baris per tanggal dan channel. Kolom date, channel, impressions, clicks, ad_spend, conversions. Angka konversi agregat ini TIDAK terhubung langsung dengan pesanan individual.

## Aturan analisis
1. Gabungkan transaksi ke pelanggan melalui customer_id, dan ke produk melalui product_id (many-to-one).
2. Nilai diskon berupa PROPORSI 0.0–0.4, bukan rupiah. Item gross = quantity*unit_price, item net = quantity*unit_price*(1-discount).
3. Untuk laporan realized sales, gunakan order_status=='completed'; cancelled dan returned dipisah.
4. Ongkos kirim berada di *tingkat item* dalam data ini, bukan di tingkat order; jangan menjumlah ongkos kirim berulang kali apabila tujuan Anda menghitung ongkir per pesanan. Untuk simplifikasi, pendapatan barang tidak memasukkan ongkir.
5. Margin kotor estimasi produk per item = quantity*(unit_price*(1-discount)-cost_price). Belum memasukkan ongkir, biaya pemasaran, pajak, atau overhead.
6. Hitung order unik dengan nunique(order_id), bukan jumlah baris transaksi.
7. Waktu transaksi tersedia hanya sebagai TANGGAL. Soal berdasarkan JAM perlu dataset tambahan; gunakan analisis hari dalam minggu sebagai pengganti.
8. Usia pelanggan sengaja mengandung sebagian nilai kosong. Ini data latihan untuk penanganan missing values, jangan menganggap usia kosong berarti nol.
9. Tanggal akhir analisis yang konsisten: 2025-12-31. Saat membuat cohort retention, gunakan observasi tersedia dan perlakukan periode masa depan sebagai NA.
10. Tidak tersedia atribut eksperimen terkontrol atau identitas atribusi pemasaran. Jangan menyatakan hubungan kausal hanya dari grafik atau mengklaim bahwa channel marketing tertentu menghasilkan order yang spesifik.
11. Data bersifat realistis untuk latihan, namun tidak dikalibrasi ke ekonomi sebenarnya; tidak untuk membuat klaim tentang perilaku konsumen Indonesia.

## Contoh membaca di Python
```python
import pandas as pd
from pathlib import Path
base = Path('NusantaraMart_Dataset')
transactions = pd.read_csv(base/'transactions.csv', parse_dates=['order_date'])
customers = pd.read_csv(base/'customers.csv', parse_dates=['registration_date'])
products = pd.read_csv(base/'products.csv')
marketing = pd.read_csv(base/'marketing.csv', parse_dates=['date'])
df = transactions.merge(customers, on='customer_id', validate='many_to_one').merge(products, on='product_id', validate='many_to_one')
df['gross_revenue'] = df['quantity'] * df['unit_price']
df['net_revenue'] = df['gross_revenue'] * (1-df['discount'])
completed = df[df['order_status'].eq('completed')]
print(completed.groupby('category')['net_revenue'].sum().sort_values(ascending=False))
```

## Catatan penting pengajaran
Peristiwa marketing.csv dihasilkan terpisah dari transactions.csv. KPI funnel dapat dihitung, tetapi data ini tidak menyediakan join order-level attribution. Beberapa tugas eksploratif bisa menunjukkan hubungan lemah/tidak ada; hal itu tetap hasil yang benar.
