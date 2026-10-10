# Kontrak API SisaBaik

Base URL lokal: `http://127.0.0.1:3000`.

Kontrak ini mengikuti implementasi Chapter 7 pada modul SisaBaik. Contoh format response bukan pengujian tambahan. ID hasil pengujian dapat berbeda pada pelaksanaan berikutnya.

## 1. Format dan tipe data

POST dan PUT harus mengirim header `Content-Type: application/json` dan body berupa object JSON. Parser membatasi ukuran body JSON hingga 20 KB. Field body yang tidak didukung ditolak.

Response berhasil:

```json
{
  "success": true,
  "data": {}
}
```

Response gagal:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "hargaPenawaran tidak boleh melebihi hargaNormal."
  }
}
```

Koleksi menambahkan `meta`. DELETE berhasil mengembalikan `204` dengan body kosong; client tidak memanggil `response.json()` untuk response tersebut.

| Nilai | Tipe dan aturan |
| --- | --- |
| ID | Response memakai string digit. Input menerima string digit positif atau number yang masih safe integer. Rentang ID sampai `9223372036854775807`, tanpa nol di depan. |
| Uang | Integer rupiah, bukan string; kapasitas kolom `NUMERIC(12,0)` sampai `999999999999`. |
| Waktu | String ISO 8601 dengan zona waktu, misalnya akhiran `Z` atau offset `+07:00`. Response memakai ISO dengan UTC. |
| Stok dan kuantitas | Integer JSON; angka berbentuk string dalam body tidak diterima. |

## 2. Endpoint yang tersedia

| Method | Path | Fungsi | Status berhasil |
| --- | --- | --- | --- |
| GET | `/api/health` | Memeriksa proses dan akses database. | 200 |
| GET | `/api/penawaran` | Membaca koleksi penawaran dengan pencarian, filter, dan paginasi. | 200 |
| GET | `/api/penawaran/:id` | Membaca detail penawaran. | 200 |
| POST | `/api/penawaran` | Membuat penawaran. | 201 dan header Location |
| PUT | `/api/penawaran/:id` | Mengganti seluruh atribut yang dapat diedit. | 200 |
| DELETE | `/api/penawaran/:id` | Menghapus penawaran tanpa riwayat pesanan. | 204, body kosong |
| GET | `/api/pesanan` | Membaca ringkasan pesanan dengan paginasi. | 200 |
| GET | `/api/pesanan/:id` | Membaca pesanan beserta detail items. | 200 |
| POST | `/api/pesanan` | Membuat pesanan dalam satu transaksi database. | 201 dan header Location |

PUT/DELETE pesanan, PATCH, login, pembayaran, dan endpoint pengambilan tidak disediakan pada kontrak Chapter 7.

## 3. Health check

```http
GET /api/health
```

Endpoint menjalankan pemeriksaan `SELECT 1` pada database. Jika berhasil, body memuat `success: true` dan object `data` dengan `status: "ok"`, `database: "connected"`, serta `waktu` pemeriksaan dalam ISO 8601.

## 4. Koleksi penawaran

```http
GET /api/penawaran?q=roti&kategori=roti&status=aktif&tersedia=true&page=1&limit=2
```

| Query | Default | Aturan dan fungsi |
| --- | --- | --- |
| `q` | String kosong | Teks setelah trim, maksimum 100 karakter; mencari potongan nama penawaran atau nama usaha tanpa membedakan huruf besar/kecil. |
| `kategori` | String kosong | Teks setelah trim, maksimum 40 karakter; pencocokan tepat. String kosong menonaktifkan filter. |
| `status` | String kosong | `draft`, `aktif`, `habis`, atau `kedaluwarsa`; string kosong menonaktifkan filter. |
| `tersedia` | `false` | Hanya menerima string query `true` atau `false`. `true` memilih status aktif, stok lebih dari nol, dan waktu belum berakhir. |
| `page` | `1` | Integer positif dari 1 sampai 10000. |
| `limit` | `10` | Integer positif dari 1 sampai 50. |

Parameter dapat digabung. Parameter di atas yang dikirim berulang akan ditolak, misalnya `q=roti&q=nasi` atau `page=1&page=2`. Query `tersedia=false` tidak menyaring hanya penawaran yang tidak tersedia; nilai tersebut menonaktifkan filter ketersediaan.

Pencarian memperlakukan input sebagai potongan teks literal. Karakter `%` dan `_` bukan wildcard. Data diurutkan dengan ID menurun; parameter pengurutan bebas tidak tersedia.

Contoh bentuk response ketika tidak ada hasil:

```json
{
  "success": true,
  "data": [],
  "meta": {
    "jumlah": 0,
    "page": 1,
    "limit": 10,
    "totalPages": 0,
    "q": "",
    "kategori": "",
    "status": "",
    "tersedia": false
  }
}
```

`meta.jumlah` adalah total seluruh baris yang cocok dengan filter. `totalPages` dihitung dari jumlah tersebut dan limit. Halaman yang valid tetapi melampaui halaman terakhir tetap menghasilkan `200` dan array kosong; total hasil tidak berubah menjadi nol hanya karena halaman tersebut kosong. COUNT dan SELECT halaman memakai filter serta snapshot transaksi baca yang sama dalam satu request.

## 5. Detail penawaran

```http
GET /api/penawaran/16
```

Response berhasil memuat satu object penawaran pada `data`:

| Field | Tipe |
| --- | --- |
| `id`, `penyediaId` | String digit |
| `penyedia`, `nama`, `kategori`, `satuan`, `status` | String |
| `hargaNormal`, `hargaPenawaran`, `stok` | Integer |
| `berakhirPada`, `dibuatPada` | String ISO 8601 |

ID tidak valid ditolak dengan `400 VALIDATION_ERROR`. ID yang valid tetapi tidak ditemukan menghasilkan `404 OFFER_NOT_FOUND`.

## 6. POST dan PUT penawaran

```http
POST /api/penawaran
Content-Type: application/json
```

```http
PUT /api/penawaran/:id
Content-Type: application/json
```

Seluruh field berikut wajib dikirim pada kedua method:

| Field | Tipe | Validasi |
| --- | --- | --- |
| `penyediaId` | String digit atau number safe integer | ID positif valid; merujuk penyedia dengan akun aktif dan peran penyedia. |
| `nama` | String | 3–120 karakter setelah trim. |
| `kategori` | String | 1–40 karakter setelah trim; mengikuti constraint database. |
| `hargaNormal` | Integer | 1–999999999999. |
| `hargaPenawaran` | Integer | 0–999999999999 dan tidak melebihi hargaNormal. |
| `stok` | Integer | 0–2147483647. |
| `satuan` | String | 1–20 karakter setelah trim. |
| `status` | String | `draft`, `aktif`, `habis`, atau `kedaluwarsa`. |
| `berakhirPada` | String | ISO 8601 dengan zona waktu, 20–35 karakter, tanggal/jam valid, dan berada di masa depan saat request diproses. |

Jangan mengirim `id`, `dibuatPada`, `penyedia`, atau kolom lain di luar daftar tersebut. Field API memakai camelCase, misalnya `hargaPenawaran`, bukan nama kolom SQL `harga_penawaran`.

Contoh POST melalui Console browser pada halaman `http://127.0.0.1:3000/`:

```javascript
const body = {
  penyediaId: "1",
  nama: "Roti uji dokumentasi",
  kategori: "roti",
  hargaNormal: 10000,
  hargaPenawaran: 5000,
  stok: 2,
  satuan: "paket",
  status: "aktif",
  berakhirPada: new Date(Date.now() + 86400000).toISOString()
};

const respons = await fetch("/api/penawaran", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body)
});

console.log(respons.status, respons.headers.get("Location"));
console.log(await respons.json());
```

Contoh membuat data baru; penyedia `1` harus ada dan aktif. Waktu dibuat relatif sehari ke depan agar contoh tidak menggunakan tanggal kedaluwarsa. POST menghasilkan `201`, object penawaran pada `data`, dan `Location: /api/penawaran/<id-baru>`.

PUT memakai body lengkap yang sama bentuknya, dengan path berisi ID penawaran yang akan diubah. `penyediaId` harus tetap sama dengan pemilik semula. Mengganti penyedia ditolak dengan `409 PROVIDER_IMMUTABLE`. ID dan `dibuatPada` tetap dipertahankan. Body PUT yang hanya berisi satu atribut ditolak dengan `400`.

## 7. DELETE penawaran

```http
DELETE /api/penawaran/:id
```

- `204` jika penawaran berhasil dihapus; body kosong.
- `404 OFFER_NOT_FOUND` jika ID valid tetapi penawaran tidak ada.
- `409 REFERENCE_CONFLICT` jika penawaran masih dirujuk riwayat pesanan.

