# API Contract Draft

Status dokumen: `REVIEW` v0.3, 19 Oktober 2026. Cakupan versi ini adalah pencarian helper, percakapan, alur inti pesanan, dompet, dan panel admin. Modul autentikasi, notifikasi, dan verifikasi menyusul.

Perubahan dari v0.2 mengikuti DEC-25 sampai DEC-27 di `14_perubahan-model-marketplace.md`:
- Empat endpoint dinyatakan DEPRECATED: API-ORD-04D (konfirmasi broadcast), API-ORD-06 dan API-ORD-07 (struk talangan), API-ORD-10 (siarkan ulang). Nomornya tidak dipakai ulang.
- Enam endpoint baru: API-ORD-13 (kirim ulang ke helper lain), API-HLP-02 (detail helper), API-CHT-01 sampai API-CHT-04 (percakapan).
- API-ORD-02 tidak lagi menerima `booking_type` dan `advance_limit`. Field `target_helper_id` diganti `helper_id` dan wajib diisi.
- Contoh uang memakai contoh baku Household di `05_erd-draft.md` bagian 2.2.

Base URL pengembangan `https://api.tolongindong.dev/v1`. Seluruh permintaan memakai `Content-Type: application/json`, kecuali unggah berkas yang memakai `multipart/form-data`.

## 1. Konvensi umum

Setiap permintaan terautentikasi membawa header `Authorization: Bearer <access_token>`. Setiap permintaan yang mengubah uang atau status pesanan wajib membawa header `X-Idempotency-Key` berisi UUID yang dibuat aplikasi. Server menyimpan kunci itu selama 24 jam dan mengembalikan hasil yang sama untuk kunci yang sama.

Semua respons memakai pembungkus yang seragam supaya Mobile cukup menulis satu penangan.

```json
{
  "success": true,
  "message": "Permintaan pesanan terkirim",
  "data": {},
  "meta": { "request_id": "req_9f2c1a", "server_time": "2026-10-19T09:41:00+07:00" }
}
```

Respons gagal memakai bentuk yang sama dengan `success` bernilai salah dan objek galat yang bisa dipetakan ke pesan pengguna.

```json
{
  "success": false,
  "message": "Saldo TD-Wallet kamu tidak mencukupi",
  "error": {
    "code": "WALLET_INSUFFICIENT_BALANCE",
    "detail": { "required": 48000, "available": 21500, "shortage": 26500 }
  },
  "meta": { "request_id": "req_9f2c1b", "server_time": "2026-10-19T09:41:00+07:00" }
}
```

Aturan kode status: `200` operasi berhasil, `201` sumber daya baru, `400` validasi masukan, `401` token hilang atau kedaluwarsa, `403` tidak berhak atas sumber daya atau akun disuspend, `404` tidak ditemukan, `409` konflik status, `422` aturan bisnis dilanggar, `429` permintaan berlebih, `500` kesalahan server.

Waktu selalu dikirim dalam ISO 8601 dengan zona waktu. Mobile menghitung sisa waktu dari selisih terhadap `meta.server_time`, bukan jam perangkat.

Semua nominal uang dikirim sebagai bilangan bulat rupiah, sama dengan kolom `bigint` di ERD.

Akun dengan `is_suspended` bernilai benar menerima `403 ACCOUNT_SUSPENDED` pada seluruh endpoint selain logout.

## 2. Daftar endpoint

