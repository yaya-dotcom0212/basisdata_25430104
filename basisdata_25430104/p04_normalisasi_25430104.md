# Normalisasi Proyek Toko Daring Lampung - Pertemuan 4

**Nama:** Dea Novita  **NIM:** 25430104  **Kelas:** D **Tanggal:** 17 Oktober 2026

> Sumber: `p02_kebutuhan_data_toko_daring_lampung_104.md` (Modul 2). Aturan bisnis AB-01 sampai AB-07 dan entitas diambil dari dokumen tersebut.



## 1. Dokumen Sumber

- **Nama dokumen:** Invoice/Struk Pesanan Daring, Bukti Pembayaran Digital, dan Resi Pengiriman Logistik
- **Aturan bisnis yang dipakai:** AB-01 sampai AB-07

**Asumsi yang dipakai:**
- A1: setiap pesanan memiliki tepat satu bukti pembayaran.
- A2: setiap pesanan memiliki tepat satu resi pengiriman.
- A3: `berat_paket` diambil dari dokumen sumber 4 (resi), karena tidak ada di daftar entitas P2.

## 2. UNF (Bentuk Tak Ternormalisasi)

```
PESANAN_UNF (no_pesanan, tgl_pesanan, id_pelanggan, nama_pelanggan, email_pelanggan,
             no_hp_pelanggan, alamat_lampung, status_member, poin_loyalitas,
             diskon_member, ongkos_kirim, total_harga, status_pesanan,
             { kode_produk, nama_produk, kategori_khas, qty_pesanan,
               harga_satuan_pesanan, subtotal },
             kode_pembayaran, metode_bayar, tgl_bayar, jumlah_bayar, status_pembayaran,
             no_resi, nama_ekspedisi, berat_paket, tgl_kirim, tgl_diterima,
             status_pengiriman)
```

## 3. Dependensi Fungsional

| Kode | Dependensi fungsional | Dasar |
|---|---|---|
| FD1 | no_pesanan → tgl_pesanan, id_pelanggan, diskon_member, ongkos_kirim, status_pesanan | Invoice |
| FD2 | id_pelanggan → nama_pelanggan, email_pelanggan, no_hp_pelanggan, alamat_lampung, status_member, poin_loyalitas | Formulir profil |
| FD3 | email_pelanggan → id_pelanggan; no_hp_pelanggan → id_pelanggan | AB-06 (unik) |
| FD4 | kode_produk → nama_produk, kategori_khas, harga_jual, stok_produk, min_stok | Katalog produk |
| FD5 | (no_pesanan, kode_produk) → qty_pesanan, harga_satuan_pesanan | AB-04 (harga dikunci) |
| FD6 | (qty_pesanan, harga_satuan_pesanan) → subtotal | Nilai turunan |
| FD7 | no_pesanan → kode_pembayaran, metode_bayar, tgl_bayar, jumlah_bayar, status_pembayaran | A1 |
| FD8 | no_pesanan → no_resi, nama_ekspedisi, berat_paket, tgl_kirim, tgl_diterima, status_pengiriman | A2 |
| FD9 | no_resi → no_pesanan | AB-06 (resi unik) |

**Titik analisis:** AB-02 memberi diskon 5% kepada member aktif, sehingga `diskon_member` bergantung pada `status_member`. Pertanyaannya, apakah `diskon_member` bisa dihitung ulang dari data pelanggan yang sekarang? Jawabannya tidak, karena status member bisa berubah sesudah transaksi. Ini dibahas di bagian 5.

## 4. Tahapan Normalisasi

### 4.1 Menuju 1NF
Hilangkan grup berulang dengan satu baris per produk per pesanan. Kunci primer menjadi (no_pesanan, kode_produk).

**Bukti L.1 (tahapan UNF sampai 3NF):**

![alt text](tahapan-1.png)

### 4.2 Menuju 2NF
Ketergantungan parsial dipisah:
- FD1, FD2, FD7, dan FD8 hanya bergantung pada `no_pesanan`, sehingga masuk tabel PESANAN.
- FD4 hanya bergantung pada `kode_produk`, sehingga masuk tabel PRODUK.
- FD5 dan FD6 bergantung pada kunci penuh, sehingga masuk tabel DETAIL_PESANAN.

```
PESANAN        (no_pesanan, tgl_pesanan, id_pelanggan, nama_pelanggan, email_pelanggan,
                no_hp_pelanggan, alamat_lampung, status_member, poin_loyalitas,
                diskon_member, ongkos_kirim, status_pesanan,
                kode_pembayaran, metode_bayar, ..., no_resi, nama_ekspedisi, ...)
PRODUK         (kode_produk, nama_produk, kategori_khas, harga_jual, stok_produk, min_stok)
DETAIL_PESANAN (no_pesanan, kode_produk, qty_pesanan, harga_satuan_pesanan, subtotal)
```

