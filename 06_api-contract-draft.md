# API Contract Draft

Status dokumen: `REVIEW` v0.2, 28 September 2026. Cakupan versi ini adalah alur inti pesanan, dompet, dan panel admin. Modul percakapan, notifikasi, dan verifikasi menyusul.

Perubahan dari v0.1 mengikuti DEC-15 sampai DEC-24 di `10_notulen-sinkronisasi-erd-final.md`:
- Field `assignment_mode` dan `preferred_helper_id` diganti `booking_type` dan `target_helper_id`, sama dengan nama kolom di ERD.
- Contoh uang memisahkan upah jasa dari talangan dan memakai rumus settlement baru.
- Endpoint penyesuaian memakai bentuk jamak `/adjustments`.
- Endpoint baru: API-ORD-04E, API-ORD-09A, API-ORD-10, API-ORD-11, API-ORD-12, API-ADM-10 sampai API-ADM-13.

Base URL pengembangan `https://api.tolongindong.dev/v1`. Seluruh permintaan memakai `Content-Type: application/json`, kecuali unggah berkas yang memakai `multipart/form-data`.

## 1. Konvensi umum

Setiap permintaan terautentikasi membawa header `Authorization: Bearer <access_token>`. Setiap permintaan yang mengubah uang atau status pesanan wajib membawa header `X-Idempotency-Key` berisi UUID yang dibuat aplikasi. Server menyimpan kunci itu selama 24 jam dan mengembalikan hasil yang sama untuk kunci yang sama.

Semua respons memakai pembungkus yang seragam supaya Mobile cukup menulis satu penangan.

```json
{
  "success": true,
  "message": "Pesanan berhasil dibuat",
  "data": {},
  "meta": { "request_id": "req_9f2c1a", "server_time": "2026-09-28T09:41:00+07:00" }
}
```

Respons gagal memakai bentuk yang sama dengan `success` bernilai salah dan objek galat yang bisa dipetakan ke pesan pengguna.

```json
{
  "success": false,
  "message": "Saldo TD-Wallet kamu tidak mencukupi",
  "error": {
    "code": "WALLET_INSUFFICIENT_BALANCE",
    "detail": { "required": 68000, "available": 21500, "shortage": 46500 }
  },
  "meta": { "request_id": "req_9f2c1b", "server_time": "2026-09-28T09:41:00+07:00" }
}
```

Aturan kode status: `200` operasi berhasil, `201` sumber daya baru, `400` validasi masukan, `401` token hilang atau kedaluwarsa, `403` tidak berhak atas sumber daya atau akun disuspend, `404` tidak ditemukan, `409` konflik status, `422` aturan bisnis dilanggar, `429` permintaan berlebih, `500` kesalahan server.

Waktu selalu dikirim dalam ISO 8601 dengan zona waktu. Mobile menghitung sisa waktu dari selisih terhadap `meta.server_time`, bukan jam perangkat.

Semua nominal uang dikirim sebagai bilangan bulat rupiah, sama dengan kolom `bigint` di ERD.

Akun dengan `is_suspended` bernilai benar menerima `403 ACCOUNT_SUSPENDED` pada seluruh endpoint selain logout.

## 2. Daftar endpoint