| ID | Method dan path | Peran yang berhak | Keterangan |
| --- | --- | --- | --- |
| API-HLP-01 | `GET /helpers` | Client | Daftar helper yang relevan, dengan penyaringan dan pengurutan. Direvisi v0.3 |
| API-HLP-02 | `GET /helpers/{helper_id}` | Client | Detail helper beserta status bisa dipesan. Baru v0.3 |
| API-CHT-01 | `GET /chats` | Client dan helper | Daftar ruang percakapan milik pengguna. Baru v0.3 |
| API-CHT-02 | `POST /chats` | Client | Membuka ruang percakapan dengan helper, atau mengambil ruang yang sudah ada. Baru v0.3 |
| API-CHT-03 | `GET /chats/{room_id}/messages` | Peserta ruang | Isi percakapan dengan penomoran halaman. Baru v0.3 |
| API-CHT-04 | `POST /chats/{room_id}/messages` | Peserta ruang | Mengirim pesan. Baru v0.3 |
| API-ORD-01 | `POST /orders/estimate` | Client | Menghitung rentang biaya acuan sebelum permintaan dikirim |
| API-ORD-02 | `POST /orders` | Client | Mengirim permintaan pesanan ke helper pilihan. Direvisi v0.3 |
| API-ORD-03 | `GET /orders/{id}` | Client dan helper pada pesanan itu | Detail pesanan, status, quote, rincian uang, dan daftar status berikutnya yang sah |
| API-ORD-04A | `POST /orders/{id}/offers` | Helper tujuan | Mengirim quote |
| API-ORD-04B | `GET /orders/{id}/offers` | Client pemilik pesanan | Melihat quote yang masuk |
| API-ORD-04C | `POST /orders/{id}/offers/{offer_id}/select` | Client pemilik pesanan | Menyetujui quote dan menahan dana |
| API-ORD-04D | ~~`POST /orders/{id}/offers/{offer_id}/confirm`~~ | | DEPRECATED v0.3, konfirmasi broadcast tidak ada lagi |
| API-ORD-04E | `POST /orders/{id}/offers/decline` | Helper tujuan | Menolak permintaan pesanan |
| API-ORD-05 | `POST /orders/{id}/status` | Helper pada pesanan itu | Memajukan status pekerjaan |
| API-ORD-06 | ~~`POST /orders/{id}/adjustments`~~ | | DEPRECATED v0.3, talangan tidak ada lagi |
| API-ORD-07 | ~~`POST /orders/{id}/adjustments/{adjustment_id}/respond`~~ | | DEPRECATED v0.3, talangan tidak ada lagi |
| API-ORD-08 | `POST /orders/{id}/confirm` | Client pada pesanan itu | Mengonfirmasi selesai dan memicu settlement |
| API-ORD-09A | `GET /orders/{id}/cancel-preview` | Client atau helper pada pesanan itu | Rincian dana dan kompensasi sebelum membatalkan |
| API-ORD-09 | `POST /orders/{id}/cancel` | Client atau helper pada pesanan itu | Membatalkan dengan aturan menurut status |
| API-ORD-10 | ~~`POST /orders/{id}/rebroadcast`~~ | | DEPRECATED v0.3, digantikan API-ORD-13 |
| API-ORD-11 | `POST /orders/{id}/attachments` | Client atau helper pada pesanan itu | Mengunggah foto bukti |
| API-ORD-12 | `POST /orders/{id}/disputes` | Client atau helper pada pesanan Delivery | Mengajukan sengketa |
| API-ORD-13 | `POST /orders/{id}/resend` | Client pemilik pesanan yang kedaluwarsa | Mengirim data pesanan yang sama ke helper lain sebagai pesanan baru. Baru v0.3 |
| API-WLT-01 | `GET /wallet` | Pemilik dompet | Saldo tersedia dan saldo tertahan |
| API-WLT-02 | `GET /wallet/transactions` | Pemilik dompet | Riwayat mutasi dengan penomoran halaman |
| API-WLT-03 | `POST /wallet/topup` | Pemilik dompet | Pengisian saldo simulasi |
| API-ADM-01 | `POST /admin/auth/login` | Admin | Login khusus admin, terpisah dari login Client dan Helper |
| API-ADM-02 | `GET /admin/verifications?status=pending` | Admin | Daftar pengajuan verifikasi yang menunggu |
| API-ADM-03 | `POST /admin/verifications/{id}/approve` | Admin | Menyetujui pengajuan verifikasi |
| API-ADM-04 | `POST /admin/verifications/{id}/reject` | Admin | Menolak pengajuan, alasan wajib |
| API-ADM-05 | `GET /admin/disputes?status=pending` | Admin | Daftar sengketa yang menunggu, hanya Delivery, beserta percakapan bertanda pesanan itu |
| API-ADM-06 | `POST /admin/disputes/{id}/resolve` | Admin | Memutuskan sengketa |
| API-ADM-07 | `GET /admin/commission-summary` | Admin | Ringkasan komisi bersih, dapat disaring rentang tanggal |
| API-ADM-08 | `POST /admin/users/{id}/suspend` | Admin | Menonaktifkan akun, alasan wajib |
| API-ADM-09 | `POST /admin/users/{id}/reactivate` | Admin | Mengaktifkan kembali akun |
| API-ADM-10 | `POST /admin/orders/{id}/cancel` | Admin | Membatalkan pesanan aktif, alasan wajib |
| API-ADM-11 | `GET /admin/vouchers` | Admin | Daftar voucher |
| API-ADM-12 | `POST /admin/vouchers` | Admin | Membuat voucher dengan validasi keras |
| API-ADM-13 | `PATCH /admin/vouchers/{id}` | Admin | Mengubah atau menonaktifkan voucher |

