# Laporan Praktikum Basis Data - Pertemuan 03

**Nama:** Dea Novita  **NIM:** 25430104  **Kelas:** D  **Tanggal:** 12 Oktober 2026

## 1. Tujuan Praktikum

Di pertemuan ini saya belajar mengubah daftar kebutuhan data dari Pertemuan 2 menjadi gambar ERD. Saya menentukan entitas, atribut, kunci, dan relasi beserta kardinalitasnya, semuanya berdasarkan aturan bisnis. Relasi banyak-ke-banyak saya pecah dengan entitas asosiatif, lalu ERD-nya saya cek lagi terhadap kebutuhan informasi (KI).

## 2. Ringkasan Dasar Teori

ERD menggambarkan data lewat **entitas** (hal yang dicatat), **atribut** (sifat dari entitas), dan **relasi** (hubungan antarentitas yang dibaca sebagai kalimat berkata kerja). Kardinalitas menjawab "paling banyak berapa", sedangkan partisipasi menjawab "wajib atau boleh kosong". Di notasi Crow's Foot, simbol yang dekat dengan kotak entitas menunjukkan maksimum, dan simbol yang lebih jauh menunjukkan minimum. Setiap ujung garis harus punya dasar aturan bisnis, jadi tidak boleh asal tebak.

Relasi banyak-ke-banyak tidak bisa disimpan langsung di tabel relasional. Karena itu relasinya dipecah dengan entitas asosiatif, yang sekaligus menyimpan data milik relasi tersebut (contohnya qty dan harga saat transaksi). Atribut turunan seperti subtotal bisa dihitung dari data lain, jadi saya tandai saja dulu, dan keputusan apakah disimpan atau tidak saya tunda ke Pertemuan 4.

## 3. Hasil Langkah Percobaan

### D.1 Entitas Kopma

Ada delapan entitas di Kopma: petugas, anggota, pemasok, barang, penjualan, detail_penjualan, pembelian, dan detail_pembelian. Pembelian dan detailnya saya pecah dengan pola yang sama seperti penjualan.

| Entitas | PK | Kunci alternatif | Atribut turunan |
|---|---|---|---|
| petugas | id_petugas | kode_petugas | - |
| anggota | id_anggota | no_anggota, nim_anggota | - |
| pemasok | id_pemasok | - | - |
| barang | id_barang | kode_barang | - |
| penjualan | id_penjualan | no_nota_penjualan | total nota, diskon 5% |
| detail_penjualan | id_penjualan + id_barang | - | subtotal (qty x harga) |
| pembelian | id_pembelian | no_faktur_pembelian | total faktur |
| detail_pembelian | id_pembelian + id_barang | - | subtotal beli |

### D.2 Relasi dan kardinalitas Kopma

| Relasi | Kiri | Kanan | Dasar |
|---|---|---|---|
| petugas melayani penjualan | 1 | 0..N | wawancara kasir |
| anggota melakukan penjualan | 0..1 | 0..N | AB-02 |
| penjualan berisi detail_penjualan | 1 | 1..N | AB-01 |
| barang tercantum di detail_penjualan | 1 | 0..N | AB-04 |
| pemasok menerima pembelian | 1 | 0..N | AB-06 |
| pembelian berisi detail_pembelian | 1 | 1..N | faktur pemasok |
| barang tercantum di detail_pembelian | 1 | 0..N | faktur pemasok |

### D.3 ERD Kopma di draw.io

Saya menggambar kedelapan entitas memakai pustaka Entity Relation di draw.io, dengan satu warna untuk tiap entitas. Gambarnya sudah lengkap dengan PK/FK, atribut, dan Crow's Foot berlabel angka, plus legenda di bagian bawah. Gambar lengkapnya ada di **Lampiran A**.

### D.4 Validasi terhadap KI

| KI | Jalur di ERD |
|---|---|
| KI-01 Omzet per hari/bulan | penjualan -> detail_penjualan |
| KI-02 Lima barang terlaris | barang <- detail_penjualan -> penjualan |
| KI-03 Stok di bawah minimum | barang |
| KI-04 Sepuluh anggota belanja terbesar | anggota -> penjualan -> detail_penjualan |

Semua KI bisa ditelusuri lewat ERD, jadi tidak ada entitas yang perlu ditambah.


## 4. Jawaban Titik Analisis