| ID | Method dan path | Peran yang berhak | Keterangan |
| --- | --- | --- | --- |
| API-ORD-01 | `POST /orders/estimate` | Client | Menghitung rentang biaya acuan sebelum pesanan dibuat |
| API-ORD-02 | `POST /orders` | Client | Membuat pesanan broadcast atau direct booking |
| API-ORD-03 | `GET /orders/{id}` | Client dan helper pada pesanan itu | Detail pesanan, status, rincian uang, dan daftar status berikutnya yang sah |
| API-ORD-04A | `POST /orders/{id}/offers` | Helper yang disiarkan atau dituju | Mengajukan tawaran (broadcast) atau quote (direct) |
| API-ORD-04B | `GET /orders/{id}/offers` | Client pemilik pesanan | Melihat tawaran atau quote yang masuk |
| API-ORD-04C | `POST /orders/{id}/offers/{offer_id}/select` | Client pemilik pesanan | Broadcast: memilih tawaran, menunggu konfirmasi. Direct: menyetujui quote dan menahan dana |
| API-ORD-04D | `POST /orders/{id}/offers/{offer_id}/confirm` | Helper terpilih | Khusus broadcast, konfirmasi ketersediaan dan menahan dana |
| API-ORD-04E | `POST /orders/{id}/offers/decline` | Helper yang dituju | Khusus direct, menolak permintaan quote. Baru, DEC-21 |
| API-ORD-05 | `POST /orders/{id}/status` | Helper pada pesanan itu | Memajukan status pekerjaan |
| API-ORD-06 | `POST /orders/{id}/adjustments` | Helper pada pesanan itu | Mengunggah struk talangan |
| API-ORD-07 | `POST /orders/{id}/adjustments/{adjustment_id}/respond` | Client pada pesanan itu | Menyetujui atau menolak struk yang melewati batas |
| API-ORD-08 | `POST /orders/{id}/confirm` | Client pada pesanan itu | Mengonfirmasi selesai dan memicu settlement |
| API-ORD-09A | `GET /orders/{id}/cancel-preview` | Client atau helper pada pesanan itu | Rincian dana dan kompensasi sebelum membatalkan. Baru, FR-ORD-011 |
| API-ORD-09 | `POST /orders/{id}/cancel` | Client atau helper pada pesanan itu | Membatalkan dengan aturan menurut status |
| API-ORD-10 | `POST /orders/{id}/rebroadcast` | Client pemilik pesanan direct yang kedaluwarsa | Membuat pesanan broadcast baru dengan data yang sama. Baru, DEC-21 |
| API-ORD-11 | `POST /orders/{id}/attachments` | Client atau helper pada pesanan itu | Mengunggah foto bukti. Baru, DEC-16 |
| API-ORD-12 | `POST /orders/{id}/disputes` | Client atau helper pada pesanan Delivery | Mengajukan sengketa. Baru, sebelumnya belum punya endpoint |
| API-HLP-01 | `GET /helpers` | Client | Daftar helper dengan penyaringan dan pengurutan |
| API-WLT-01 | `GET /wallet` | Pemilik dompet | Saldo tersedia dan saldo tertahan |
| API-WLT-02 | `GET /wallet/transactions` | Pemilik dompet | Riwayat mutasi dengan penomoran halaman |
| API-WLT-03 | `POST /wallet/topup` | Pemilik dompet | Pengisian saldo simulasi |
| API-ADM-01 | `POST /admin/auth/login` | Admin | Login khusus admin, terpisah dari login Client dan Helper |
| API-ADM-02 | `GET /admin/verifications?status=pending` | Admin | Daftar pengajuan verifikasi yang menunggu |
| API-ADM-03 | `POST /admin/verifications/{id}/approve` | Admin | Menyetujui pengajuan verifikasi |
| API-ADM-04 | `POST /admin/verifications/{id}/reject` | Admin | Menolak pengajuan, alasan wajib |
| API-ADM-05 | `GET /admin/disputes?status=pending` | Admin | Daftar sengketa yang menunggu, hanya Delivery |
| API-ADM-06 | `POST /admin/disputes/{id}/resolve` | Admin | Memutuskan sengketa |
| API-ADM-07 | `GET /admin/commission-summary` | Admin | Ringkasan komisi bersih, dapat disaring rentang tanggal |
| API-ADM-08 | `POST /admin/users/{id}/suspend` | Admin | Menonaktifkan akun, alasan wajib |
| API-ADM-09 | `POST /admin/users/{id}/reactivate` | Admin | Mengaktifkan kembali akun |
| API-ADM-10 | `POST /admin/orders/{id}/cancel` | Admin | Membatalkan pesanan aktif, alasan wajib. Baru, DEC-22 |
| API-ADM-11 | `GET /admin/vouchers` | Admin | Daftar voucher. Baru, DEC-23 |
| API-ADM-12 | `POST /admin/vouchers` | Admin | Membuat voucher dengan validasi keras. Baru, DEC-23 |
| API-ADM-13 | `PATCH /admin/vouchers/{id}` | Admin | Mengubah atau menonaktifkan voucher. Baru, DEC-23 |