## 3. Rincian endpoint kritis

### API-HLP-01 daftar helper

`GET /helpers?category_code=household&lat=-6.9733&lng=107.6303&sort=distance&page=1`

| Parameter | Wajib | Keterangan |
| --- | --- | --- |
| `category_code` | Ya | `delivery`, `personal`, atau `household` |
| `lat`, `lng` | Ya | Lokasi aktif client (FR-PRF-004), titik acuan radius 10 km (FR-HLP-005) |
| `sort` | Tidak | `distance` (bawaan), `rating`, atau `rate` untuk tarif terendah (FR-HLP-004) |
| `page` | Tidak | Penomoran halaman, 20 helper per halaman |

Server hanya mengembalikan helper yang `verified`, `available`, tidak disuspend, menguasai kategori itu, dan bukan milik client yang meminta (FR-VER-004, FR-ORD-016).

Respons `200`.

```json
{
  "success": true,
  "data": {
    "total_nearby": 12,
    "helpers": [
      {
        "helper_id": 41,
        "name": "Agus P.",
        "photo_url": "https://files.tolongindong.dev/public/users/88.jpg",
        "is_verified": true,
        "rating_average": 4.8,
        "rating_count": 32,
        "distance_km": 1.4,
        "hourly_rate": 25000,
        "categories": ["household", "delivery"],
        "is_busy": false,
        "can_be_ordered": true
      }
    ]
  },
  "meta": { "page": 1, "per_page": 20 }
}
```

`is_busy` bernilai benar kalau helper memegang pesanan `accepted` sampai `awaiting_confirmation`. Pada kondisi itu `can_be_ordered` bernilai salah dan Mobile menonaktifkan tombol Pesan, tetapi tombol Chat tetap aktif (FR-HLP-011). Galat: `400 CATEGORY_REQUIRED`, `422 CATEGORY_NOT_AVAILABLE`.

### API-HLP-02 detail helper

`GET /helpers/{helper_id}`

Respons berisi field yang sama dengan satu baris API-HLP-01, ditambah `completed_order_count`, lima ulasan terbaru, dan `chat_room_id` kalau client sudah pernah membuka ruang percakapan dengan helper ini. Galat: `404 HELPER_NOT_FOUND` (termasuk helper yang belum `verified` atau disuspend), `403 SELF_HELPER_PROFILE` kalau client membuka profil helper miliknya sendiri lewat tautan langsung.

### API-CHT-02 membuka ruang percakapan

`POST /chats`

```json
{ "helper_id": 41 }
```

Server mengembalikan ruang yang sudah ada untuk pasangan client dan helper itu, atau membuat ruang baru (FR-CHT-007). Respons `201` untuk ruang baru, `200` untuk ruang yang sudah ada.

