<p align="center">
  <img src="logo.png" alt="SambasKu" width="320" />
</p>

# SambasKu Images

Repositori **penyimpanan file gambar publik** untuk aplikasi **SambasKu**
(Kamus Digital Sambas-Indonesia).

## Apa isi repo ini?

Gambar di sini **bukan** konten editorial yang di-commit manual. File
diunggah otomatis oleh **backend SambasKu API** saat pengguna (admin web
atau aplikasi mobile) mengirim gambar kata atau avatar.

| Sumber submit | Cara |
| ------------- | ---- |
| Admin (web) | Unggah di form kata → `POST /api/v1/images?purpose=word` |
| Campaign notifikasi | Console campaign/template → `POST /api/v1/images?purpose=campaign` |
| Mobile (Flutter) | Sheet gambar kata → endpoint yang sama (`purpose=word`) |
| Avatar (web / mobile) | `POST /api/v1/users/me/avatar` |
| Backend API | Menerima multipart → menulis file ke repo ini lewat **GitHub Contents API** |

Repo ini harus **publik** agar client bisa memuat file lewat **jsDelivr CDN**
(`cdn.jsdelivr.net/gh/…`). Tampilan yang di-resize (lebar tetap, output webp)
dibungkus **wsrv.nl** di client. URL yang disimpan di database tetap URL
jsDelivr.

Bukti lamaran verifikator dan lampiran laporan bug **tidak** masuk ke sini.
Keduanya privat dan tetap di ImageKit.

## Struktur path

```text
assets/
├── words/
│   └── <ulid>.<ext>              # jpg | png | webp
├── campaigns/
│   └── <ulid>.<ext>              # gambar notification campaign
├── discussions/
│   └── <ulid>.<ext>
├── sponsors/
│   └── <ulid>.<ext>              # logo/avatar sponsor, id = id sponsors.json
└── avatars/
    └── <userId>/
        └── <ulid>.<ext>
```

Contoh:

```text
assets/words/01HXYZ….webp
assets/avatars/01HUSER…/01HABC….jpg
assets/sponsors/01K40MIQH8F7V9FQMSKPZMHWCN.webp
```

- Gambar kata tidak memakai slug lemma. ULID dibuat **sebelum** kata
  tersimpan, jadi path tidak bergantung pada id kata.
- Avatar dikelompokkan per `userId` (karakter selain huruf, angka, `_`,
  dan `-` dibuang).
- Nama file ULID unik dan immutable. Unggah baru = file baru. Ganti avatar
  menghapus file lama (best-effort) setelah baris user terbarui.

Format yang diterima API: **JPEG, PNG, WebP**. Ukuran maksimal **5 MB**.
Isi file dicek magic byte, bukan hanya ekstensi.

## URL publik (dipakai client)

```text
https://cdn.jsdelivr.net/gh/sambasku/images@main/<path>
```

Contoh:

```text
https://cdn.jsdelivr.net/gh/sambasku/images@main/assets/words/01HXYZ….webp
```

URL lengkap disimpan di database SambasKu (Turso / SQLite):

| Tabel / kolom | Arti |
| ------------- | ---- |
| `word_images.url` | URL jsDelivr gambar kata |
| `word_images.provider` | `github` |
| `word_images.provider_file_id` | Path relatif di repo ini |
| `word_images.sha` | Blob SHA GitHub (untuk hapus) |
| `users.avatar_url` | URL jsDelivr avatar |
| `users.avatar_provider_file_id` + `avatar_sha` | Path dan SHA avatar |

Client (web, admin, mobile) memakai helper `displayImageUrl`: URL jsDelivr
dibungkus `https://wsrv.nl/?url=…&w=…&fit=cover&output=webp`. URL lain
(misalnya baris ImageKit lama) ditampilkan apa adanya.

> Catatan: file baru bisa butuh beberapa detik sampai jsDelivr mengindeks
> commit GitHub. Path ULID immutable jadi cache CDN aman setelah itu.

## Yang tidak dilakukan di repo ini

- Tidak ada UI upload manual yang didukung sebagai alur produksi
- Tidak menyimpan gambar privat (bukti verifikator, lampiran bug)
- Jangan rename / pindah file yang sudah di-referensi DB. URL CDN akan putus

## Akses API (server saja)

Backend memakai fine-grained PAT dengan izin **Contents: Read & Write** pada
repo ini saja. Variabel lingkungan (di API / Workers):

```env
PUBLIC_IMAGE_PROVIDER=github
PUBLIC_IMAGE_GITHUB_URL=https://github.com/sambasku/images
PUBLIC_IMAGE_GITHUB_TOKEN=<pat>
```

Tanpa token atau URL, upload membalas **503** `PUBLIC_IMAGE_UPLOAD_UNAVAILABLE`.

Ringkasan env ada di `api/README.md`.