Penawaran yang pernah dipesan dipertahankan untuk riwayat. Penghentian penawaran dapat dilakukan melalui PUT lengkap untuk mengubah status, tanpa menghapus relasi transaksi.

## 8. Koleksi dan detail pesanan

```http
GET /api/pesanan?page=1&limit=10
GET /api/pesanan/8
```

Koleksi mendukung `page` dan `limit` dengan default serta batas yang sama seperti koleksi penawaran. Filter `q`, `kategori`, `status`, dan `tersedia` tidak disediakan untuk koleksi pesanan. Koleksi diurutkan menurut ID menurun.

`data` pada koleksi berupa array ringkasan pesanan. Setiap ringkasan berisi `id`, `pembeliId`, `penyediaId`, `penyedia`, `namaPemesan`, `status`, `total`, `nilaiStokTerselamatkan`, dan `dibuatPada`. Koleksi menggunakan `meta` dengan `jumlah`, `page`, `limit`, dan `totalPages`, tanpa `items` pada setiap ringkasan.

Endpoint detail menambahkan array `items`. Contoh detail pesanan yang tercatat dalam pengujian tanggal 10 Oktober 2026:

```json
{
  "success": true,
  "data": {
    "id": "8",
    "pembeliId": "3",
    "penyediaId": "1",
    "penyedia": "Lapau Uni Rina",
    "namaPemesan": "Siti Rahma",
    "status": "menunggu-konfirmasi",
    "total": 10000,
    "nilaiStokTerselamatkan": 20000,
    "dibuatPada": "2026-10-10T12:47:40.806Z",
    "items": [
      {
        "penawaranId": "16",
        "nama": "Uji C7 1791636460376",
        "kuantitas": 2,
        "hargaSatuan": 5000,
        "subtotal": 10000,
        "nilaiStokTerselamatkan": 20000
      }
    ]
  }
}
```

ID pesanan yang valid tetapi tidak ditemukan menghasilkan `404 ORDER_NOT_FOUND`. Nilai detail di atas merupakan data pengujian tersimpan, bukan contoh yang menjamin ID `8` ada pada database baru.

## 9. POST pesanan

```http
POST /api/pesanan
Content-Type: application/json
```

Contoh body:

```json
{
  "emailPembeli": "siti.pembeli@example.test",
  "items": [
    {
      "penawaranId": "16",
      "kuantitas": 1
    }
  ]
}
```

Ganti `penawaranId` dengan ID penawaran aktif yang belum berakhir dan stoknya cukup. Penawaran `16` pada hasil uji sudah habis, sehingga body contoh tersebut tidak otomatis merupakan pesanan yang dapat berhasil pada database hasil pengujian.

| Field | Tipe | Validasi |
| --- | --- | --- |
| `emailPembeli` | String | 3–254 karakter setelah trim dan berbentuk email; harus sesuai akun pembeli aktif. |
| `items` | Array | Wajib berisi 1–10 object item. |
| `items[].penawaranId` | String digit atau number safe integer | ID positif valid; tidak boleh muncul dua kali dalam satu request. |
| `items[].kuantitas` | Integer | 1–1000, serta tidak melebihi stok penawaran. |

Body hanya menerima `emailPembeli` dan `items`; setiap item hanya menerima `penawaranId` dan `kuantitas`. `namaPemesan`, harga satuan, subtotal, total, status pesanan, dan ID pesanan tidak dikirim client. Data tersebut dibaca atau dihitung server.

Aturan transaksi:

1. Pembeli harus aktif dan berperan pembeli.
2. Semua penawaran dalam satu pesanan berasal dari **satu penyedia**.
3. Setiap penawaran berstatus aktif, belum berakhir, dan memiliki stok yang cukup.
4. Total nilai normal tidak melampaui `999999999999`.
5. Penawaran dikunci menurut urutan ID dalam satu transaksi. Jika ada kegagalan, perubahan dibatalkan melalui rollback.
6. Pesanan berhasil menyimpan header dan detail harga, mengurangi stok, serta mengubah status penawaran menjadi `habis` jika stok mencapai nol.

Response berhasil adalah `201`, dengan bentuk detail pesanan pada `data` termasuk `items`. Header Location berbentuk `/api/pesanan/<id-baru>`. Response dikirim setelah transaksi berhasil di-commit.

