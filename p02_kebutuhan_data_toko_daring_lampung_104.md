# Dokumen Kebutuhan Data - Toko Daring Lampung

**Nama :** Dea Novita 
**NIM:** 25430104
**Tanggal:** 6 Oktober 2026  



## 1. Latar Belakang dan Aktivitas Organisasi
Toko Daring Lampung merupakan platform e-commerce lokal yang berfokus pada penjualan produk-produk khas Provinsi Lampung, seperti olahan keripik pisang, kopi robusta, kain tapis, hingga kerajinan tangan lokal. Aktivitas utama platform meliputi pendaftaran akun pelanggan, katalogisasi produk lokal, pemrosesan pesanan daring (*checkout*), verifikasi pembayaran digital, penanganan pengiriman barang melalui jasa logistik/kurir, serta penyusunan laporan penjualan harian dan bulanan. Sistem basis data diperlukan untuk memastikan integritas stok produk lokal, mencatat riwayat transaksi secara terstruktur, dan mengelola program *member* serta poin loyalitas pelanggan.



## 2. Aktor dan Proses Bisnis

| Kode | Proses Bisnis | Aktor Utama | Pemicu |
| :--- | :--- | :--- | :--- |
| **PB-01** | Registrasi & Kelola Akun Pelanggan | Pelanggan / Sistem | Pelanggan mendaftarkan akun baru atau memperbarui data profil |
| **PB-02** | Pemesanan Produk (Checkout) | Pelanggan | Pelanggan menaruh barang di keranjang dan melakukan *checkout* |
| **PB-03** | Pembayaran & Konfirmasi Pesanan | Pelanggan / Payment Gateway | Pelanggan melakukan transfer atau pembayaran elektronik |
| **PB-04** | Pengemasan & Pengiriman Barang | Petugas Logistik / Kurir | Pembayaran pesanan telah berhasil terkonfirmasi oleh sistem |
| **PB-05** | Pengelolaan Stok & Produk Lokal | Admin Toko / Petugas Gudang | Adanya penambahan komoditas Lampung baru atau penyesuaian harga |
| **PB-06** | Penyusunan Laporan Penjualan Daring | Manajer Toko | Memasuki awal bulan atau periode evaluasi berkala |



## 3. Dokumen Sumber yang Dianalisis
1. **Formulir Profil Akun Pelanggan:** Berisi data akun (*email*, nama, nomor telepon, dan alamat pengiriman di Lampung/luar daerah).
2. **Invoice / Struk Pesanan Daring:** Memuat nomor pesanan, tanggal transaksi, rincian produk khas yang dibeli, harga per unit saat transaksi, potongan *member*, ongkos kirim, dan total bayar.
3. **Bukti Pembayaran Digital:** Catatan konfirmasi transaksi dari *payment gateway* (nomor referensi, metode bayar, dan waktu bayar).
4. **Resi Pengiriman Logistik:** Memuat nomor resi, nama jasa kurir, berat paket, serta status pelacakan pengiriman.



## 4. Entitas Kandidat dan Elemen Data
1. **Pelanggan**
   * Atribut: `id_pelanggan` (PK), `nama_pelanggan`, `email_pelanggan`, `no_hp_pelanggan`, `alamat_lampung`, `status_member`, `poin_loyalitas`
2. **Produk**
   * Atribut: `kode_produk` (PK), `nama_produk`, `kategori_khas`, `harga_jual`, `stok_produk`, `min_stok`
3. **Pesanan (Order)**
   * Atribut: `no_pesanan` (PK), `tgl_pesanan`, `id_pelanggan` (FK), `total_harga`, `diskon_member`, `ongkos_kirim`, `status_pesanan`
4. **Detail Pesanan**
   * Atribut: `no_pesanan` (FK), `kode_produk` (FK), `qty_pesanan`, `harga_satuan_pesanan`
5. **Pembayaran**
   * Atribut: `kode_pembayaran` (PK), `no_pesanan` (FK), `tgl_bayar`, `metode_bayar`, `jumlah_bayar`, `status_pembayaran`
6. **Pengiriman**
   * Atribut: `no_resi` (PK), `no_pesanan` (FK), `nama_ekspedisi`, `tgl_kirim`, `tgl_diterima`, `status_pengiriman`
7. **Admin / Petugas Gudang**
   * Atribut: `kode_petugas` (PK), `nama_petugas`, `peran_petugas`



## 5. Aturan Bisnis

| Kode | Nama Aturan Bisnis | Deskripsi dan Formula (NIM 104) |
| :--- | :--- | :--- |
| **AB-01** | Batas Item Pesanan | Setiap 1 pesanan daring maksimal menampung **7 variasi produk** lokal yang berbeda ($P + 2 = 5 + 2 = 7$). |
| **AB-02** | Diskon Member Loyalty | Pelanggan terdaftar berstatus *Member* aktif berhak mendapat diskon **5%** ($P = 5\%$) dari total belanja produk. |
| **AB-03** | Validasi Stok Daring | Pesanan otomatis dibatalkan jika $qty > stok\_produk$. Stok produk di gudang tidak boleh bernilai negatif. |
| **AB-04** | Integritas Harga Historis | `harga_satuan_pesanan` di Detail Pesanan mengunci harga produk saat *checkout* agar tidak berubah jika harga master naik. |
| **AB-05** | Ambang Restock Produk | Notifikasi otomatis dikirim ke Petugas Gudang jika $stok\_produk \le min\_stok$. |
| **AB-06** | Keunikan Identitas Akun | Atribut `email_pelanggan` dan `no_hp_pelanggan` bersifat unik untuk mencegah akumulasi akun ganda. |
| **AB-07** | Akumulasi Poin Belanja | Setiap kelipatan transaksi Rp10.000, pelanggan berhak mendapatkan 1 poin loyalitas. |



