# p02_kebutuhan_data_kopma_104.md

**Nama File:** Dea Novita  
**NIM:** 25430104  
**Tanggal:** 6 Oktober 2026  
**Modul:** 02 - Analisis Kebutuhan Data Koperasi Mahasiswa (Kopma)



## **1. Latar Belakang dan Aktivitas Organisasi**

**Koperasi Mahasiswa (Kopma)** merupakan unit kegiatan mahasiswa yang menjalankan usaha toko ritel & toko daring untuk memenuhi kebutuhan civitas akademika kampus dan masyarakat umum di wilayah Lampung. Aktivitas utama Kopma meliputi pengelolaan akun pengguna (**admin**, **staf**, **pembeli/anggota**), katalog produk per kategori, pemesanan barang, pembayaran digital/COD, pengiriman resi, hingga pelaporan operasional. Untuk menjaga kualitas pelayanan dan transparansi, Kopma membutuhkan sistem basis data terstruktur **`toko_daring_lampung`** yang mencatat transaksi secara real-time, mengontrol stok barang, serta mengelola status pembayaran dan pengiriman secara akurat.



## **2. Aktor dan Proses Bisnis**

| **Kode** | **Proses Bisnis** | **Aktor** | **Pemicu** |
| :--- | :--- | :--- | :--- |
| **PB-01** | **Mengelola Pengguna & Anggota** | **Admin (`pengguna`)** | **Pendaftaran pengguna baru atau pembaharuan hak akses** |
| **PB-02** | **Mencatat Transaksi Penjualan** | **Pembeli (`pengguna`), Staf** | **Pembeli melakukan checkout pesanan** |
| **PB-03** | **Mengelola Katalog & Stok** | **Staf (`pengguna`)** | **Perubahan stok atau penambahan produk baru** |
| **PB-04** | **Memproses Pembayaran** | **Pembeli, Admin** | **Pembeli mengunggah bukti transfer / QRIS / COD** |
| **PB-05** | **Mengelola Pengiriman & Resi** | **Kurir / Staf (`pengguna`)** | **Pesanan siap dikirim ke alamat tujuan** |
| **PB-06** | **Menyusun Laporan Operasional** | **Admin (`pengguna`)** | **Periode rekapitulasi harian/bulanan** |



## **3. Dokumen Sumber yang Dianalisis**

1. **Formulir Pendaftaran & Profil Pengguna:** Berisi **`nama_lengkap`**, **`email_pengguna`**, **`nomor_telepon`**, **`peran_akses`**.
2. **Katalog Produk & Kategori:** Berisi **`nama_produk`**, **`merek_produk`**, **`harga_produk`**, **`stok_produk`**, **`nama_kategori`**.
3. **Struk / Invoice Pesanan:** Berisi **`id_pesanan`**, **`tanggal_pesanan`**, **`total_bayar`**, **`alamat_tujuan`**, **`kota_kabupaten`**, serta rincian **`detail_pesanan`**.
4. **Bukti Pembayaran:** Berisi **`metode_pembayaran`**, **`jumlah_transfer`**, **`bukti_pembayaran`**, **`status_pembayaran`**.
5. **Resi Pengiriman:** Berisi **`kurir_ekspedisi`**, **`nomor_resi`**, **`tanggal_dikirim`**, **`tanggal_diterima`**, **`bukti_paket_sampai`**.



## **4. Entitas Kandidat dan Elemen Data**

1. **Pengguna (`pengguna`)**
   * Atribut: **`id_pengguna` (PK)**, **`nama_lengkap`**, **`email_pengguna` (UNIQUE)**, **`kata_sandi`**, **`nomor_telepon`**, **`peran_akses`**
2. **Kategori Produk (`kategori_produk`)**
   * Atribut: **`id_kategori` (PK)**, **`nama_kategori`**, **`deskripsi_kategori`**