## 3. Rincian endpoint kritis

### API-ORD-02 membuat pesanan

`POST /orders`

```json
{
  "category_code": "food_run",
  "booking_type": "broadcast",
  "target_helper_id": null,
  "task_description": "Beli Nasi Padang rendang 1 porsi di RM Sederhana, sambal dipisah",
  "pickup_address_id": 12,
  "destination_address_id": 7,
  "scheduled_at": null,
  "client_estimated_price": 15000,
  "advance_limit": 50000,
  "voucher_code": "HEMAT2K"
}
```

Aturan validasi:
- `advance_limit` wajib lebih dari nol untuk kategori dengan `requires_advance`, dan wajib nol untuk kategori lain.
- `pickup_address_id` dan `destination_address_id` wajib sesuai konfigurasi kategori. Personal dan Household hanya butuh `destination_address_id` sebagai lokasi pengerjaan.
- Kalau `booking_type` bernilai `direct`, `target_helper_id` wajib diisi dan helper itu harus tersedia, tidak memegang pesanan aktif, dan punya `max_advance_limit` minimal sebesar `advance_limit`.
- Voucher hanya dicek keberadaan dan masa berlakunya di sini. Syarat minimal pesanan dicek saat tawaran dipilih (DEC-23).
- Server tidak menahan dana pada langkah ini.

Respons `201` untuk broadcast.

```json
{
  "success": true,
  "message": "Mencari helper di sekitarmu",
  "data": {
    "order_id": 2942,
    "order_number": "TD-2942",
    "booking_type": "broadcast",
    "status": "searching",
    "reference_pricing": {
      "base_fee": 10000,
      "distance_fee": 5000,
      "service_fee": 10000,
      "range": { "min": 12000, "max": 25000 }
    },
    "client_estimated_price": 15000,
    "advance_limit": 50000,
    "voucher": { "code": "HEMAT2K", "discount_value": 2000, "min_order_amount": 20000 },
    "offer_window_expires_at": "2026-09-28T09:46:00+07:00"
  }
}
```

Untuk `direct`, respons sama dengan `status` bernilai `awaiting_quote` dan `offer_window_expires_at` menandai batas 300 detik helper memberi quote.

Galat: `422 ADDRESS_OUT_OF_SERVICE_AREA`, `422 VOUCHER_NOT_FOUND_OR_EXPIRED`, `422 ADVANCE_LIMIT_REQUIRED`, `409 HELPER_NOT_AVAILABLE` (direct booking ke helper yang tidak tersedia), `422 HELPER_ADVANCE_LIMIT_TOO_LOW` (direct booking, DEC-17), `403 ACCOUNT_SUSPENDED`.

### API-ORD-04A mengajukan tawaran atau quote

`POST /orders/{id}/offers`

```json
{ "proposed_price": 20000 }
```

`proposed_price` adalah upah jasa saja, tidak pernah memuat uang belanja. Server memeriksa helper bukan pembuat pesanan, jendela 300 detik belum tertutup, helper tidak memegang pesanan aktif, dan `max_advance_limit` helper tidak kurang dari `advance_limit` pesanan.

Respons `201`.

```json
{
  "success": true,
  "message": "Tawaran terkirim, menunggu client memilih",
  "data": {
    "offer_id": 501,
    "order_id": 2942,
    "proposed_price": 20000,
    "earning_estimate": {
      "service_fee": 20000,
      "platform_commission": 2000,
      "net_service_earning": 18000,
      "advance_note": "Belanja sampai Rp 50.000 diganti penuh di luar upah"
    },
    "offer_status": "submitted",
    "valid_until": null
  }
}
```

