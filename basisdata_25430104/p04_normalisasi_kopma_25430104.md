# Normalisasi Kopma - Pertemuan 4

**Nama:** Dea Novita  **NIM:** 25430104  **Kelas:** D  **Tanggal:** 17 Oktober 2026

> Sumber: `p02_kebutuhan_data_kopma_104.md` (Modul 2). Aturan bisnis AB-01 sampai AB-07 dan entitas diambil dari dokumen tersebut.


## 1. Dokumen Sumber

- **Nama dokumen:** Struk/Invoice Pesanan, Bukti Pembayaran, dan Resi Pengiriman Kopma
- **Aturan bisnis yang dipakai:** AB-01 sampai AB-07

**Asumsi yang dipakai:**
- A1: setiap pesanan memiliki tepat satu bukti pembayaran.
- A2: setiap pesanan memiliki tepat satu resi pengiriman.

## 2. UNF (Bentuk Tak Ternormalisasi)

```
PESANAN_UNF (id_pesanan, tanggal_pesanan, id_pembeli, nama_lengkap, email_pengguna,
             nomor_telepon, peran_akses, alamat_tujuan, kota_kabupaten,
             status_pesanan, total_bayar,
             { id_produk, nama_produk, merek_produk, id_kategori, nama_kategori,
               jumlah_beli, harga_produk, subtotal_harga },
             metode_pembayaran, tanggal_pembayaran, jumlah_transfer,
             bukti_pembayaran, status_pembayaran,
             kurir_ekspedisi, nomor_resi, tanggal_dikirim, tanggal_diterima,
             nama_penerima, bukti_paket_sampai, status_pengiriman)
```

## 3. Dependensi Fungsional

| Kode | Dependensi fungsional | Dasar |
|---|---|---|
| FD1 | id_pesanan → tanggal_pesanan, id_pembeli, alamat_tujuan, kota_kabupaten, status_pesanan | Struk pesanan |
| FD2 | id_pembeli → nama_lengkap, email_pengguna, nomor_telepon, peran_akses | Formulir pendaftaran |
| FD3 | email_pengguna → id_pembeli | AB-06 (email unik) |
| FD4 | id_produk → nama_produk, merek_produk, id_kategori, harga_produk, stok_produk | Katalog produk |
| FD5 | id_kategori → nama_kategori | Katalog kategori |
| FD6 | (id_pesanan, id_produk) → jumlah_beli, harga_satuan_beli | AB-04 (harga dikunci saat transaksi) |
| FD7 | (jumlah_beli, harga_satuan_beli) → subtotal_harga | Nilai turunan |
| FD8 | id_pesanan → metode_pembayaran, tanggal_pembayaran, jumlah_transfer, bukti_pembayaran, status_pembayaran | A1 |
| FD9 | id_pesanan → kurir_ekspedisi, nomor_resi, tanggal_dikirim, tanggal_diterima, nama_penerima, bukti_paket_sampai, status_pengiriman | A2 |
| FD10 | nomor_resi → id_pesanan | AB-06 (resi unik) |


**Titik analisis:** FD6 mengharuskan harga saat transaksi (`harga_satuan_beli`) disimpan di detail pesanan. Di P2 hanya ada `subtotal_harga`, sehingga harga satuan saat beli tidak bisa dipulihkan dari data itu. Kekurangan ini saya perbaiki di bagian 6.

## 4. Tahapan Normalisasi

### 4.1 Menuju 1NF
Hilangkan grup berulang dengan satu baris per produk per pesanan. Kunci primer menjadi (id_pesanan, id_produk).

**Bukti L.1 (tahapan UNF sampai 3NF):**

![alt text](tahapan.png) Gambar 4.2 Tahapan normalisasi UNF sampai 3NF (Buku Praktikum Basis Data, hlm. 35)

### 4.2 Menuju 2NF
Ketergantungan parsial dipisah:
- FD1, FD2, FD8, dan FD9 hanya bergantung pada `id_pesanan`, sehingga masuk tabel PESANAN.
- FD4 hanya bergantung pada `id_produk`, sehingga masuk tabel PRODUK.
- FD6 dan FD7 bergantung pada kunci penuh, sehingga masuk tabel DETAIL_PESANAN.

```
PESANAN       (id_pesanan, tanggal_pesanan, id_pembeli, nama_lengkap, email_pengguna,
               nomor_telepon, peran_akses, alamat_tujuan, kota_kabupaten, status_pesanan,
               metode_pembayaran, ..., kurir_ekspedisi, nomor_resi, ...)
PRODUK        (id_produk, nama_produk, merek_produk, id_kategori, nama_kategori, harga_produk, stok_produk)
DETAIL_PESANAN(id_pesanan, id_produk, jumlah_beli, harga_satuan_beli, subtotal_harga)
```