### 4.3 Menuju 3NF
Ketergantungan transitif dipisah:
- `no_pesanan → id_pelanggan → nama, email, telepon, alamat, status_member, poin` (FD2): dipisah menjadi PELANGGAN.
- Pembayaran dan pengiriman punya kunci sendiri, sehingga dipisah menjadi PEMBAYARAN dan PENGIRIMAN. Keduanya diberi FK `no_pesanan` yang UNIQUE (asumsi A1 dan A2).

Hasil akhir 3NF ada di bagian 7.

## 5. Keputusan Nilai Turunan

| Nilai | Disimpan / dihitung | Alasan |
|---|---|---|
| `subtotal` | Dihitung | = `qty_pesanan` × `harga_satuan_pesanan`. Harga satuan sudah dikunci (AB-04), jadi hasilnya selalu sama. |
| `total_harga` | Dihitung | = SUM(`subtotal`) − `diskon_member` + `ongkos_kirim`. Jika disimpan, bisa tidak cocok dengan detailnya. |
| `diskon_member` | Disimpan | Bergantung pada status member saat transaksi (AB-02). Status bisa berubah kemudian, sehingga diskon harus disimpan sebagai fakta historis. |
| `jumlah_bayar` | Disimpan | Uang yang benar-benar dibayarkan. Bisa dibandingkan dengan `total_harga` untuk memeriksa pembayaran. |
| `poin_loyalitas` | Disimpan | Saldo berjalan di PELANGGAN. Tidak bisa dihitung dari satu pesanan saja, karena bisa berkurang saat ditukarkan. |

**Titik analisis:** Mengapa `diskon_member` disimpan padahal bisa dihitung dari subtotal? Karena status member bisa berubah sesudah transaksi. Jika diskon dihitung ulang, nota lama bisa berubah nilainya. Ini prinsip yang sama dengan AB-04 untuk harga.

**Catatan AB-07:** poin dihitung per kelipatan Rp10.000. Saya mengasumsikan dasar hitungnya adalah `total_harga` setelah diskon. Asumsi ini perlu dikonfirmasi dengan pemilik sistem.

## 6. Perbaikan dari P2

Normalisasi tidak menemukan dependensi yang melanggar, tetapi ada satu catatan dari P2:
- `total_harga` di P2 ditulis sebagai atribut tersimpan. Saya mengubahnya menjadi nilai turunan agar tidak bisa tidak konsisten dengan detailnya.

## 7. Skema Relasional Akhir (3NF)

```
PELANGGAN        (id_pelanggan, nama_pelanggan, email_pelanggan UNIQUE, no_hp_pelanggan UNIQUE, alamat_lampung, status_member, poin_loyalitas)
PRODUK           (kode_produk, nama_produk, kategori_khas, harga_jual, stok_produk, min_stok)
PESANAN          (no_pesanan, tgl_pesanan, id_pelanggan*, diskon_member, ongkos_kirim, status_pesanan)
DETAIL_PESANAN   (no_pesanan*, kode_produk*, qty_pesanan, harga_satuan_pesanan)   -- kunci: (no_pesanan, kode_produk)
PEMBAYARAN       (kode_pembayaran, no_pesanan* UNIQUE, tgl_bayar, metode_bayar, jumlah_bayar, status_pembayaran)
PENGIRIMAN       (no_resi, no_pesanan* UNIQUE, nama_ekspedisi, berat_paket, tgl_kirim, tgl_diterima, status_pengiriman)
PETUGAS          (kode_petugas, nama_petugas, peran_petugas)
-- garis bawah = kunci primer, * = kunci tamu
```

`PETUGAS` tidak punya dependensi fungsional dengan tabel transaksi, sehingga tidak perlu dipecah.

## 8. Pemeriksaan BCNF

| Tabel | Penentu yang ada | Kunci kandidat | Hasil |
|---|---|---|---|
| PELANGGAN | id_pelanggan; email_pelanggan; no_hp_pelanggan | ketiganya | Memenuhi BCNF |
| PRODUK | kode_produk | kode_produk | Memenuhi BCNF |
| PESANAN | no_pesanan | no_pesanan | Memenuhi BCNF |
| DETAIL_PESANAN | (no_pesanan, kode_produk) | (no_pesanan, kode_produk) | Memenuhi BCNF |
| PEMBAYARAN | kode_pembayaran; no_pesanan | keduanya | Memenuhi BCNF |
| PENGIRIMAN | no_resi; no_pesanan | keduanya | Memenuhi BCNF |
| PETUGAS | kode_petugas | kode_petugas | Memenuhi BCNF |

