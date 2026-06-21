## Hi there 👋

# Sistem Pendaftaran Magang — BMKG Maritim Tanjung Perak Surabaya

Dokumentasi ini menjelaskan cara kerja, cara penggunaan, dan cara pemeliharaan **Sistem Pendaftaran Magang Online** untuk Stasiun Meteorologi Maritim Kelas II Tanjung Perak Surabaya.

---

## 1. Gambaran Umum Sistem

Sistem ini terdiri dari dua bagian utama yang saling terhubung:

| Bagian | Fungsi | Lokasi |
|---|---|---|
| **Website Pendaftaran** (`index.html`) | Halaman publik yang diakses calon pendaftar untuk mengisi formulir magang | Hosting (misal GitHub Pages) |
| **Backend / Server** (`Code.gs`) | Menyimpan data pendaftar ke Google Sheets, menyimpan berkas ke Google Drive, mengatur kuota & jadwal batch | Google Apps Script |

Cara kerja singkatnya:

```
[Calon Pendaftar] → mengisi form di Website
       ↓
[Website] → mengirim data via internet ke Google Apps Script
       ↓
[Google Apps Script] → menyimpan data ke Google Sheets
                     → menyimpan berkas (CV, Transkrip, dll) ke Google Drive
       ↓
[Admin BMKG] → membuka Google Sheets untuk melihat & memverifikasi pendaftar
```

Tidak ada server tambahan yang perlu disewa atau dibayar — semua berjalan gratis di atas akun Google (Sheets, Drive, Apps Script) dan hosting statis (GitHub Pages).

---

## 2. Konsep Sistem Batch (Gelombang Pendaftaran)

Pendaftaran magang dibagi menjadi **7 batch per tahun**, masing-masing mengikuti ritme kalender akademik kampus di Indonesia (semester ganjil/genap dan masa libur).

| Batch | Periode Pendaftaran Dibuka | Mulai Magang |
|---|---|---|
| Batch 1 | Desember – Januari | Februari |
| Batch 2 | Februari | Maret |
| Batch 3 | Maret – April | Mei |
| Batch 4 | Mei – Juni | Juli |
| Batch 5 | Juli | Agustus |
| Batch 6 | Agustus – September | Oktober |
| Batch 7 | Oktober – November | Desember |

**Aturan penting yang berlaku otomatis:**

1. **Batch ditentukan otomatis dari tanggal hari ini** — bukan dipilih oleh pendaftar. Kalau hari ini bulan Mei, sistem otomatis menempatkan pendaftar baru ke Batch 4, tanpa perlu ada yang mengatur manual.
2. **Form hanya bisa diisi saat periode batch sedang berjalan.** Karena ketujuh batch ini menutup penuh 12 bulan tanpa celah, di bulan apa pun selalu ada satu batch yang sedang terbuka untuk diisi.
3. **Tanggal mulai magang otomatis terisi** sesuai jadwal batch (misalnya daftar di periode Batch 4 → tanggal mulai otomatis terisi 1 Juli), dan pendaftar **tidak bisa mengubahnya sendiri**.
4. **Kuota dihitung ulang dari nol setiap kali batch baru dibuka.** Kuota tidak terpengaruh oleh peserta batch sebelumnya yang masih menjalani magang (termasuk yang mengambil program 5 bulan).
5. **Validasi batch dilakukan dua kali** — sekali di tampilan website (untuk pengalaman pengguna), dan sekali lagi di server Google Apps Script (sebagai sumber kebenaran akhir, tidak bisa dimanipulasi dari sisi pengguna).
6. **Teks tahun di halaman web mengikuti tahun berjalan secara otomatis.** Saat memasuki tahun 2027, semua teks "2026" di halaman (judul, footer, dll) otomatis berubah menjadi "2027" tanpa perlu mengedit kode. *(Catatan: ini hanya teks tampilan — data pendaftar tahun-tahun sebelumnya tidak pernah terhapus oleh perubahan ini.)*

---

## 3. Tiga Pilihan Durasi Program