```json
{
  "success": true,
  "data": {
    "room_id": 310,
    "helper": { "helper_id": 41, "name": "Agus P.", "phone_masked": "0857xxxx3321", "can_be_ordered": true },
    "notice": "Pembayaran di luar TD-Wallet tidak dilindungi penahanan dana dan tidak bisa dibantu lewat sengketa",
    "active_order": null
  }
}
```

Galat: `403 SELF_CHAT_NOT_ALLOWED` (INV-05), `404 HELPER_NOT_FOUND`, `429 CHAT_ROOM_LIMIT_REACHED` kalau client sudah membuka 20 ruang baru hari itu (FR-CHT-010), `403 ACCOUNT_SUSPENDED`. Helper yang memanggil endpoint ini menerima `403 CHAT_START_CLIENT_ONLY`.

### API-CHT-03 dan API-CHT-04 isi dan kirim pesan

**Membaca.** `GET /chats/{room_id}/messages?before_id=9120&limit=30` mengembalikan pesan dari yang terbaru, masing masing dengan `message_id`, `sender_user_id`, `content`, `attachment_url`, `order_id`, `read_at`, dan `created_at`. Pesan yang dibaca penerima ditandai `read_at` pada permintaan yang sama.

**Mengirim.** `POST /chats/{room_id}/messages`

```json
{ "content": "Besok jam 9 bisa bersihkan kamar kos 3x4? Kira kira berapa?", "attachment_url": null }
```

Respons `201` berisi pesan yang tersimpan. Server mengisi `order_id` otomatis kalau pasangan itu sedang punya pesanan berstatus `awaiting_quote` sampai `disputed` **(turunan SA)**, lalu mengirim notifikasi `chat_message` ke penerima. Galat: `403 NOT_CHAT_PARTICIPANT`, `422 MESSAGE_EMPTY`, `403 ACCOUNT_SUSPENDED`, `409 COUNTERPART_SUSPENDED` kalau lawan bicara sedang disuspend. Untuk NFR-PERF-04, Mobile memakai polling lima detik pada layar percakapan selama sambungan real time belum tersedia.

### API-ORD-02 mengirim permintaan pesanan

`POST /orders`

```json
{
  "helper_id": 41,
  "category_code": "household",
  "task_description": "Bersihkan kamar kos 3x4 m dan cuci piring",
  "pickup_address_id": null,
  "destination_address_id": 7,
  "scheduled_at": "2026-10-20T09:00:00+07:00",
  "client_estimated_price": 50000,
  "voucher_code": "HEMAT2K"
}
```

Aturan validasi:
- `helper_id` wajib. Helper harus `verified`, `available`, tidak disuspend, menguasai `category_code`, tidak memegang pesanan aktif, dan bukan milik client sendiri.
- `pickup_address_id` dan `destination_address_id` wajib sesuai konfigurasi kategori. Personal dan Household hanya butuh `destination_address_id` sebagai lokasi pengerjaan.
- Voucher hanya dicek keberadaan dan masa berlakunya di sini. Syarat minimal pesanan dicek saat client menyetujui quote (DEC-23).
- Server tidak menahan dana pada langkah ini.
- Server membuat ruang percakapan untuk pasangan ini kalau belum ada (FR-CHT-007).

Respons `201`.

```json
{
  "success": true,
  "message": "Permintaan terkirim, menunggu quote dari helper",
  "data": {
    "order_id": 2942,
    "order_number": "TD-2942",
    "status": "awaiting_quote",
    "helper": { "helper_id": 41, "name": "Agus P." },
    "reference_pricing": {
      "base_fee": 10000,
      "distance_fee": 0,
      "service_fee": 10000,
      "range": { "min": 30000, "max": 60000 }
    },
    "client_estimated_price": 50000,
    "voucher": { "code": "HEMAT2K", "discount_value": 2000, "min_order_amount": 20000 },
    "quote_deadline_at": "2026-10-19T09:46:00+07:00",
    "chat_room_id": 310
  }
}
```