**Titik Analisis 1.** `no_anggota` dan NIM memang sama-sama unik, tetapi keduanya dibuat manusia sehingga bisa salah ketik atau berubah format. Kunci primer sebaiknya pendek dan stabil, jadi saya memakai `id_anggota` yang dibuat otomatis. Kalau NIM dijadikan PK, setiap kali NIM harus dikoreksi, semua nilainya di tabel penjualan ikut harus diubah. NIM tetap saya simpan sebagai kunci alternatif `UNIQUE`, jadi pencarian lewat NIM (AB-05) masih bisa dilakukan.

**Titik Analisis 2.** Yang memaksanya adalah **AB-01**, yaitu setiap nota minimal punya satu baris barang. Aturan ini tidak bisa dijaga oleh struktur tabel saja. Kunci tamu hanya memastikan detail menunjuk ke nota yang ada, bukan memastikan nota punya detail. Apalagi nota disimpan lebih dulu baru detailnya, jadi sempat ada nota tanpa detail. Solusinya adalah transaksi di Modul 14: nota dan barisnya disimpan sekaligus, lalu `COMMIT` hanya dijalankan kalau ada minimal satu baris.

**Titik Analisis 3.** ERD saat ini **belum bisa** menjawabnya, karena petugas hanya terhubung ke penjualan, sedangkan pembelian hanya menunjuk ke pemasok. Perlu relasi baru *petugas menerima pembelian* (`id_petugas_penerima` di pembelian). Sisi petugas 0..1 (kosong selama status masih "dipesan"), sisi pembelian 0..N. Ini harus dicatat sebagai aturan bisnis baru, **AB-07**: pembelian berstatus diterima wajib mencatat petugas penerima yang berperan gudang. Bagian "berperan gudang" butuh trigger atau pengecekan di aplikasi.

## 5. Hasil Latihan dan Modifikasi

**E.1 Poin loyalitas.** Saya membuat entitas baru **`riwayat_poin`** (id_riwayat_poin, id_anggota, id_penjualan, jenis tambah/tukar, jumlah_poin, tgl_riwayat), bukan sekadar atribut saldo di tabel anggota. Alasannya, kalau hanya saldo yang disimpan, angka lama akan tertimpa dan kita tidak bisa melacak poin itu berasal dari mana. Saldo sendiri adalah nilai turunan yang dihitung dari riwayat. Relasinya: anggota 1 ke 0..N riwayat_poin, dan penjualan 1 ke 0..N riwayat_poin. Aturan barunya: **AB-08** (setiap Rp10.000 belanja anggota aktif mendapat 1 poin) dan **AB-09** (50 poin ditukar potongan Rp5.000, saldo tidak boleh negatif).

**E.2 Penjualan-barang tanpa entitas asosiatif.** Kalau entitas asosiatifnya dihilangkan, relasi penjualan dan barang menjadi banyak-ke-banyak langsung. Akibatnya `qty` dan `harga_satuan` saat transaksi tidak punya tempat, padahal keduanya milik relasi, bukan milik penjualan maupun barang. AB-04 jadi dilanggar, dan KI-01, KI-02, serta KI-04 tidak bisa dihitung dengan benar. Selain itu, tabel relasional tidak bisa menyimpan banyak nilai dalam satu sel, jadi entitas asosiatif tetap diperlukan.

## 6. Tugas Mandiri: Milestone Proyek 3

Proyek saya adalah toko daring Kopma (`toko_daring_lampung`). Nilai P saya 5, sehingga AB-01 membatasi satu pesanan maksimal 7 jenis produk. Gambar ERD proyek ada di **Lampiran B**.

**Entitas dan kunci:** pengguna (PK id_pengguna, alternatif email_pengguna), kategori_produk (id_kategori), produk (id_produk, FK id_kategori), pesanan (id_pesanan, FK id_pembeli), detail_pesanan (id_detail, FK id_pesanan dan id_produk), pembayaran (id_pembayaran, FK id_pesanan), pengiriman_resi (id_pengiriman, FK id_pesanan, alternatif nomor_resi), dan banner_utama (id_banner). Atribut turunan: `total_bayar` dan `subtotal_harga`.

**Relasi dan kardinalitas:**

| Relasi | Kiri | Kanan | Dasar |
|---|---|---|---|
| pengguna melakukan pesanan | 1 | 0..N | PB-02 (usulan AB-08) |
| kategori_produk mengelompokkan produk | 1 | 0..N | PB-03 |
| pesanan berisi detail_pesanan | 1 | 1..N (maks. 7) | AB-01 |
| produk tercantum di detail_pesanan | 1 | 0..N | AB-03, AB-04 |
| pesanan dibayar lewat pembayaran | 1 | 0..N | PB-04, AB-07 |
| pesanan dikirim lewat pengiriman_resi | 1 | 0..N | AB-07, AB-06 |