Pada direct booking, `valid_until` berisi batas 300 detik quote berlaku.

Galat: `409 OFFER_WINDOW_CLOSED`, `403 SELF_ORDER_NOT_ALLOWED` (INV-05), `409 HELPER_HAS_ACTIVE_ORDER` (INV-06), `422 HELPER_ADVANCE_LIMIT_TOO_LOW` (DEC-17), `409 OFFER_ALREADY_SUBMITTED`.

### API-ORD-04C memilih tawaran atau menyetujui quote

`POST /orders/{id}/offers/{offer_id}/select`

Header wajib `X-Idempotency-Key`. Badan permintaan kosong. Pada langkah ini server mengecek syarat voucher terhadap `proposed_price`. Kalau tidak memenuhi syarat, pemilihan tetap berjalan tanpa voucher dan respons memuat `voucher.applied` bernilai salah beserta alasannya.

**Broadcast.** Server mengubah tawaran menjadi `selected`, status pesanan menjadi `pending_confirmation`, dan meminta konfirmasi helper dalam 60 detik. Dana belum ditahan.

```json
{
  "success": true,
  "message": "Menunggu konfirmasi dari helper",
  "data": {
    "order_id": 2942,
    "status": "pending_confirmation",
    "selected_offer_id": 501,
    "confirmation_expires_at": "2026-09-28T09:44:00+07:00",
    "pricing_preview": {
      "total_amount": 20000,
      "voucher": { "code": "HEMAT2K", "applied": true, "discount": 2000 },
      "service_client_charge": 18000,
      "advance_limit": 50000,
      "held_amount": 68000
    }
  }
}
```

**Direct booking.** Server langsung menahan dana sebesar `held_amount`, mengubah quote menjadi `confirmed`, dan status pesanan menjadi `accepted`. Respons sama dengan API-ORD-04D.

Galat: `409 OFFER_NOT_SELECTABLE` (tawaran sudah ditarik atau kedaluwarsa), `409 QUOTE_EXPIRED` (direct, lewat 300 detik), `422 WALLET_INSUFFICIENT_BALANCE` (direct, saldo tidak cukup, quote tetap berlaku sampai `valid_until`).

### API-ORD-04D konfirmasi ketersediaan (khusus broadcast)

`POST /orders/{id}/offers/{offer_id}/confirm`

Header wajib `X-Idempotency-Key`. Ini titik penahanan dana pada broadcast. Server memeriksa batas 60 detik, memastikan helper belum mengambil pesanan lain, lalu memindahkan `held_amount` dari saldo tersedia client ke saldo tertahan.

Respons `200`.

```json
{
  "success": true,
  "message": "Pesanan diterima, dana client sudah dijamin",
  "data": {
    "order_id": 2942,
    "status": "accepted",
    "client": { "name": "Budi S.", "phone_masked": "0812xxxx9012" },
    "money": {
      "total_amount": 20000,
      "voucher_discount": 2000,
      "service_client_charge": 18000,
      "advance_limit": 50000,
      "held_amount": 68000
    },
    "hold_reference": "HOLD-2942-01"
  }
}
```

Galat: `409 CONFIRMATION_EXPIRED` (status kembali `searching`), `409 HELPER_HAS_ACTIVE_ORDER` (status kembali `searching`), `422 CLIENT_INSUFFICIENT_BALANCE` (status kembali `searching`, client diberi tahu mengisi saldo).

### API-ORD-04E menolak permintaan quote (khusus direct)

`POST /orders/{id}/offers/decline`

```json
{ "reason": "Sedang di luar kota" }
```

Server mengubah baris `order_offer` helper menjadi `declined` dan status pesanan menjadi `expired`, lalu memberi tahu client dengan tombol "Siarkan ke helper lain" yang memanggil API-ORD-10. Galat: `409 ORDER_NOT_AWAITING_QUOTE`.

### API-ORD-05 memajukan status

`POST /orders/{id}/status`

```json
{ "to_status": "in_progress", "note": null }
```

