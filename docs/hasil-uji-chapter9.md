# HASIL UJI CHAPTER 9 — LANGKAH 11–14

**Nama penguji:** Hafizul Hanif  
**NIM:** 23367025  
**Mata kuliah:** Pengembangan Web  
**Proyek:** SisaBaik — Vue 3  
**Tanggal pengujian:** 1 November 2026  
**Branch:** `feature/vue-components`  
**Commit yang telah dibuktikan:** `9c528a1` — `feat: build SisaBaik Vue components`  
**Versi Node dan npm:** Belum tercatat; isi setelah menjalankan `node --version` dan `npm --version`.  
**Lingkungan:** Visual Studio Code, Firefox, frontend `http://127.0.0.1:5173/`.

> **Ruang lingkup:** Pengujian frontend menggunakan enam data *fixture* lokal. Chapter 9 belum mengintegrasikan REST API, autentikasi, maupun transaksi nyata dari backend Chapter 8. Kolom **belum diverifikasi** berarti belum ada bukti pengujian aktual, bukan berarti pengujian gagal.

## Langkah 11 — Melengkapi gaya dan memeriksa responsivitas

**Tujuan:** Memeriksa stylesheet, identitas visual, tampilan katalog, responsivitas pada lebar 375/768/1280 px, dan aksesibilitas keyboard dasar.

| Pemeriksaan | Hasil aktual berdasarkan screenshot | Status |
| --- | --- | --- |
| Logo, header, hero, dan ilustrasi | Muncul pada tampilan beranda SisaBaik | Lulus (desktop) |
| Kartu katalog dan styling | Tiga kartu dan gambar kategorinya tampil konsisten | Lulus (desktop) |
| Data pembeli awal | 3 penawaran, 15 unit, 2 penyedia | Lulus |
| Harga gratis | Sayur Layak Olah berlabel **Gratis** | Lulus |
| Ringkasan mode penyedia | Belum ada screenshot mode Penyedia Rina | Belum diverifikasi |
| Tampilan responsif 525 × 825 px | Hero dan header tersusun vertikal pada Firefox Responsive Design Mode; belum membuktikan bebas scroll horizontal seluruh halaman | Sebagian terverifikasi |
| Tampilan lebar 375 px | Belum ada screenshot DevTools pada 375 px | Belum diverifikasi |
| Tampilan lebar 768 px | Belum ada screenshot DevTools pada 768 px | Belum diverifikasi |
| Tampilan lebar 1280 px | Tampilan desktop ada, tetapi ukuran viewport tepat belum dicatat | Belum diverifikasi pada ukuran tepat |
| Tab, Shift+Tab, fokus, dan tautan lewati konten | Belum ada catatan pengujian keyboard | Belum diverifikasi |

Pengamatan awal menunjukkan antarmuka desktop sudah dapat dibaca dan digunakan. Belum ada dasar untuk menyatakan lulus seluruh pengujian responsivitas atau audit WCAG.
## Langkah 12 — Menelusuri perubahan state melalui skenario

| No | Skenario modul | Hasil aktual yang terlihat | Status |
| --- | --- | --- | --- |
| 1 | Mode pembeli setelah refresh | 3 kartu; 15 unit; 2 penyedia | Lulus |
| 2 | Pencarian `LAPAU` | Pencarian `lapau` menghasilkan 2 kartu Lapau Uni Rina; **case-insensitive belum diuji langsung** | Sebagian terverifikasi |
| 3 | Pencarian kata tanpa hasil | `zzzz` menghasilkan 0 kartu dan pesan belum ada makanan yang cocok | Lulus |
| 4 | Filter kategori roti | 1 kartu Paket Roti Sore | Lulus |
| 5 | Pilih Paket Roti Sore | Panel Pilihan Anda muncul dengan Rp15.000/paket; stok pada kartu tetap 6 berdasarkan dua screenshot | Lulus untuk tampilan |
| 6 | Beralih ke mode penyedia | Belum ada bukti ringkasan 3 penawaran, 2 tersedia, 9 unit | Belum diverifikasi |
| 7 | Kirim formulir kosong | Belum ada bukti validasi form | Belum diverifikasi |
| 8 | Harga normal Rp20.000, penawaran Rp30.000 | Belum ada bukti penolakan harga | Belum diverifikasi |
| 9 | Form valid Paket Gratis Latihan | Belum ada bukti penambahan data dan nilai 4/3/11 | Belum diverifikasi |
| 10 | Kembali ke pembeli setelah tambah | Belum ada bukti 4 kartu, 17 unit, 2 penyedia | Belum diverifikasi |
| 11 | Beralih lagi ke penyedia | Belum ada bukti form reset tetapi data tetap ada | Belum diverifikasi |
| 12 | Refresh setelah tambah | Belum ada bukti data baru hilang dari memori | Belum diverifikasi |
| 13 | Logo, ilustrasi, badge, harga gratis | Tampak logo, gambar kategori, diskon dan label Gratis | Lulus untuk tampilan |
| 14 | 375, 768, 1280 px dan keyboard | Belum ada bukti untuk seluruh kondisi | Belum diverifikasi |
| 15 | Penawaran otomatis kedaluwarsa setelah 15 detik | Belum ada bukti perubahan ke 2 kartu dan 11 unit | Belum diverifikasi |