| Durasi | Kuota per Batch |
|---|---|
| 1 Bulan | 10 slot |
| 3 Bulan | 10 slot |
| 5 Bulan | 10 slot |

Kuota berlaku **per durasi, per batch**. Artinya tiap batch baru dibuka, ketiga durasi ini kembali punya 10 slot kosong masing-masing.

---

## 4. Data yang Tersimpan di Google Sheets

Setiap pendaftaran baru otomatis menambah satu baris di Google Sheets dengan kolom-kolom berikut:

| Kolom | Isi |
|---|---|
| Reg ID | Nomor unik pendaftaran (format: `BMKG-xxxxxx`) |
| Timestamp | Waktu submit (sesuai jam pendaftar) |
| Batch | Nama batch & bulan mulai magang |
| Batch No | Nomor batch (1–7), untuk keperluan filter/hitung |
| Nama, NIM, Email, No. WhatsApp | Data diri pendaftar |
| Universitas, Jurusan, Program Studi, Semester, IPK | Data akademik |
| Durasi | 1 / 3 / 5 Bulan |
| Nama Program | Nama paket magang sesuai durasi |
| Tanggal Mulai, Tanggal Selesai | Dihitung otomatis oleh server |
| Keahlian | Isian opsional dari pendaftar |
| Link CV, Link Transkrip, Link Proposal, Link Foto | Tautan langsung ke berkas di Google Drive |
| Status | Default: "Menunggu Verifikasi" — **kolom ini admin yang ubah manual** |

---

## 5. Struktur Penyimpanan Berkas di Google Drive

Berkas yang diunggah pendaftar (CV, Transkrip, Proposal, Pas Foto) otomatis tersimpan rapi dengan struktur folder:

```
📁 Folder Utama 
 └── 📁 Batch 1
 │    └── 📁 Nama Pendaftar - BMKG-123456
 │         ├── CV_nama_file.pdf
 │         ├── Transkrip_nama_file.pdf
 │         ├── Proposal_nama_file.pdf
 │         └── Foto_nama_file.jpg
 └── 📁 Batch 2
      └── ...
```

Setiap berkas otomatis diatur agar **siapa pun dengan link bisa melihat** (tidak perlu login Google), supaya admin yang membuka link dari Sheets bisa langsung melihat file tanpa kendala izin akses.

---

## 6. Panduan untuk Admin

### A. Cara Memantau Pendaftar Baru

1. Buka Google Sheets yang sudah disiapkan.
2. Baris baru akan otomatis muncul setiap ada pendaftaran masuk — **tidak perlu refresh manual**, cukup buka ulang Sheets-nya.
3. Untuk memeriksa berkas pendaftar, klik kolom **Link CV / Link Transkrip / Link Proposal / Link Foto** — akan langsung membuka file di Google Drive.

### B. Cara Memverifikasi Pendaftar

Kolom **Status** di paling kanan adalah kolom kerja admin. Disarankan menggunakan label konsisten, misalnya:

- `Menunggu Verifikasi` (default otomatis dari sistem)
- `Diterima`
- `Ditolak`
- `Dibatalkan`

> **Penting:** Status `Dibatalkan` punya efek khusus — baris dengan status ini **tidak dihitung** dalam kuota batch. Jadi kalau ada pendaftar yang mengundurkan diri atau datanya tidak valid, ubah statusnya jadi `Dibatalkan` agar slot kuotanya otomatis kembali tersedia untuk pendaftar lain di batch yang sama.

### C. Cara Mengecek Kuota Batch yang Sedang Berjalan

Kuota bisa dicek dengan dua cara:

1. **Lewat website** — buka halaman pendaftaran, kuota tiap durasi program otomatis tertampil dan ter-update sesuai data Sheets.
2. **Lewat Sheets manual** — filter kolom **Batch No** sesuai batch yang sedang berjalan, lalu hitung baris yang **bukan** berstatus `Dibatalkan`, dikelompokkan per kolom **Durasi**.

### D. Mengubah Jadwal Batch (jika kebijakan BMKG berubah)

Jadwal 7 batch (bulan pendaftaran & bulan mulai magang) ditulis di dua tempat dan **harus selalu sama** di keduanya:

- File `index.html`, bagian `BATCH_SCHEDULE`
- File `Code.gs`, bagian `BATCH_SCHEDULE`

Kalau BMKG ingin mengubah jadwal batch (misalnya menyesuaikan kalender akademik tahun ajaran baru), kedua bagian ini harus diedit bersamaan oleh seseorang yang familiar dengan kode (developer/IT), karena formatnya berupa kode JavaScript, bukan teks bebas.

### E. Mengubah Kuota Maksimal per Batch

Cari baris `QUOTA_MAX` di `Code.gs` (saat ini diset `10`). Ubah angkanya sesuai kebutuhan, lalu **deploy ulang** Apps Script (lihat bagian 8).

### F. Backup Data

Karena seluruh data tersimpan permanen di Google Sheets & Drive milik akun BMKG sendiri, disarankan:

- Sesekali download Sheets sebagai cadangan (**File → Download → Microsoft Excel/CSV**)
- Jangan menghapus folder Drive utama, karena seluruh tautan berkas pendaftar mengacu ke folder tersebut

---

## 7. Penjelasan Teknis Singkat (untuk pengetahuan internal / IT BMKG)

- **Frontend**: satu file HTML mandiri (`index.html`) dengan CSS dan JavaScript di dalamnya, tidak membutuhkan server tambahan — bisa dihosting statis di GitHub Pages atau penyedia hosting statis lainnya.
- **Backend**: Google Apps Script (`Code.gs`), berjalan sebagai *Web App* yang menerima dua jenis permintaan:
  - `GET` — mengambil status batch & kuota terkini (dipanggil otomatis saat halaman dimuat)
  - `POST` — menerima dan menyimpan data pendaftaran baru
- **Validasi ganda**: batch dan kuota dicek di sisi tampilan (untuk kenyamanan pengguna) **dan** dicek ulang di server (sebagai keputusan akhir yang tidak bisa dimanipulasi dari browser pengguna).
- **Tidak ada database eksternal** — Google Sheets berperan sebagai database, Google Drive sebagai penyimpanan berkas.
- **Zona waktu**: semua tanggal dihitung menggunakan zona waktu skrip (`Asia/Jakarta` sebagai default).

---

## 8. Cara Deploy / Memasang Ulang Sistem (jika perlu pindah akun atau server baru)

1. **Buat Google Sheets baru** sebagai database, salin ID-nya dari URL.
2. **Buat folder Google Drive baru** sebagai penyimpanan berkas, salin ID-nya dari URL.
3. Buka Google Sheets → **Extensions → Apps Script**, lalu tempel isi `Code.gs`.
4. Isi `SHEET_ID` dan `DRIVE_FOLDER_ID` di bagian atas `Code.gs` sesuai ID yang sudah disalin.
5. Klik **Deploy → New deployment → Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Salin **Web app URL** hasil deploy (format: `https://script.google.com/macros/s/XXXXXX/exec`).
7. Tempel URL tersebut ke variabel `GOOGLE_SCRIPT_URL` di `index.html`.
8. Unggah `index.html` ke hosting (misalnya GitHub Pages).

**Catatan:** setiap kali isi `Code.gs` diedit ulang, harus dilakukan **deployment baru** (Deploy → Manage deployments → ikon pensil → New version → Deploy) agar perubahan berlaku di URL yang sudah terpasang di website.

---

## 9. Berkas yang Diserahkan

| Berkas | Keterangan |
|---|---|
| `index.html` | Halaman website pendaftaran (frontend) |
| `Code.gs` | Kode backend Google Apps Script |
| `README.md` / dokumen ini | Panduan penggunaan & pemeliharaan sistem |

---

## 10. Kontak Pengembang

Sistem ini dikembangkan sebagai bagian dari program **Magang Berdampak** di Stasiun Meteorologi Maritim Kelas II Tanjung Perak Surabaya. Untuk pertanyaan teknis lebih lanjut terkait modifikasi sistem, dapat menghubungi pengembang melalui kontak yang telah diserahkan terpisah kepada pihak BMKG.