## 6. Kebutuhan Informasi

| Kode | Kebutuhan Informasi | Target / Deskripsi Spesifikasi |
| :--- | :--- | :--- |
| **KI-01** | Laporan Penjualan Daring Harian | Sistem mampu mengolah beban transaksi hingga **65 pesanan/hari** ($40 + 5 \times 5 = 65$). |
| **KI-02** | Analisis Produk Khas Terlaris | Menyajikan daftar 5 Produk Khas Lampung (*Kopi Robusta, Keripik Pisang, Tapis, dll.*) paling laris per bulan. |
| **KI-03** | Peringatan Stok Minimum Gudang | Menampilkan daftar produk lokal yang stoknya berada pada atau di bawah `min_stok`. |
| **KI-04** | Rekap Diskon & Poin Member | Menampilkan total potongan diskon 5% dan akumulasi poin loyalitas pelanggan yang ditukarkan. |
| **KI-05** | Laporan Pengawasan Keranjang | Menampilkan daftar transaksi yang mendekati atau mencapai limit 7 jenis produk per *checkout*. |



## 7. Matriks CRUD

| Kode Proses | Nama Proses Bisnis | Pelanggan | Produk | Pesanan | Detail Pesanan | Pembayaran | Pengiriman |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **PB-01** | Registrasi Pelanggan | C, U | | | | | |
| **PB-02** | Pemesanan Produk (Checkout) | R, U (poin) | R, U (stok) | C | C | | |
| **PB-03** | Pembayaran & Konfirmasi | R | | U (status) | R | C | |
| **PB-04** | Pengemasan & Pengiriman | R | | U (status) | R | R | C, U |
| **PB-05** | Pengelolaan Stok & Produk | | C, R, U, D | | | | |
| **PB-06** | Penyusunan Laporan Daring | R | R | R | R | R | R |

*Keterangan: C = Create, R = Read, U = Update, D = Delete*



## 8. Kamus Data Awal

| Elemen Data | Deskripsi | Tipe Data | Contoh Nilai | Aturan Bisnis / Constraint | Penanggung Jawab (*Data Steward*) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id_pelanggan` | Kode unik identitas pelanggan | String | CUST-0104 | Format CUST-4digit, Unik | Admin Toko |
| `no_pesanan` | Nomor unik transaksi pesanan | String | ORD-2610-0104 | Unik per transaksi | System Automation |
| `harga_satuan_pesanan` | Harga aktual produk saat checkout | Integer | 35000 | Nominal Rupiah $\ge 0$, AB-04 | Admin Toko |
| `stok_produk` | Stok fisik produk di gudang Lampung | Integer | 120 | Integer $\ge 0$, AB-03 | Petugas Gudang |
| `diskon_member` | Nilai potongan harga untuk *Member* | Integer | 5000 | Nominal $\ge 0$, AB-02 ($P=5\%$) | Admin Toko |
| `no_resi` | Nomor resi pengiriman kurir | String | LGP-990104 | Unik per pengiriman | Petugas Logistik |



## 9. Kebutuhan Non-Fungsional Data

* **Volume Data:** Didesain untuk menangani rata-rata 65 pesanan harian dengan proyeksi skalabilitas hingga 25.000 transaksi per tahun.
* **Retensi Data:** Seluruh berkas pesanan, bukti pembayaran, dan riwayat pengiriman disimpan minimal selama 5 tahun untuk pembukuan pajak dan kewajiban audit usaha.
* **Privasi dan Keamanan:** Data sensitif seperti `alamat_lampung`, `email_pelanggan`, dan `no_hp_pelanggan` dilindungi dengan enkripsi serta pembatasan hak akses; kurir pengiriman hanya dapat melihat alamat tanpa bisa mengubah data profil pelanggan.



## 10. Isu Kualitas Data yang Diantisipasi

1. **Pencegahan Anomali Harga Historis:** Memisahkan atribut harga produk master dan harga di detail pesanan agar penyesuaian harga barang lokal di kemudian hari tidak merusak laporan pendapatan lama.
2. **Pencegahan Akun Ganda (*Duplicate Account*):** Mengaktifkan *constraint UNIQUE* pada `email_pelanggan` dan `no_hp_pelanggan`.
3. **Inkonsistensi Stok Daring vs Gudang:** Penerapan mekanisme *atomic transaction* (penguncian stok sementara saat proses *checkout*) untuk mencegah situasi *overselling* saat dua pelanggan membeli item terakhir bersamaan.++