**Penghitungan ringkasan pembeli:** Paket Roti Sore (6 paket) + Nasi Padang Hemat (4 porsi) + Sayur Layak Olah (5 paket) = **15 unit stok**, dalam **3 penawaran tersedia** dari **2 penyedia unik**. Satuan stok tidak dianggap sebagai berat makanan. Ringkasan tetap 3/15/2 saat filter hanya menyisakan satu kartu, karena ringkasan dihitung dari semua penawaran tersedia, bukan hasil filter.

**Penghitungan ringkasan penyedia menurut *fixture* modul (ekspektasi, belum diuji langsung):** Lapau Uni Rina memiliki tiga penawaran, mencakup satu *draft*; dua penawaran tersedia dengan stok 4 + 5 = **9 unit**. Angka ini harus dibandingkan dengan tampilan mode penyedia saat diuji, bukan langsung dinyatakan lulus.

**Penjelasan aliran data berdasarkan kode `App.vue` yang diperiksa:** `App.vue` menyimpan `penawaran`, `pencarian`, `kategori`, `pilihanId`, dan waktu pada `ref`; `computed` menghitung katalog serta jumlah stok. `FilterPenawaran` menerima `props` dan mengirim `ubah-pencarian`, `ubah-kategori`, serta `reset`. `KartuPenawaran` menerima data kartu dan mengirim `pilih`; induk mengubah `pilihanId` dan memberi fokus pada ringkasan melalui `nextTick`. `FormPenawaran` mengirim `simpan` kepada induk yang menambah penawaran ke array lokal. Bukti pengujian form tetap perlu disiapkan.
## Langkah 13 — Membuat build dan memeriksa hasilnya

**Perintah yang diuji:**

```bash
cd frontend
npm run build
```

**Hasil aktual:** Terminal menunjukkan `sisabaik-frontend@1.0.1`, Vite `v8.3.3`, **23 modules transformed**, output `frontend/dist` yang mencakup `index.html` dan berkas `assets`, serta **built in 346ms**. **Status: LULUS.** Kesalahan `Missing script: build` sebelumnya terjadi karena perintah dijalankan di root repositori backend; setelah pindah ke `frontend`, build berjalan berhasil.

**Pembaruan preview (bukti terbaru):** `npm run preview` berhasil menjalankan server Vite pada `http://127.0.0.1:4173/`. Screenshot membuktikan server preview aktif, tetapi belum memperlihatkan browser mengakses port 4173. Karena itu pengujian interaksi katalog, filter, dan formulir pada preview **belum diverifikasi**. Build terbaru berhasil memproses 23 modul dalam **445 ms** (durasi dapat berbeda antar-run).
## Langkah 14 — Mendokumentasikan dan menyimpan kemajuan proyek

**Bukti Git:** Screenshot terminal menunjukkan commit `9c528a1` pada branch `feature/vue-components`, dengan pesan `feat: build SisaBaik Vue components`. Output commit memperlihatkan 21 file diubah. Pemeriksaan `git check-ignore` juga menunjukkan `frontend/node_modules` dan `frontend/dist` diabaikan. File `frontend/package-lock.json` terlihat masuk dalam daftar commit.

**Dokumentasi yang perlu tersedia di root proyek:** `README.md` dan `docs/hasil-uji-chapter9.md`. README harus menjelaskan Node dan npm, cara instalasi, menjalankan `dev/build/preview`, aset SVG lokal, penggunaan *fixture* di memori, serta bahwa frontend Chapter 9 belum memanggil API. Jangan mengubah dokumentasi backend Chapter 8 yang sudah benar.