Pembayaran saya buat 0..N karena pembayaran yang gagal bisa diulang. Pengiriman awalnya nol karena resi baru terbit setelah pembayaran Lunas atau COD Lampung. Adapun `banner_utama` berdiri sendiri tanpa relasi.

**Entitas asosiatif:** `detail_pesanan` berada di antara pesanan dan produk, dengan atribut sendiri `jumlah_beli` dan `subtotal_harga` (harga saat transaksi, AB-04).

**Penelusuran KI dan pemetaan:**

| KI | Jalur di ERD | Entitas ke PB |
|---|---|---|
| KI-01 Omzet harian | pesanan (disaring lewat pembayaran Lunas) | pesanan: PB-02, PB-04 |
| KI-02 Produk terlaris | produk <- detail_pesanan -> pesanan | produk: PB-02, PB-03; detail_pesanan: PB-02 |
| KI-03 Stok minimum | produk | kategori_produk: PB-03 |
| KI-04 Pembayaran Pending | pembayaran -> pesanan -> pengguna | pembayaran: PB-04 |
| KI-05 Tracking pengiriman | pengiriman_resi -> pesanan -> pengguna | pengiriman_resi: PB-05; pengguna: PB-01 |



**Temuan dan keputusan desain:**
1. AB-02 (diskon member 5%) belum punya tempat di ERD. Usulan saya: tambah `status_member` di pengguna dan `diskon_pesanan` di pesanan.
2. `banner_utama` belum terkait dengan proses bisnis maupun KI mana pun. Perlu ada proses baru "Mengelola tampilan toko", atau alasan yang jelas kenapa entitas ini dipertahankan.
3. `total_bayar` dan `subtotal_harga` adalah atribut turunan, dan keputusannya dibahas di Pertemuan 4. Kalau normalisasi menemukan masalah, ERD akan diperbaiki dan dicatat di commit.

## 7. Pembahasan dan Kendala

Bagian yang paling gampang keliru adalah arah kardinalitas. Cara saya mengeceknya adalah membaca dari entitas di seberang, misalnya "untuk satu pesanan, berapa pengguna paling banyak?", lalu jawabannya digambar di ujung garis yang dekat dengan pengguna. Aturan "minimal satu" pada pesanan dan detailnya juga tidak bisa dijamin oleh struktur tabel, jadi harus dilanjutkan dengan transaksi di Modul 14. Kendala teknis di draw.io: [ISI bila ada, atau tulis "tidak ada"].

## 8. Kesimpulan

ERD membuat kebutuhan data lebih mudah dibaca bersama, asalkan setiap relasi punya kata kerja dan dasar aturan bisnis. Entitas asosiatif seperti `detail_pesanan` diperlukan untuk menyimpan data milik relasi. Validasi terhadap KI juga membantu menemukan celah di ERD, seperti AB-02 dan `banner_utama`, sebelum masuk ke normalisasi.

## 9. Pernyataan Penggunaan AI

Saya memakai Claude (Anthropic) untuk membantu menyusun draf laporan, membuat gambar ERD proyek dan skrip SQL, serta mengecek kardinalitas. Semua hasilnya saya baca dan cocokkan dengan modul dan kode saya sendiri. [SESUAIKAN dengan penggunaan yang sebenarnya.]

## 10. Bukti Git

- Repositori: `https://github.com/[USERNAME]/basisdata-25430104`
- Commit: `p03: erd kopma`
- Hash: `[ISI dari git log -1 --format=%h]`



## Lampiran

### Lampiran A. ERD Kopma

![ERD Kopma](<img/kopma.drawio.png>) ![dan](<img/digKOP.png>)

*Gambar A. ERD Kopma (`p03_erd_kopma_25430104.drawio` dan `.png`).*

### Lampiran B. ERD Proyek Toko Daring

![ERD proyek](<img/tokoo.drawio.png>)![dan](<img width="960" height="540" alt="toko" src="https://github.com/user-attachments/assets/3a233562-c004-4634-ba4c-ac8f2eed6b0a" />)

*Gambar B. ERD proyek toko daring (`p03_erd_25430104.drawio` dan `.png`).*