Galat: `422 ADDRESS_OUT_OF_SERVICE_AREA`, `422 VOUCHER_NOT_FOUND_OR_EXPIRED`, `409 HELPER_NOT_AVAILABLE` (helper `unavailable`, disuspend, atau belum `verified`), `409 HELPER_HAS_ACTIVE_ORDER`, `422 HELPER_CATEGORY_MISMATCH`, `403 SELF_ORDER_NOT_ALLOWED`, `403 ACCOUNT_SUSPENDED`.

### API-ORD-04A mengirim quote

`POST /orders/{id}/offers`

```json
{ "proposed_price": 50000 }
```

`proposed_price` adalah upah jasa. Server memeriksa pengirim adalah helper pesanan itu, status `awaiting_quote`, belum lewat `quote_deadline_at`, dan belum ada quote untuk pesanan ini.

Respons `201`.

```json
{
  "success": true,
  "message": "Quote terkirim, menunggu persetujuan client",
  "data": {
    "offer_id": 501,
    "order_id": 2942,
    "proposed_price": 50000,
    "earning_estimate": {
      "service_fee": 50000,
      "platform_commission": 5000,
      "net_service_earning": 45000
    },
    "offer_status": "submitted",
    "valid_until": "2026-10-19T09:49:30+07:00"
  }
}
```

Galat: `409 QUOTE_DEADLINE_PASSED`, `403 NOT_ORDER_HELPER`, `409 QUOTE_ALREADY_SUBMITTED`, `409 ORDER_NOT_AWAITING_QUOTE`, `422 PRICE_INVALID` (nol atau negatif).

### API-ORD-04C menyetujui quote

`POST /orders/{id}/offers/{offer_id}/select`

Header wajib `X-Idempotency-Key`. Badan permintaan kosong. Ini satu satunya titik penahanan dana. Server menjalankan langkah berikut dalam satu transaksi basis data:

1. Memastikan quote `submitted` dan belum lewat `valid_until`.
2. Mengunci baris helper dan memastikan helper belum memegang pesanan aktif.
3. Mengecek syarat voucher terhadap `proposed_price`. Kalau tidak memenuhi syarat, persetujuan tetap berjalan tanpa voucher dan respons memuat `voucher.applied` bernilai salah beserta alasannya.
4. Memindahkan `held_amount` dari saldo tersedia client ke saldo tertahan.
5. Mengubah quote menjadi `confirmed` dan status pesanan menjadi `accepted`.
6. Mengubah pesanan lain ke helper ini yang masih `awaiting_quote` menjadi `expired` dan memberi tahu client masing masing (FR-ORD-024).

Respons `200`.

```json
{
  "success": true,
  "message": "Pesanan diterima, dana kamu sudah ditahan",
  "data": {
    "order_id": 2942,
    "status": "accepted",
    "helper": { "name": "Agus P.", "phone_masked": "0857xxxx3321" },
    "money": {
      "total_amount": 50000,
      "voucher": { "code": "HEMAT2K", "applied": true, "discount": 2000 },
      "service_client_charge": 48000,
      "held_amount": 48000
    },
    "hold_reference": "HOLD-2942-01"
  }
}
```

Galat: `409 QUOTE_EXPIRED` (lewat `valid_until`, status pesanan menjadi `expired`), `409 OFFER_NOT_SELECTABLE` (quote sudah `withdrawn` atau `confirmed`), `422 WALLET_INSUFFICIENT_BALANCE` (quote tetap berlaku sampai `valid_until` supaya client bisa mengisi saldo), `409 HELPER_HAS_ACTIVE_ORDER` (helper baru saja menerima pesanan lain, status pesanan ini menjadi `expired`).

### API-ORD-04E menolak permintaan pesanan

`POST /orders/{id}/offers/decline`

```json
{ "reason": "Sedang di luar kota" }
```

