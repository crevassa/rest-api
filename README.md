# Praktikum REST API Express (API Katalog Produk)

API CRUD produk dengan Express 5. Data disimpan di memori (Map) dan hilang saat server berhenti.
Versi ini sudah memuat tugas praktikum: field `stock`.

## Cara menjalankan

Prasyarat: Node.js 22 atau lebih baru, npm. Postman untuk uji manual.

```powershell
node -v
npm -v
npm install
npm test
npm start
```

- `npm test` menjalankan 11 pengujian otomatis lalu selesai.
- `npm start` membuka server di http://127.0.0.1:3000 dan terus berjalan. Hentikan dengan `Ctrl+C`.
- Cek di browser: http://127.0.0.1:3000/api/products, hasil awal `{"data":[]}`.

## Endpoint

| Metode | Path | Operasi | Sukses |
|---|---|---|---|
| GET | /api/products | Baca daftar | 200 |
| GET | /api/products/:id | Baca satu | 200 |
| POST | /api/products | Buat | 201 + header Location |
| PUT | /api/products/:id | Ganti semua field (name, price, stock) | 200 |
| PATCH | /api/products/:id | Ubah field yang dikirim | 200 |
| DELETE | /api/products/:id | Hapus | 204 |

Produk: `id` (dibuat server), `name` (1 sampai 80 karakter), `price` (bilangan bulat >= 0),
`stock` (bilangan bulat >= 0). POST dan PUT wajib name, price, stock. PATCH menerima satu atau lebih field.

## Struktur

```
src/app.js         merakit Express (tanpa membuka port)
src/server.js      membuka port 3000
src/routes.js      metode + path -> middleware -> controller
src/controller.js  menyusun status dan body respons
src/service.js     CRUD di Map, find() melempar 404
src/validation.js  validasi ID dan body
src/errors.js      ApiError, 404 rute, error handler terpusat
test/products.test.js
```

Lingkungan pengujian: Node.js v22.22.2, npm 10.9.7, Express 5.3.0 (versi terpasang oleh npm install).