**Catatan status:** Commit kode awal sudah terbukti, tetapi **commit yang memasukkan laporan ini belum dibuktikan**. Setelah menyalin folder `docs`, mengisi bukti uji yang belum ada, dan memperbarui README, jalankan dari root repositori:

```bash
git status
git check-ignore frontend/node_modules frontend/dist
git add frontend README.md docs/hasil-uji-chapter9.md
git diff --staged --stat
git commit -m "docs: record Chapter 9 tests for steps 11-14"
git status
```

Jalankan `git push -u origin feature/vue-components` **hanya jika** remote GitHub kelas memang telah dikonfigurasi. Jangan memasukkan `.env`, kata sandi, JWT, atau data akun asli ke dokumentasi.
## Pengujian lanjutan yang perlu dikerjakan

1. Periksa mode Penyedia Rina, lalu uji formulir kosong, harga tidak masuk akal, angka stok 1,5, harga kosong, tanggal kemarin, nama pendek, serta form valid dengan harga 0.
2. Bandingkan jumlah data sesudah menambah penawaran dan setelah melakukan refresh.
3. Periksa ukuran 375, 768, dan 1280 px di DevTools, tidak ada scroll horizontal, serta fokus keyboard.
4. Uji batas waktu 15 detik sesuai petunjuk modul, lalu **pulihkan data fixture** ke nilai semula.
5. Buka `http://127.0.0.1:4173/` dan uji kartu, filter, serta form pada preview yang sudah berhasil dijalankan; catat versi Node.
6. Lakukan dan catat hasil aktual pengujian form invalid, form valid, mobile, dan preview sebelum mengklaim seluruh skenario lulus.

## Kesimpulan sementara

Langkah 11 sudah menunjukkan tampilan katalog desktop dan identitas visual berjalan. Pada Langkah 12, katalog, filter, pencarian, *empty state*, dan pemilihan makanan berhasil ditunjukkan melalui tangkapan layar; pengujian form, responsivitas, dan kedaluwarsa belum lengkap. Langkah 13 telah lulus build produksi dan server preview berhasil dijalankan; tampilan/interaksi pada alamat preview 4173 masih perlu diuji. Pada Langkah 14, commit kode awal sudah terbukti, sedangkan dokumentasi pengujian perlu ditambahkan ke repositori. Laporan ini memisahkan **hasil aktual** dari **ekspektasi modul** agar tidak mengklaim pengujian yang belum dilaksanakan.

## Pembaruan pengamatan 1 November 2026 (22.40 WIB)

- **Langkah 11 — Responsif:** Firefox Responsive Design Mode memperlihatkan viewport **525 × 825 px** dengan `DPR 1.25`. Logo, label Tampilan, heading dan tombol hero tampak tersusun sesuai lebar yang sempit. Ini **bukan** pengganti pengujian 375, 768, 1280 px maupun pemeriksaan scrollbar horizontal.
- **Langkah 11 — Desktop:** Katalog tetap memperlihatkan tiga penawaran awal, stok total 15, dan dua penyedia, dengan label Gratis pada Sayur Layak Olah.
- **Langkah 13 — Build dan preview:** Build pada Vite 8.3.3 selesai dalam **445 ms**, **23 modul** ditransformasi, dan perintah `npm run preview` menghasilkan `Local: http://127.0.0.1:4173/`. Status **build lulus** dan **server preview aktif**; pemeriksaan browser pada port 4173 belum ada bukti.

- **Langkah 14 — README:** Bagian `Frontend Vue (Chapter 9)` yang dikirim sudah memuat perintah dev/build/preview, aset SVG, fixture di memori, dan belum adanya request API. Namun bagian sebelumnya masih menyebut status Chapter 7 dan mengatakan autentikasi belum tersedia; ini tidak selaras dengan catatan Chapter 9 tentang backend Chapter 8. Perlu diselaraskan dengan kondisi backend aktual, **tanpa menghapus materi lama**. Perhatikan juga agar Markdown disimpan dengan backtick normal, tidak memuat escape `\` atau bold ganda `**\*`.

- **Catatan editor:** Screenshot terbaru memperlihatkan **6 masalah** di panel Problems pada `App.vue`. Pesan rinci belum tersedia sehingga statusnya tidak dapat disimpulkan; build yang lulus tidak otomatis menjadikan hasil pemeriksaan editor lulus.