3. **Produk (`produk`)**
   * Atribut: **`id_produk` (PK)**, **`id_kategori` (FK)**, **`nama_produk`**, **`merek_produk`**, **`harga_produk`**, **`stok_produk`**, **`deskripsi_produk`**, **`gambar_utama`**, **`rating_produk`**
4. **Pesanan (`pesanan`)**
   * Atribut: **`id_pesanan` (PK)**, **`id_pembeli` (FK)**, **`tanggal_pesanan`**, **`total_bayar`**, **`alamat_tujuan`**, **`kota_kabupaten`**, **`status_pesanan`**
5. **Detail Pesanan (`detail_pesanan`)**
   * Atribut: **`id_detail` (PK)**, **`id_pesanan` (FK)**, **`id_produk` (FK)**, **`jumlah_beli`**, **`subtotal_harga`**
6. **Pembayaran (`pembayaran`)**
   * Atribut: **`id_pembayaran` (PK)**, **`id_pesanan` (FK)**, **`metode_pembayaran`**, **`tanggal_pembayaran`**, **`jumlah_transfer`**, **`bukti_pembayaran`**, **`status_pembayaran`**
7. **Pengiriman Resi (`pengiriman_resi`)**
   * Atribut: **`id_pengiriman` (PK)**, **`id_pesanan` (FK)**, **`kurir_ekspedisi`**, **`nomor_resi` (UNIQUE)**, **`tanggal_dikirim`**, **`tanggal_diterima`**, **`nama_penerima`**, **`bukti_paket_sampai`**, **`status_pengiriman`**
8. **Banner Utama (`banner_utama`)**
   * Atribut: **`id_banner` (PK)**, **`judul_banner`**, **`gambar_banner`**, **`tautan_promo`**, **`urutan_tampil`**


## **5. Aturan Bisnis**

| **Kode** | **Nama Aturan Bisnis** | **Deskripsi dan Formula (NIM 104)** |
| :--- | :--- | :--- |
| **AB-01** | **Batas Item Pesanan** | **Satu pesanan (`pesanan`) menampung maksimal 7 jenis produk ($P + 2 = 5 + 2 = 7$).** |
| **AB-02** | **Diskon Anggota / Promo** | **Promo khusus member aktif potongan sebesar 5% ($P = 5\%$) dari total belanja.** |
| **AB-03** | **Validasi Stok** | **Transaksi ditolak jika `jumlah_beli` > `stok_produk`. `stok_produk` tidak boleh negatif.** |
| **AB-04** | **Konsistensi Subtotal** | **`subtotal_harga` di `detail_pesanan` mencatat harga perkalian `jumlah_beli` × `harga_produk` saat transaksi terjadi.** |
| **AB-05** | **Ambang Restock** | **Notifikasi restock aktif jika `stok_produk` $\le 10$ unit.** |
| **AB-06** | **Keunikan Identitas** | **Atribut `email_pengguna` pada tabel `pengguna` dan `nomor_resi` pada `pengiriman_resi` bersifat unik.** |
| **AB-07** | **Status Pembayaran & Pengiriman** | **`pengiriman_resi` baru dapat diterbitkan jika `status_pembayaran` = 'Lunas' atau 'COD Lampung'.** |



## **6. Kebutuhan Informasi**

| **Kode** | **Kebutuhan Informasi** | **Target / Deskripsi Spesifikasi** |
| :--- | :--- | :--- |
| **KI-01** | **Laporan Omzet Harian** | **Mampu memproses beban transaksi hingga 65 transaksi/hari ($40 + 5 \times 5 = 65$).** |
| **KI-02** | **Analisis Produk Terlaris** | **Menyajikan daftar 5 produk terlaris berdasarkan akumulasi `jumlah_beli` bulanan.** |
| **KI-03** | **Peringatan Stok Minimum** | **Menampilkan daftar `produk` yang `stok_produk` $\le 10$ secara otomatis.** |
| **KI-04** | **Monitoring Pembayaran** | **Menampilkan daftar pembayaran dengan `status_pembayaran` = 'Pending' untuk diverifikasi admin.** |
| **KI-05** | **Tracking Pengiriman** | **Menampilkan `status_pengiriman` ('Dalam Perjalanan' / 'Sampai Tujuan') beserta `bukti_paket_sampai`.** |