Server menolak transisi yang tidak ada di state machine `04` bagian 4 dengan `409 INVALID_STATUS_TRANSITION`, beserta status saat ini dan daftar status berikutnya yang sah. Mobile memakai daftar itu untuk menentukan tombol.

Dua pemeriksaan foto:
- Transisi `arrived` ke `in_progress` pada Delivery ditolak dengan `422 PICKUP_PROOF_REQUIRED` kalau belum ada lampiran `pickup_proof` (FR-ORD-021).
- Transisi `in_progress` ke `awaiting_confirmation` pada Food run, Delivery, dan Laundry ditolak dengan `422 COMPLETION_PROOF_REQUIRED` kalau belum ada lampiran `completion_proof` (FR-ORD-013).

Foto diunggah lebih dulu lewat API-ORD-11, bukan lewat body endpoint ini.

### API-ORD-06 dan API-ORD-07 struk talangan

**Mengunggah struk.** `POST /orders/{id}/adjustments`

```json
{ "receipt_attachment_id": 881, "requested_amount": 43000 }
```

Respons `201` ketika struk masih dalam batas.

```json
{
  "success": true,
  "message": "Struk diterima",
  "data": {
    "adjustment_id": 71,
    "approval_status": "auto_approved",
    "requested_amount": 43000,
    "recognized_amount": 43000,
    "advance": { "limit": 50000, "used": 43000, "remaining": 7000 },
    "order_status": "in_progress"
  }
}
```

Ketika struk melewati sisa batas, `approval_status` bernilai `pending`, `order_status` menjadi `price_adjustment`, dan respons memuat `excess_amount` yang akan ditahan kalau client setuju.

Galat: `409 ADJUSTMENT_PENDING_EXISTS` (FR-ORD-019), `422 CATEGORY_HAS_NO_ADVANCE`, `422 RECEIPT_ATTACHMENT_INVALID`.

**Merespons struk.** `POST /orders/{id}/adjustments/{adjustment_id}/respond`

Header wajib `X-Idempotency-Key`.

```json
{ "decision": "approve" }
```