Hanya untuk pesanan `awaiting_quote` yang belum punya quote. Server mengubah status pesanan menjadi `expired`, menyimpan alasan di `order_status_history.reason`, lalu memberi tahu client dengan tombol "Pilih helper lain" yang memanggil API-ORD-13. Tidak ada baris `order_offers` yang dibuat. Galat: `409 ORDER_NOT_AWAITING_QUOTE`, `409 QUOTE_ALREADY_SUBMITTED`, `403 NOT_ORDER_HELPER`.

### API-ORD-05 memajukan status

`POST /orders/{id}/status`

```json
{ "to_status": "in_progress", "note": null }
```

Server menolak transisi yang tidak ada di state machine `04` bagian 4 dengan `409 INVALID_STATUS_TRANSITION`, beserta status saat ini dan daftar status berikutnya yang sah. Mobile memakai daftar itu untuk menentukan tombol.

Dua pemeriksaan foto, dibaca dari konfigurasi kategori:
- Transisi `arrived` ke `in_progress` pada Delivery ditolak dengan `422 PICKUP_PROOF_REQUIRED` kalau belum ada lampiran `pickup_proof` (FR-ORD-021).
- Transisi `in_progress` ke `awaiting_confirmation` pada Delivery ditolak dengan `422 COMPLETION_PROOF_REQUIRED` kalau belum ada lampiran `completion_proof` (FR-ORD-013).

Foto diunggah lebih dulu lewat API-ORD-11, bukan lewat body endpoint ini.

### API-ORD-08 konfirmasi selesai

`POST /orders/{id}/confirm`

Header wajib `X-Idempotency-Key`. Server menjalankan settlement sesuai `05_erd-draft.md` bagian 2.2, menutup pesanan, dan membuka jendela penilaian.

```json
{
  "success": true,
  "message": "Pesanan selesai",
  "data": {
    "order_id": 2942,
    "status": "completed",
    "settlement": {
      "held_amount": 48000,
      "total_amount": 50000,
      "voucher_discount": 2000,
      "service_client_charge": 48000,
      "helper_payout": 45000,
      "helper_release": 45000,
      "platform_commission": 3000,
      "client_refund": 0
    },
    "rating_prompt": { "required": true }
  }
}
```

Pemeriksaan: `45.000 + 3.000 + 0 = 48.000` (INV-09). Komisi kotor Rp 5.000, Rp 2.000 di antaranya dipakai menanggung voucher, sehingga komisi bersih Rp 3.000. Upah bersih helper tetap Rp 45.000 dengan atau tanpa voucher (INV-12).

Galat: `409 ORDER_NOT_AWAITING_CONFIRMATION`. Permintaan ulang dengan kunci sama mengembalikan hasil yang sama (INV-04).

### API-ORD-09A dan API-ORD-09 pembatalan

**Pratinjau.** `GET /orders/{id}/cancel-preview`

```json
{
  "success": true,
  "data": {
    "order_id": 3001,
    "status": "on_the_way",
    "cancellable": true,
    "held_amount": 36000,
    "cancellation_fee": 10000,
    "client_refund": 26000,
    "voucher_returned": false,
    "rule": "on_the_way: 25 persen dari harga jasa"
  }
}
```

**Membatalkan.** `POST /orders/{id}/cancel`

Header wajib `X-Idempotency-Key`.

```json
{ "reason": "Rencana berubah" }
```

Server menghitung ulang di sisi server, tidak memakai angka dari pratinjau. Aturan lengkap di `04` bagian 5. Pembatalan pada `awaiting_quote` mengubah quote yang sudah masuk menjadi `withdrawn`. Galat: `409 CANCELLATION_NOT_ALLOWED` (client pada `in_progress` atau setelahnya, atau helper pada `awaiting_quote` yang seharusnya memakai API-ORD-04E), `409 ORDER_ALREADY_FINAL`.

### API-ORD-11 unggah foto bukti

`POST /orders/{id}/attachments` dengan `multipart/form-data`, field `attachment_type` (`pickup_proof`, `completion_proof`, `dispute_evidence`) dan `file`.

