# Langkah praktikum REST API Express

Urutan kerja dari awal sampai siap dikumpulkan. Semua hasil pada tabel di bawah adalah hasil aktual
yang saya ambil dari server ini (Node.js v22.22.2). Hasilmu di Postman harus sama.

## 1. Persiapan

1. Pastikan Node.js 22+ dan npm: `node -v` dan `npm -v`. Pasang Postman dan VS Code.
2. Ekstrak zip, buka folder `rest-api` di VS Code, lalu buka terminal di folder itu
   (harus ada `package.json` di dalamnya, cek dengan `dir`).
3. Pasang dependency: `npm install`.

Folder ini sudah berisi versi akhir (dasar modul + tugas `stock`). Bagian yang ditambahkan untuk tugas
diberi komentar `[TUGAS]` di `src/validation.js` dan `test/products.test.js`.

## 2. Jalankan pengujian otomatis

```powershell
npm test
```

Hasil yang diharapkan: `# tests 11`, `# pass 11`, `# fail 0`. Modul menyebut 8 kasus untuk versi dasar.
Di sini ada 11 karena ada 3 tes tambahan untuk `stock`. Screenshot keluaran ini untuk bukti.

## 3. Jalankan server

```powershell
npm start
```

Terminal menampilkan `API aktif di http://127.0.0.1:3000/api/products` dan tetap berjalan.
Buka URL itu di browser, hasil awalnya `{"data":[]}`. Jalankan Postman di jendela lain.
Untuk mengulang dari data kosong, tekan `Ctrl+C` lalu `npm start` lagi.

## 4. Uji manual di Postman (alamat dasar http://127.0.0.1:3000)

Untuk POST, PUT, PATCH: pilih Body, raw, lalu JSON di dropdown (supaya Content-Type application/json).
Jalankan urut. Request pertama POST setelah server baru dimulai akan mendapat id 1.

| No | Metode dan path | Body JSON | Diharapkan | Hasil aktual |
|---|---|---|---|---|
| 1 | GET /api/products | - | 200, array kosong | 200 `{"data":[]}` |
| 2 | POST /api/products | `{"name":"Mouse","price":75000,"stock":10}` | 201, id baru, header Location | 201 `{"data":{"id":1,"name":"Mouse","price":75000,"stock":10}}`, Location `/api/products/1` |
| 3 | GET /api/products/1 | - | 200, produk tadi | 200 `{"data":{"id":1,"name":"Mouse","price":75000,"stock":10}}` |

Jalankan kasus negatif PUT dan PATCH di bawah ini **sebelum** DELETE, lalu GET /api/products/1 untuk memastikan
data tidak berubah:

| No | Request | Body | Diharapkan | Hasil aktual |
|---|---|---|---|---|
| N1 | POST /api/products | `{"name":" ","price":-1}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: name, price, stock |
| N2 | PUT /api/products/1 | `{"price":1000}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: name, stock |
| N3 | PATCH /api/products/1 | `{}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: Kirim minimal satu field. |
| N4 | GET /api/products/abc | - | 400 INVALID_ID | 400 INVALID_ID |
| N5 | POST /api/products | `{"name":` (JSON terpotong) | 400 INVALID_JSON | 400 INVALID_JSON |
| N6 | POST /api/products | Mouse, tipe Text (text/plain) | 415 UNSUPPORTED_MEDIA_TYPE | 415 UNSUPPORTED_MEDIA_TYPE |
| N7 | POST /api/products | `{"name":"Mouse","price":75000,"stock":-1}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: stock |
| N8 | POST /api/products | `{"name":"Mouse","price":75000}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: stock |
| N9 | PATCH /api/products/1 | `{"stock":-1}` | 400 INVALID_INPUT | 400 INVALID_INPUT, details: stock |
| N10 | GET /api/products/1 | - | 200, data tetap | 200 `{"data":{"id":1,"name":"Mouse","price":75000,"stock":10}}` |

Lanjutan alur utama:

| No | Metode dan path | Body JSON | Diharapkan | Hasil aktual |
|---|---|---|---|---|
| 4 | PUT /api/products/1 | `{"name":"Keyboard","price":120000,"stock":5}` | 200, ketiga field berubah | 200 `{"data":{"id":1,"name":"Keyboard","price":120000,"stock":5}}` |
| 5 | PATCH /api/products/1 | `{"price":95000}` | 200, harga berubah, name dan stock tetap | 200 `{"data":{"id":1,"name":"Keyboard","price":95000,"stock":5}}` |
| 5b | PATCH /api/products/1 | `{"stock":0}` | 200, stock 0 diterima | 200 `{"data":{"id":1,"name":"Keyboard","price":95000,"stock":0}}` |
| 6 | DELETE /api/products/1 | - | 204, body kosong | 204, body kosong |
| 7 | GET /api/products/1 | - | 404 PRODUCT_NOT_FOUND | 404 `{"error":{"code":"PRODUCT_NOT_FOUND","message":"Produk tidak ditemukan."}}` |

Tambahan: GET /api/unknown memberi 404 ROUTE_NOT_FOUND. PUT /api/products/999 dengan body lengkap memberi 404
PRODUCT_NOT_FOUND, tetapi tanpa `stock` memberi 400 (validasi dijalankan sebelum pencarian produk).

## 5. Screenshot yang perlu diambil

1. Keluaran `npm test` (11 pass, 0 fail).
2. Browser di `/api/products` dengan `{"data":[]}`.
3. Postman untuk tiap baris di tabel (minimal 1 sampai 7 dan N1 sampai N9): tampilkan metode, URL, body, status, dan isi respons.
4. DELETE (status 204, body kosong) dan GET setelahnya (404).
5. Tab Headers pada POST yang menunjukkan `Location`.

## 6. Yang diubah untuk tugas `stock` (agar bisa dijelaskan)

`src/validation.js`, tiga tempat:
1. `'stock'` masuk daftar field yang diizinkan (tanpa ini `stock` dianggap field tidak dikenal).
2. Blok validasi baru: `if (!partial || Object.hasOwn(body, 'stock'))` memeriksa bilangan bulat aman >= 0.
   `!partial` berarti wajib pada POST dan PUT. `Object.hasOwn` berarti pada PATCH hanya diperiksa jika dikirim.
3. `fields.stock = body.stock` agar `stock` diteruskan ke service.

`service.js`, `controller.js`, `routes.js` tidak berubah karena service meneruskan `fields` apa adanya.

`test/products.test.js`:
- `product` dan objek `fields` pada tes PUT dan PATCH ditambah `stock`.
- Ekspektasi PATCH harga memakai `{ id: 1, ...product, price: 0 }` sehingga stock ikut diperiksa.
- Tes "Input tidak valid": `error.details.length` berubah dari 3 menjadi 4 (id tak dikenal, name, price, stock hilang).
- Tiga tes baru: stock boleh 0 (POST, PUT, PATCH), stock negatif atau pecahan atau string atau hilang ditolak
  (data tidak berubah), dan PATCH harga mempertahankan stock.

## 7. Penjelasan yang diminta modul

**Perbedaan PUT dan PATCH.** `replace` (PUT) membuat objek baru dari id lama dan seluruh field dari request,
jadi name, price, dan stock wajib dikirim. `update` (PATCH) menggabungkan field yang sudah ada dengan field yang
dikirim, jadi cukup mengirim yang berubah. Contoh nyata: PATCH `{"price":95000}` pada Keyboard (stock 5)
mengubah harga saja dan stock tetap 5, sedangkan PUT `{"price":1000}` ditolak 400 karena name dan stock hilang.

**Penelusuran satu request gagal:** PATCH /api/products/1 dengan `{"stock":-1}` (kasus N9).
1. `express.json` membaca body JSON (batas 16 KB) menjadi `req.body = { stock: -1 }`.
2. Router mencocokkan PATCH `/api/products/:id`.
3. `validateId` lolos (id 1 disimpan di `res.locals.id`).
4. `validateProduct({ partial: true })`: Content-Type benar, `stock` termasuk field yang diizinkan, name dan price
   dilewati karena mode partial. Pemeriksaan stock gagal karena -1 < 0, sehingga rincian ditambahkan dan
   `ApiError(400, 'INVALID_INPUT', 'Validasi gagal.', details)` dilempar.
5. Express 5 meneruskan exception sinkron ke error handler. `controller.update` dan `service.update` tidak pernah
   dijalankan, sehingga data tidak berubah (dibuktikan GET N10).
6. `errorHandler` menerima `ApiError` apa adanya dan membalas 400 dengan
   `{"error":{"code":"INVALID_INPUT","message":"Validasi gagal.","details":["stock harus berupa bilangan bulat nonnegatif."]}}`.

## 8. Yang dikumpulkan (menurut modul)

- Kode proyek (tanpa `node_modules`) dan hasil `npm test`.
- Tabel uji manual: metode, path, input, hasil yang diharapkan, hasil aktual (tabel bagian 4, isi dengan hasil Postman milikmu).
- Penjelasan singkat PUT vs PATCH dan penelusuran satu request gagal (bagian 7).