`approve` menahan dana tambahan sebesar kelebihan, `recognized_amount` sama dengan `requested_amount`. `reject` tidak menahan apa pun, `recognized_amount` diisi sisa batas yang belum terpakai. Pada kedua pilihan status pesanan kembali ke `in_progress`. Galat: `409 ADJUSTMENT_NOT_PENDING`, `422 WALLET_INSUFFICIENT_BALANCE` (approve, saldo tidak cukup untuk kelebihan).

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
      "held_amount": 68000,
      "total_amount": 20000,
      "voucher_discount": 2000,
      "service_client_charge": 18000,
      "advance_actual": 43000,
      "helper_payout": 18000,
      "helper_release": 61000,
      "platform_commission": 0,
      "client_refund": 7000
    },
    "rating_prompt": { "required": true }
  }
}
```

Pemeriksaan: `61.000 + 0 + 7.000 = 68.000` (INV-09). Komisi kotor Rp 2.000 seluruhnya dipakai menanggung voucher, sehingga komisi bersih Rp 0. Upah bersih helper tetap Rp 18.000 (INV-12).

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
    "advance_reimbursement": 0,
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

Server menghitung ulang di sisi server, tidak memakai angka dari pratinjau. Aturan lengkap di `04` bagian 5. Galat: `409 CANCELLATION_NOT_ALLOWED` (client pada `in_progress` atau setelahnya), `409 ORDER_ALREADY_FINAL`.

### API-ORD-10 siarkan ulang direct booking

`POST /orders/{id}/rebroadcast`

Hanya untuk pesanan `direct` berstatus `expired`. Server membuat pesanan baru `broadcast` dengan kategori, deskripsi, alamat, estimasi, batas talangan, dan voucher yang sama, lalu mengembalikan `order_id` baru dengan status `searching`. Galat: `409 ORDER_NOT_REBROADCASTABLE`.

### API-ORD-11 unggah foto bukti

`POST /orders/{id}/attachments` dengan `multipart/form-data`, field `attachment_type` (`pickup_proof`, `completion_proof`, `dispute_evidence`, `receipt`) dan `file`.

Respons `201`.

```json
{
  "success": true,
  "data": { "attachment_id": 881, "attachment_type": "receipt", "file_url": "https://files.tolongindong.dev/orders/2942/881.jpg" }
}
```

Galat: `413 FILE_TOO_LARGE`, `422 ATTACHMENT_TYPE_NOT_ALLOWED` (misalnya `receipt` pada kategori tanpa talangan), `403 NOT_ORDER_PARTICIPANT`.

### API-ORD-12 mengajukan sengketa

`POST /orders/{id}/disputes`

```json
{ "reason": "Kue tart hancur saat diterima", "evidence_attachment_ids": [902, 903] }
```

Server memeriksa kategori punya `is_dispute_eligible`, status `awaiting_confirmation`, dan masih dalam 24 jam sejak `helper_finished_at`. Status pesanan menjadi `disputed` dan dana tetap tertahan. Galat: `422 DISPUTE_NOT_ELIGIBLE` (kategori selain Delivery), `409 DISPUTE_WINDOW_CLOSED`, `409 DISPUTE_ALREADY_EXISTS`.

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

Galat `409 USER_HAS_ACTIVE_ORDER` kalau pengguna masih punya pesanan berstatus `pending_confirmation` sampai `awaiting_confirmation` atau `disputed`, baik sebagai client maupun helper:

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

Kalau berhasil, server menarik tawaran `submitted` milik pengguna, membatalkan pesanan client yang masih `searching` atau `awaiting_quote` sebagai `cancelled_by_admin`, mencabut seluruh token, dan mencatat `suspend_account` di `admin_action_log`. Galat lain: `422 REASON_REQUIRED`, `409 ALREADY_SUSPENDED`.

### API-ADM-10 admin membatalkan pesanan

`POST /admin/orders/{id}/cancel`

```json
{ "reason": "Pembatalan sebelum suspend akun helper" }
```

Hanya untuk status `pending_confirmation` sampai `awaiting_confirmation`. Dana jasa kembali penuh ke client, talangan sah diganti ke helper, tanpa kompensasi pembatalan, voucher kembali ke client.

```json
{
  "success": true,
  "message": "Pesanan dibatalkan admin",
  "data": {
    "order_id": 3120,
    "status": "cancelled_by_admin",
    "held_amount": 68000,
    "advance_reimbursement": 40000,
    "cancellation_fee": 0,
    "client_refund": 28000,
    "voucher_returned": true
  }
}
```

Galat: `409 ORDER_IN_DISPUTE` (pesanan `disputed`, gunakan API-ADM-06), `409 ORDER_ALREADY_FINAL`, `422 REASON_REQUIRED`.

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
  "valid_from": "2026-10-01T00:00:00+07:00",
  "valid_until": "2026-10-31T23:59:59+07:00"
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

Jangan menyimpan status pesanan sebagai sumber kebenaran di aplikasi. Tampilkan status dari server, dan pakai polling lima detik di halaman pelacakan kalau sambungan real time belum tersedia.

Buat idempotency key sekali per niat pengguna, bukan per percobaan kirim. Kunci baru hanya dibuat kalau pengguna kembali ke layar dan menekan tombol lagi dari awal.

Pesan galat yang dilihat pengguna diambil dari peta kode galat lokal, bukan dari `message` server, supaya konsisten dengan NFR-USE-02.

Tampilkan angka uang dari server apa adanya. Aplikasi tidak menghitung komisi, voucher, atau pengembalian sendiri, karena pembulatan dan batas voucher hanya dijamin benar di server.

Daftar lengkap nilai `order.status` untuk pemetaan layar ada di `05_erd-draft.md` bagian 4. Status `awaiting_quote`, `pending_confirmation`, `cancelled_by_admin`, dan `partially_refunded` baru pada versi ini dan butuh tampilan sendiri.