Respons `201`.

```json
{
  "success": true,
  "data": { "attachment_id": 881, "attachment_type": "pickup_proof", "file_url": "https://files.tolongindong.dev/orders/3110/881.jpg" }
}
```

Galat: `413 FILE_TOO_LARGE`, `422 ATTACHMENT_TYPE_NOT_ALLOWED` (misalnya `pickup_proof` pada kategori tanpa foto penjemputan), `403 NOT_ORDER_PARTICIPANT`.

### API-ORD-12 mengajukan sengketa

`POST /orders/{id}/disputes`

```json
{ "reason": "Kue tart hancur saat diterima", "evidence_attachment_ids": [902, 903] }
```

Server memeriksa kategori punya `is_dispute_eligible`, status `awaiting_confirmation`, dan masih dalam 24 jam sejak `helper_finished_at`. Status pesanan menjadi `disputed` dan dana tetap tertahan. Galat: `422 DISPUTE_NOT_ELIGIBLE` (kategori selain Delivery), `409 DISPUTE_WINDOW_CLOSED`, `409 DISPUTE_ALREADY_EXISTS`.

### API-ORD-13 kirim ulang ke helper lain

`POST /orders/{id}/resend`

```json
{ "helper_id": 57 }
```

Hanya untuk pesanan berstatus `expired` milik client yang meminta. Server membuat pesanan baru berstatus `awaiting_quote` untuk helper baru dengan kategori, deskripsi, alamat, waktu, estimasi, dan voucher yang sama, lalu mengembalikan `order_id` baru dengan bentuk respons sama seperti API-ORD-02. Pesanan lama tetap `expired`. Helper baru diperiksa dengan aturan yang sama seperti API-ORD-02. Galat: `409 ORDER_NOT_RESENDABLE` (status bukan `expired`), `422 SAME_HELPER_AS_BEFORE`, dan seluruh galat helper pada API-ORD-02.

### API-ADM-06 memutuskan sengketa

`POST /admin/disputes/{id}/resolve`

```json
{ "resolution": "split", "admin_note": "Kerusakan tidak bisa dipastikan terjadi di perjalanan" }
```

Nilai `resolution`: `favor_helper` menjalankan settlement normal dengan komisi dan status `completed`. `favor_client` mengembalikan seluruh `held_amount` ke client dan status `refunded`. `split` membagi `held_amount` 50:50 tanpa komisi platform dan status `partially_refunded`. Admin tidak mengirim persentase.

```json
{
  "success": true,
  "message": "Sengketa diputuskan",
  "data": {
    "dispute_id": 12,
    "order_id": 3110,
    "resolution": "split",
    "held_amount": 60000,
    "helper_release_amount": 30000,
    "client_refund_amount": 30000,
    "platform_commission": 0,
    "order_status": "partially_refunded",
    "resolved_by": { "admin_id": 3, "name": "Admin" }
  }
}
```

Galat: `422 ADMIN_NOTE_REQUIRED`, `409 DISPUTE_ALREADY_RESOLVED`, `409 DISPUTE_NOT_ELIGIBLE` (pertahanan kedua kalau kategori bukan Delivery).

### API-ADM-08 suspend akun

`POST /admin/users/{id}/suspend`

```json
{ "reason": "Laporan berkata kasar berulang" }
```

Galat `409 USER_HAS_ACTIVE_ORDER` kalau pengguna masih punya pesanan berstatus `accepted` sampai `awaiting_confirmation` atau `disputed`, baik sebagai client maupun helper:

```json
{
  "success": false,
  "message": "Pengguna masih memiliki pesanan yang sedang berjalan",
  "error": {
    "code": "USER_HAS_ACTIVE_ORDER",
    "detail": { "active_orders": [ { "order_number": "TD-3120", "status": "in_progress", "role": "helper" } ] }
  }
}
```