## **7. Matriks CRUD**

| **Kode Proses** | **Nama Proses Bisnis** | **pengguna** | kategori_produk | produk | pesanan | detail_pesanan | **pembayaran** | **pengiriman_resi** |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| PB-01 | Mengelola Pengguna | C, R, U, D | | | | | | |
| PB-02 | Mencatat Penjualan | R | R | R, U (stok) | C | C | | |
| PB-03 | Mengelola Produk | | C, R, U, D | C, R, U, D | | | | |
| PB-04 | Memproses Pembayaran | | | | R | R | C, R, U |
| PB-05 | Mengelola Pengiriman | | | | R, U | | | C, R, U |
| PB-06 | Menyusun Laporan | R | R | R | R | R | R | R |

***Keterangan:** **C = Create**, **R = Read**, **U = Update**, **D = Delete***



## **8. Kamus Data Awal**

| **Elemen Data** | **Deskripsi** | **Tipe Data** | **Contoh Nilai** | **Aturan Bisnis / Constraint** | **Penanggung Jawab (*Data Steward*)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`email_pengguna`** | **Email resmi akun pengguna** | **VARCHAR(100)** | **`malik.pembeli@gmail.com`** | **UNIQUE, NOT NULL** | **Super Admin** |
| **`harga_produk`** | **Harga satuan barang** | **INT(9)** | **`250000`** | **Nominal $\ge 0$** | **Staf Operational** |
| **`stok_produk`** | **Stok fisik tersisa** | **INT(5)** | **`45`** | **INT $\ge 0$, AB-03** | **Staf Operational** |
| **`total_bayar`** | **Total tagihan pesanan** | **INT(9)** | **`285000`** | **Total `subtotal_harga`** | **Staf Operational** |
| **`metode_pembayaran`** | **Jenis pembayaran** | **VARCHAR(50)** | **`Transfer Bank BNI`** | **Enum/Option valid** | **Admin Finance** |
| **`nomor_resi`** | **Kode unik resi pengiriman** | **VARCHAR(30)** | **`LMP-JNT-20261001-0001`** | **UNIQUE** | **Courier / Staf** |



## **9. Kebutuhan Non-Fungsional Data**

* **Volume Data:** Didesain untuk memproses volume rata-rata **65 transaksi harian** dengan kapasitas hingga **20.000 entri `pesanan`** per tahun.
* **Retensi Data:** Seluruh rekap transaksi **`pesanan`**, **`pembayaran`**, dan **`pengiriman_resi`** disimpan minimal selama **5 tahun** untuk pemeriksaan audit keuangan.
* **Privasi dan Keamanan:** Atribut **`kata_sandi`** disimpan dalam bentuk **hash terenkripsi**; **`nomor_telepon`** dan **`alamat_tujuan`** terproteksi sesuai **`peran_akses`**.



## **10. Isu Kualitas Data yang Diantisipasi**

1. **Integritas Referensial (*Foreign Key Constraints*):** Penggunaan **`ON DELETE CASCADE`** pada relasi **`id_kategori`**, **`id_pembeli`**, dan **`id_pesanan`** untuk menjaga konsistensi data anak saat data induk terhapus.
2. **Pencegahan Anomali Entri Ganda:** Mengaktifkan constraint **`UNIQUE`** pada **`email_pengguna`** dan **`nomor_resi`**.
3. **Inkonsistensi Stok Barang:** Menggunakan transaksi atomik (**Atomic Transaction**) saat **`pesanan`** dibuat agar **`stok_produk`** langsung terpotong secara akurat.