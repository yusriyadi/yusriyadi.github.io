# LogViewer V3

LogViewer V3 adalah web app satu file untuk tim support NUTAPOS. Viewer membuka arsip network log Android, mendekripsi entri, dan menampilkannya sebagai daftar request HTTP serta transfer DigitalOcean Spaces. Semua pemrosesan berjalan lokal di browser.

## Membuka arsip

1. Buka [network-log-viewer-v3.html](network-log-viewer-v3.html) di browser yang mendukung Web Crypto dan dekompresi ZIP serta gzip.
2. Klik **Upload ZIP** dan pilih arsip dengan nama `NetworkLog-<outletId>-<deviceNo>.zip` atau `NetworkLog-<outletId>-<deviceNo>-<suffix>.zip`.
3. Pilih tanggal. Viewer baru mulai mendekripsi saat tanggal dipilih, lalu menggabungkan semua file `.enc` di folder tanggal itu dan mengurutkan entri berdasarkan waktu mulai request.
4. Klik kartu log untuk membuka endpoint lengkap, headers, params, response data, atau exception yang tersedia.

Arsip bisa memuat beberapa folder tanggal dengan pola `YYYY-MM-DD/*.enc`. Viewer membaca file biner **NLOG v1 dengan gzip** di dalam ZIP. File `.log` lama dilewati. Viewer mencoba kunci dari nama arsip dengan pola `<outletId>.<deviceNo>`, lalu mencoba kunci default aplikasi bila perlu. Ringkasan di bawah daftar menunjukkan jumlah file, penggunaan kunci default, dan kegagalan baca jika ada.

## Filter, detail, dan ekspor

- **Pencarian biasa** mencari potongan teks di seluruh entri asli, termasuk UUID, endpoint, headers, params, dan response. Huruf besar dan kecil diabaikan.
- **Dropdown Endpoint, Status, dan Method** membatasi hasil pada tanggal yang dipilih. Status bisa berupa kode HTTP atau hasil transfer SDK seperti `COMPLETED`, `FAILED`, dan `CANCELED`. Pilihan **Semua method** juga mencakup transfer Spaces yang tidak memiliki metode HTTP.
- **Paginasi** menampilkan 50 entri per halaman. Perubahan filter atau pencarian mengembalikan tampilan ke halaman pertama.
- **Salin** tersedia untuk Params / body dan Response data. **Salin entri JSON** menyalin satu entri lengkap dengan indentasi.
- **Unduh JSON** mengekspor hasil filter sebagai array JSON berindentasi. Jika params atau response berisi objek atau array JSON yang valid, body dimasukkan sebagai struktur JSON.
- **Unduh JSONL** mengekspor hasil filter sebagai entri asli, satu per baris. Kedua unduhan mencakup hasil filter di seluruh halaman. Pemformatan ekspor mempertahankan digit angka besar dari teks asli.

## Advanced Search

Klik **Advanced Search** di bawah pencarian biasa. Masukkan query lalu tekan **Enter** atau klik **Jalankan**. **Pakai contoh** mengisi query awal dan **Cheat Sheet** membuka referensi di viewer. **Reset** melepas query. Query aktif digabung dengan pencarian biasa dan dropdown, dan berlaku untuk tanggal yang sedang dipilih.

Tulis satu kondisi sebagai `field operator nilai`. Gabungkan kondisi dengan `and` ketika semuanya harus cocok.

| Operator | Kegunaan |
| --- | --- |
| `=` | Nilai sama persis |
| `!=` | Field tersedia dan nilainya berbeda |
| `>`, `>=`, `<`, `<=` | Perbandingan angka |
| `contains` | Field teks memuat potongan teks tertentu |

- Teks memakai tanda kutip ganda. Nilai `"29"` adalah string, sedangkan `29` adalah angka. Nama field dan pencocokan teks peka huruf besar dan kecil. Jika tanda kutip adalah bagian dari teks yang dicari, tulis `\"`.
- Angka mendukung integer, desimal, dan notasi eksponen. Pencocokan integer di luar rentang aman JavaScript membutuhkan browser yang menyediakan token angka asli.
- Boolean dan null ditulis sebagai `true`, `false`, atau `null` tanpa kutip. Field yang hilang tidak cocok dengan kondisi apa pun, termasuk `!=`.
- Gunakan titik untuk field bertingkat dan indeks mulai dari `[0]` untuk memilih satu item array. Wildcard `[*]` cocok jika minimal satu item memenuhi kondisi. Dua kondisi wildcard yang digabung dengan `and` bisa cocok dengan item yang berbeda.
- Gunakan `params` untuk body request dan `data` untuk body response. Jika body memiliki objek bernama `data`, tambahkan segmen `.data`. Body berupa teks atau JSON terpotong bisa dicari dengan `contains` pada field body itu sendiri.

### Contoh query

**Request POST dengan HTTP 400 ke atas**

```text
method = "POST" and responseCode >= 400
```

**Request ke endpoint options yang lebih dari satu detik**

```text
endpoint contains "/app/options" and responseTimeMs > 1000
```

**Transaksi dengan ID 29 di body request**

```text
params.data.TransactionID = "29"
```

**Transaksi dengan ID 29 di objek data dalam response**

```text
data.data.TransactionID = "29"
```

**Item mana pun di Details_Item dengan ID tepat 29**

```text
params.data.Details_Item[*].TransactionID = "29"
```

**Header device dan field pada item pertama response**

```text
headers.x-device-no[0] = "13" and data.data[0].OutletID = "47136"
```

**Upload arsip NetworkLog yang gagal**

```text
responseCode = "FAILED" and params.objectKey contains "NetworkLog"
```

**Response yang ditandai terpotong**

```text
data contains "[TRUNCATED"
```