### 4.3 Menuju 3NF
Ketergantungan transitif dipisah:
- `id_pesanan → id_pembeli → nama, email, telepon, peran` (FD2): dipisah menjadi PENGGUNA.
- `id_produk → id_kategori → nama_kategori` (FD5): dipisah menjadi KATEGORI_PRODUK.
- Pembayaran dan pengiriman punya kunci sendiri, sehingga dipisah menjadi PEMBAYARAN dan PENGIRIMAN_RESI. Keduanya diberi FK `id_pesanan` yang UNIQUE (asumsi A1 dan A2).

Hasil akhir 3NF ada di bagian 7.

## 5. Keputusan Nilai Turunan

| Nilai | Disimpan / dihitung | Alasan |
|---|---|---|
| `subtotal_harga` | Dihitung | = `jumlah_beli` × `harga_satuan_beli`. Harga satuan sudah dikunci (AB-04), jadi hasilnya selalu sama. |
| `total_bayar` | Dihitung | = SUM(`subtotal_harga`) − `diskon_pesanan`. Jika disimpan, bisa tidak cocok dengan detailnya. |
| `diskon_pesanan` | Disimpan | Nilainya bergantung pada status member saat transaksi (AB-02). Status member bisa berubah kemudian, sehingga diskon tidak bisa dihitung ulang dari data sekarang. Ini fakta historis. |
| `jumlah_transfer` | Disimpan | Ini uang yang benar-benar dibayarkan, bukan hasil hitungan. |

**Titik analisis:** Mengapa `diskon_pesanan` disimpan padahal bisa dihitung dari subtotal? Karena status member di tabel PENGGUNA bisa berubah setelah transaksi. Kalau diskon dihitung ulang, nota lama bisa berubah nilainya. Ini sama dengan prinsip AB-04 pada harga.

## 6. Perbaikan dari P2

Normalisasi menemukan dua kekurangan di P2 yang saya perbaiki:
1. **`harga_satuan_beli` ditambahkan ke DETAIL_PESANAN.** AB-04 mewajibkan harga saat transaksi dikunci, tetapi P2 hanya menyimpan subtotal. Tanpa harga satuan, subtotal tidak bisa dihitung ulang.
2. **`status_member` ditambahkan ke PENGGUNA.** AB-02 memberi diskon kepada member aktif, tetapi P2 tidak punya atribut status member. Tanpa atribut ini, aturan diskon tidak bisa diuji.

## 7. Skema Relasional Akhir (3NF)

```
PENGGUNA         (id_pengguna, nama_lengkap, email_pengguna UNIQUE, kata_sandi, nomor_telepon, peran_akses, status_member)
KATEGORI_PRODUK  (id_kategori, nama_kategori, deskripsi_kategori)
PRODUK           (id_produk, id_kategori*, nama_produk, merek_produk, harga_produk, stok_produk, deskripsi_produk, gambar_utama, rating_produk)
PESANAN          (id_pesanan, id_pembeli*, tanggal_pesanan, alamat_tujuan, kota_kabupaten, diskon_pesanan, status_pesanan)
DETAIL_PESANAN   (id_pesanan*, id_produk*, jumlah_beli, harga_satuan_beli)   -- kunci: (id_pesanan, id_produk)
PEMBAYARAN       (id_pembayaran, id_pesanan* UNIQUE, metode_pembayaran, tanggal_pembayaran, jumlah_transfer, bukti_pembayaran, status_pembayaran)
PENGIRIMAN_RESI  (id_pengiriman, id_pesanan* UNIQUE, kurir_ekspedisi, nomor_resi UNIQUE, tanggal_dikirim, tanggal_diterima, nama_penerima, bukti_paket_sampai, status_pengiriman)
BANNER_UTAMA     (id_banner, judul_banner, gambar_banner, tautan_promo, urutan_tampil)
-- garis bawah = kunci primer, * = kunci tamu
```

`BANNER_UTAMA` tidak punya dependensi fungsional dengan tabel transaksi, sehingga tidak perlu dipecah.

## 8. Pemeriksaan BCNF

| Tabel | Penentu yang ada | Kunci kandidat | Hasil |
|---|---|---|---|
| PENGGUNA | id_pengguna; email_pengguna | id_pengguna, email_pengguna | Memenuhi BCNF |
| KATEGORI_PRODUK | id_kategori | id_kategori | Memenuhi BCNF |
| PRODUK | id_produk | id_produk | Memenuhi BCNF |
| PESANAN | id_pesanan | id_pesanan | Memenuhi BCNF |
| DETAIL_PESANAN | (id_pesanan, id_produk) | (id_pesanan, id_produk) | Memenuhi BCNF |
| PEMBAYARAN | id_pembayaran; id_pesanan | id_pembayaran, id_pesanan | Memenuhi BCNF |
| PENGIRIMAN_RESI | id_pengiriman; id_pesanan; nomor_resi | ketiganya | Memenuhi BCNF |

---