Harga satuan pada detail merupakan snapshot saat pesanan dibuat. Perubahan harga penawaran berikutnya tidak mengubah harga detail atau total pesanan lama. `subtotal` serta nilai stok detail dihitung database; total header dihitung dari detail.

POST dapat menghasilkan transaksi baru jika diulang. Setelah timeout atau respons transaksi sibuk, periksa riwayat dan state sebelum mengulang request.

## 10. Status HTTP dan kode error

| HTTP | Code | Kondisi |
| --- | --- | --- |
| 400 | `VALIDATION_ERROR` | ID, query, bentuk body, field, tipe, atau nilai input tidak sesuai. |
| 400 | `INVALID_JSON` | Body JSON tidak dapat diparse. |
| 400 | `DATA_CONSTRAINT` | Input melanggar constraint atau beberapa aturan tipe/rentang database yang dipetakan. |
| 404 | `ROUTE_NOT_FOUND` | Method/path tidak mempunyai handler pada aplikasi. |
| 404 | `OFFER_NOT_FOUND` | Penawaran dengan ID valid tidak ditemukan. |
| 404 | `ORDER_NOT_FOUND` | Pesanan dengan ID valid tidak ditemukan. |
| 404 | `BUYER_NOT_FOUND` | Akun pembeli aktif tidak ditemukan. |
| 404 | `PROVIDER_NOT_FOUND` | Penyedia dengan akun penyedia aktif tidak ditemukan. |
| 409 | `PROVIDER_IMMUTABLE` | PUT mencoba mengganti penyedia pemilik penawaran. |
| 409 | `OFFER_UNAVAILABLE` | Penawaran tidak aktif atau sudah berakhir. |
| 409 | `INSUFFICIENT_STOCK` | Kuantitas pesanan melebihi stok. |
| 409 | `MIXED_PROVIDER` | Items dalam satu pesanan berasal dari penyedia berbeda. |
| 409 | `ORDER_TOTAL_LIMIT` | Total melampaui kapasitas skema. |
| 409 | `REFERENCE_CONFLICT` | Relasi tidak valid atau data masih dipakai riwayat pesanan. |
| 409 | `DATA_CONFLICT` | Pelanggaran data unik yang dipetakan. |
| 409 | `TRANSACTION_BUSY` | Lock, deadlock, serialisasi, atau timeout query pada kasus yang dipetakan. |
| 413 | `PAYLOAD_TOO_LARGE` | Body JSON melebihi batas 20 KB. |
| 415 | `JSON_REQUIRED` | POST/PUT pada API tidak memakai Content-Type JSON. |
| 500 | `INTERNAL_SERVER_ERROR` | Kesalahan server yang belum dipetakan. |
| 503 | `DATABASE_UNAVAILABLE` | Database tidak tersedia pada kasus koneksi yang dipetakan. |

Status berhasil: `200` untuk GET/PUT, `201` untuk POST, dan `204` untuk DELETE penawaran. Pesan error dapat memuat nama field atau penawaran yang terkait. Client menggunakan `error.code` untuk mengenali jenis kegagalan, tanpa mengandalkan teks pesan yang persis sama.

Contoh penolakan lintas penyedia:

```json
{
  "success": false,
  "error": {
    "code": "MIXED_PROVIDER",
    "message": "Satu pesanan harus dari satu penyedia."
  }
}
```

Contoh penolakan Content-Type:

```json
{
  "success": false,
  "error": {
    "code": "JSON_REQUIRED",
    "message": "Gunakan Content-Type application/json."
  }
}
```

## 11. Batas penggunaan

API belum mempunyai autentikasi maupun otorisasi. `emailPembeli` memilih akun fiktif dan tidak membuktikan identitas pengguna. Gunakan API pada komputer lokal dengan data latihan.

`nilaiStokTerselamatkan` adalah nama field legacy untuk nilai normal stok yang dipesan. Pesanan baru belum membuktikan pembayaran, pengambilan, atau dampak pengurangan limbah pangan.

Filter stok minimum merupakan latihan pengembangan terpisah dalam modul. Filter tersebut tidak dicantumkan sebagai fitur yang sudah tersedia dalam kontrak dasar ini.

## 12. Acuan

Modul Pengembangan Web Chapter 7 Membangun REST API SisaBaik: Bagian 2.2–2.13, kode lengkap helper, repository, service, dan middleware, serta Langkah 18. Bukti hasil pengujian proyek dicatat dalam laporan pada folder `docs`.