Kalau berhasil, server membatalkan seluruh pesanan `awaiting_quote` yang dibuat pengguna sebagai client maupun yang ditujukan kepadanya sebagai helper menjadi `cancelled_by_admin`, mengubah quote terkait menjadi `withdrawn`, mencabut seluruh token, dan mencatat `suspend_account` di `admin_action_log`. Ruang percakapan pengguna tidak dihapus. Galat lain: `422 REASON_REQUIRED`, `409 ALREADY_SUSPENDED`.

### API-ADM-10 admin membatalkan pesanan

`POST /admin/orders/{id}/cancel`

```json
{ "reason": "Pembatalan sebelum suspend akun helper" }
```

Hanya untuk status `accepted` sampai `awaiting_confirmation`. Dana jasa kembali penuh ke client, tanpa kompensasi pembatalan, voucher kembali ke client.

```json
{
  "success": true,
  "message": "Pesanan dibatalkan admin",
  "data": {
    "order_id": 3120,
    "status": "cancelled_by_admin",
    "held_amount": 48000,
    "cancellation_fee": 0,
    "client_refund": 48000,
    "voucher_returned": true
  }
}
```

Galat: `409 ORDER_IN_DISPUTE` (pesanan `disputed`, gunakan API-ADM-06), `409 ORDER_ALREADY_FINAL`, `409 ORDER_NOT_CANCELLABLE_BY_ADMIN` (pesanan `awaiting_quote`, batal lewat suspend), `422 REASON_REQUIRED`.

### API-ADM-12 membuat voucher

`POST /admin/vouchers`

```json
{
  "code": "HEMAT5K",
  "discount_type": "fixed",
  "discount_value": 5000,
  "max_discount_amount": null,
  "min_order_amount": 50000,
  "usage_limit": 30,
  "category_id": null,
  "valid_from": "2026-11-01T00:00:00+07:00",
  "valid_until": "2026-11-30T23:59:59+07:00"
}
```

Validasi keras (DEC-23): `fixed` ditolak kalau `min_order_amount < discount_value x 10`, `percentage` ditolak kalau `discount_value > 10`.

```json
{
  "success": false,
  "message": "Minimal pesanan terlalu kecil untuk diskon ini",
  "error": {
    "code": "VOUCHER_EXCEEDS_COMMISSION",
    "detail": { "discount_value": 5000, "min_order_amount": 45000, "required_min_order_amount": 50000 }
  }
}
```

Galat lain: `409 VOUCHER_CODE_EXISTS`, `422 INVALID_DATE_RANGE`. API-ADM-13 memakai validasi yang sama dan menerima `{ "is_active": false }` untuk menonaktifkan.

## 4. Catatan untuk Mobile Developer

Jangan menyimpan status pesanan sebagai sumber kebenaran di aplikasi. Tampilkan status dari server, dan pakai polling lima detik di halaman pelacakan dan halaman percakapan kalau sambungan real time belum tersedia.

Status `awaiting_quote` punya dua layar. Kalau API-ORD-03 belum memuat quote, tampilkan "Menunggu quote dari helper" dengan hitung mundur ke `quote_deadline_at`. Kalau sudah memuat quote `submitted`, tampilkan rincian quote dan tombol "Setuju dan Bayar" dengan hitung mundur ke `valid_until`.

Buat idempotency key sekali per niat pengguna, bukan per percobaan kirim. Kunci baru hanya dibuat kalau pengguna kembali ke layar dan menekan tombol lagi dari awal.

Pesan galat yang dilihat pengguna diambil dari peta kode galat lokal, bukan dari `message` server, supaya konsisten dengan NFR-USE-02.

Tampilkan angka uang dari server apa adanya. Aplikasi tidak menghitung komisi, voucher, atau pengembalian sendiri, karena pembulatan dan batas voucher hanya dijamin benar di server.

Daftar lengkap nilai `order.status` untuk pemetaan layar ada di `05_erd-draft.md` bagian 4. Status `searching`, `pending_confirmation`, dan `price_adjustment` sudah dihapus dan tidak perlu dipetakan.
