# Notulen Sinkronisasi Pasca ERD Final

Status dokumen: `CONFIRMED`. Dokumen ini semula agenda diskusi, lalu SA mengisi kolom **Keputusan** dari hasil diskusi. Sesi A disetujui Backend, Sesi B diputuskan tim bersama Mentor. Keputusan di sini menjadi dasar pembaruan `02`, `03`, `04`, `05`, `06`, `07`, `11`, dan class diagram di `12`, dan tercatat di decision log README sebagai DEC-14 sampai DEC-24.

| Field | Isi |
| --- | --- |
| Proyek | TolonginDong, Kelompok Spline |
| Tanggal rapat | - |
| Tanggal pencatatan ke repo | 28 September 2026 |
| Waktu dan tempat | Discord |
| Pemimpin rapat | System Analyst |
| Peserta wajib Sesi A | Backend |
| Peserta wajib Sesi B | seluruh anggota tim |
| Peserta yang dianjurkan hadir | UI/UX untuk T-06, T-09, T-11. QA untuk T-01, T-07. Mobile untuk T-02, T-09 |
| Notulis | System Analyst |
| Bahan rujukan | `erd_final.md`, `scenario.md`, jawaban 12 pertanyaan Mentor, repo `tolongindong-docs` commit `a89a487` |

## 1. Tujuan rapat

Tim sudah menjawab 12 pertanyaan terbuka ERD bersama Mentor, dan Backend sudah menyusun `erd_final.md` beserta `scenario.md`. Saat SA mencocokkan dua dokumen itu dengan repo, SA menemukan satu kesalahan rumus uang, dua versi aturan yang saling bertabrakan, dan beberapa aturan yang belum punya jawaban. Diskusi ini menyelesaikan temuan tersebut supaya SA bisa memperbarui repo dan menggambar class diagram tanpa membawa bug ke SRS dan SDD.

Diskusi dibagi dua sesi. Sesi A membahas empat topik teknis yang cukup diputuskan Backend. Sesi B membahas delapan topik produk yang butuh persetujuan Mentor dan tim.

## 2. Keputusan yang sudah final dan tidak dibuka ulang

Seluruh poin di bawah sudah diputuskan Mentor dan tercatat sebagai DEC-14.

| No | Keputusan final | Dampak yang SA terapkan di repo |
| --- | --- | --- |
| 1 | Tidak ada `ORDER_ITEM`. Satu order dianggap satu pekerjaan jasa, detail barang ditulis di deskripsi | SRS 2.5 mencatat keterbatasan: sistem tidak menyediakan rekonsiliasi barang per item |
| 2 | `PRICE_ADJUSTMENT` berkardinalitas 0..N, maksimal satu berstatus `pending` per order, semua tercatat untuk audit | FR baru FR-ORD-019 |
| 3 | Definisi kategori Food run, Delivery, Moving, Personal, Household mengikuti penjelasan Mentor. Laundry diputuskan di T-06 | Glosarium `01` diperbarui |
| 4 | `service_fee` flat sebagai komponen estimasi. Hourly rate hanya informasi profil helper. Direct booking tetap memakai quote dari helper | FR-ORD-002 dikonfirmasi. FR-ORD-003D ditulis ulang mengikuti T-09 |
| 5 | Total subsidi voucher pada satu order tidak boleh melebihi komisi platform order itu | FR baru FR-WLT-013, rumus di T-01 |
| 6 | Satu voucher per order. Voucher stacking masuk backlog | DEC-12 poin stacking digantikan. FR baru FR-WLT-012 |
| 7 | Cooldown 2 menit dihapus. Pengguna boleh langsung mengajukan ulang, asal tidak ada pengajuan lain berstatus `pending`. Anti spam ditangani rate limit API | FR-VER-006 menjadi `DEPRECATED`. FR baru FR-VER-007 |
| 8 | Foto KTP dan swafoto cukup di bucket privat dengan signed URL | Catatan "masih terbuka" pada NFR-SEC-03 dihapus |
| 9 | Batas 10 alamat menjadi business rule yang sengaja dipilih | FR-PRF-002 dikonfirmasi |
| 10 | Chat hanya tersedia setelah Client dan Helper terhubung lewat order. Chat sebelum order masuk backlog | Masuk batasan SRS |
| 11 | Kolom `phone_number` tidak dienkripsi. Akses basis data dibatasi, nomor dimasking di aplikasi | NFR-SEC-04 dipertegas |
| 12 | Admin boleh suspend akun, tetapi akun yang masih punya order aktif tidak boleh langsung disuspend | FR-ADM-009 direvisi mengikuti T-10 |

## 3. Sesi A, topik teknis (diputuskan Backend)

Kai menyetujui seluruh rekomendasi SA pada T-01 sampai T-04. Catatan tambahan dari tim pada bagian NOTE dokumen keputusan ikut mengubah T-01, dan perubahan itu tercatat di bawah.

### T-01. Rumus settlement voucher dan komisi

**Masalah.** `erd_final.md` bagian 2 poin 1 menulis `helper_payout = total_amount - platform_commission`, dengan `platform_commission` adalah komisi bersih setelah voucher. Pada contoh `scenario.md` Tahap 8 (jasa Rp50.000, voucher Rp5.000), rumus itu mencairkan Rp50.000 ke helper padahal sistem hanya menahan Rp45.000 dari client. Rumus ini melanggar INV-09 dan berisiko melanggar INV-01.

**Keputusan.** Disetujui Kai, lalu diperluas oleh catatan tim. Voucher hanya mengurangi tagihan jasa client dan komisi bersih platform. Payout helper tidak pernah tersentuh voucher. Pembulatan komisi ke bawah ke rupiah penuh.

Catatan tim menolak ide satu kolom `client_charge` yang sekaligus dipakai sebagai `held_amount`. Pada kategori bertalangan, dana yang ditahan mencakup batas talangan, sehingga tagihan jasa dan dana tertahan adalah dua angka yang berbeda. Backend wajib memisahkan empat konsep berikut dalam kolom sendiri:

| Kolom | Arti |
| --- | --- |
| `total_amount` | Harga jasa final dari quote atau tawaran yang terkonfirmasi, tanpa uang belanja |
| `service_client_charge` | Harga jasa yang dibayar client setelah voucher, `total_amount - voucher_discount` |
| `held_amount` | Dana yang sedang ditahan escrow, termasuk batas talangan dan tambahan talangan yang disetujui |
| `advance_actual` | Belanja aktual yang diakui dan diganti ke helper |

```
gross_commission      = floor(total_amount x 10%)
voucher_discount      = min(nilai_voucher, gross_commission)
service_client_charge = total_amount - voucher_discount
helper_payout         = total_amount - gross_commission            (upah bersih, tanpa talangan)
platform_commission   = gross_commission - voucher_discount        (komisi bersih, >= 0)
held_amount (awal)    = service_client_charge + advance_limit      (advance_limit = 0 untuk kategori tanpa talangan)

Saat selesai:
helper_release = helper_payout + advance_actual
client_refund  = held_amount - helper_release - platform_commission
INV-09         : helper_release + platform_commission + client_refund = held_amount
```

Contoh dari catatan tim: jasa Rp20.000, voucher Rp2.000, batas talangan Rp50.000, struk Rp43.000. `service_client_charge` Rp18.000, `held_amount` Rp68.000, helper menerima Rp18.000 + Rp43.000 = Rp61.000, komisi bersih Rp0, client menerima kembali Rp7.000. Total 61.000 + 0 + 7.000 = 68.000.

**Dampak artefak:** `05` tabel `ORDER` dan bagian rumus, contoh API-ORD-04A dan API-ORD-08 di `06`, Gherkin bagian 4 di `07`, `SettlementCalculator` di class diagram.

### T-02. Nilai enum yang belum sinkron dengan state machine

**Keputusan.** Disetujui Kai. Status `pending_confirmation` disimpan di `order.status`, tidak diturunkan dari `order_offer`. Nama nilai enum memakai `snake_case` huruf kecil sesuai konvensi Laravel.

| Kolom | Nilai final |
| --- | --- |
| `order.status` | `draft`, `searching`, `awaiting_quote`, `pending_confirmation`, `accepted`, `on_the_way`, `arrived`, `in_progress`, `price_adjustment`, `awaiting_confirmation`, `completed`, `cancelled_by_client`, `cancelled_by_helper`, `cancelled_by_admin`, `expired`, `disputed`, `refunded`, `partially_refunded` |
| `order_offer.offer_status` | `submitted`, `selected`, `confirmed`, `not_selected`, `withdrawn`, `expired`, `declined` |
| `price_adjustment.approval_status` | `pending`, `approved`, `rejected`, `auto_approved` |
| `order.booking_type` | `broadcast`, `direct` |

Nilai `partially_refunded` tidak ada di agenda. SA menambahkannya sebagai turunan T-12 karena keputusan `split` butuh status akhir sendiri, lihat bagian 6.

**Dampak artefak:** `05` bagian enum, state machine di `04`, daftar status untuk Mobile di `06`, enumerasi di class diagram.

### T-03. Tempat menyimpan foto bukti penyelesaian dan bukti sengketa

**Keputusan.** Disetujui Kai, Opsi B. Satu tabel `ORDER_ATTACHMENT` dengan kolom `order_id`, `uploaded_by_user_id`, `attachment_type` (`pickup_proof`, `completion_proof`, `dispute_evidence`, `receipt`), `file_url`, `created_at`. Struk pada `PRICE_ADJUSTMENT` menunjuk ke baris tabel ini lewat `receipt_attachment_id`, sehingga kolom `receipt_photo_url` dihapus.

**Dampak artefak:** `05` tabel baru, FR-ORD-013 dan FR-ORD-021, endpoint unggah foto API-ORD-11 di `06`.

### T-04. Catatan skema kecil

| No | Temuan | Keputusan |
| --- | --- | --- |
| a | `order` adalah reserved word di PostgreSQL | Disetujui. Nama tabel fisik `orders`, nama entitas di diagram tetap `ORDER` |
| b | Many-to-many `HELPER_PROFILE` ke `SERVICE_CATEGORY` belum punya pivot | Disetujui. Tabel `helper_service_category` dengan unique gabungan |
| c | `helper_profile.is_available` punya nilai `suspended` yang tumpang tindih dengan `user.is_suspended` | Disetujui. Nilai `suspended` dihapus dari `is_available`, suspend hanya dibaca dari `user.is_suspended` |
| d | `requires_destination_address` untuk Personal dan Household berarti lokasi pengerjaan | Disetujui. Kolom tetap, kamus data mencatat maknanya, UI/UX memakai label "Lokasi pengerjaan" |
| e | Contoh voucher `WELCOME20` melanggar batas subsidi | Disetujui. Contoh API diganti voucher Rp5.000 minimal pesanan Rp50.000 |
| f | `user.verification_status` dan `verification_request.review_status` mirip | Disetujui. Keduanya dipertahankan, `user.verification_status` diperbarui dalam transaksi yang sama saat admin memutuskan |

## 4. Sesi B, topik produk (diputuskan Mentor dan tim)

### T-05. Model uang Food run

**Keputusan.** Opsi A. Upah jasa dan uang belanja dipisah. Client menyediakan batas talangan saat membuat pesanan. Komisi platform hanya dipotong dari upah jasa, tidak pernah dari uang belanja. Helper yang `max_advance_limit`-nya di bawah batas talangan pesanan tidak boleh melihat maupun mengambil pesanan itu.

Alasan tim: kalau harga tawaran helper sekaligus dianggap mencakup harga barang, helper kena potongan platform dari uang yang hanya dipakai membeli barang milik client.

Contoh tim: upah jasa Rp20.000, belanja maksimal Rp50.000, struk Rp43.000. Helper berhak reimbursement Rp43.000, platform mengambil 10% dari upah (Rp2.000), upah bersih helper Rp18.000, total ke helper Rp61.000, sisa talangan Rp7.000 kembali ke client.

Pertanyaan ketiga (siapa merevisi `scenario.md`) tidak terjawab di diskusi. SA mengambil tugas ini, hasilnya `11_skenario-pengguna.md`.

**Dampak artefak:** FR-ORD-001, FR-ORD-014, FR-ORD-020 baru, FR-WLT-002, FR-WLT-003, `04` bagian 6, API-ORD-02, API-ORD-06, API-ORD-08, Gherkin bagian 3, `11_skenario-pengguna.md`, `EscrowService` di class diagram.

### T-06. Model Laundry dan batas kategori Personal

**Keputusan.** Laundry Opsi A: jasa jemput dan antar ke penyedia laundry. Helper mengambil pakaian, membawa ke laundry, membayar dengan talangan, mengambil kembali, lalu mengantar ke client. Helper tidak mencuci sendiri. Aturan talangan pada T-05 dan FR-ORD-014 berlaku juga untuk Laundry.

Daftar larangan Personal dari SA disetujui dengan satu tambahan dari tim: pekerjaan yang berbahaya atau berisiko tinggi terhadap keselamatan tidak termasuk Personal, walaupun bukan jasa profesional. Contoh tim: permintaan naik ke atap lantai 3 untuk memperbaiki sesuatu.

Daftar final, Personal tidak mencakup:
1. Jasa yang butuh sertifikasi atau izin, seperti listrik, medis, dan hukum.
2. Mengangkut penumpang.
3. Menangani uang tunai pihak lain di luar talangan yang tercatat.
4. Barang atau aktivitas yang melanggar hukum.
5. Pekerjaan berbahaya atau berisiko tinggi terhadap keselamatan, seperti bekerja di ketinggian.

**Dampak artefak:** konfigurasi `SERVICE_CATEGORY` di `05`, glosarium `01`, FR-ORD-014, FR-ORD-022 baru, layar pembuatan pesanan Personal untuk UI/UX.

### T-07. Aturan pembatalan final

**Keputusan.** Tabel gabungan SA disetujui. Voucher kembali kalau pembatalan gratis, hangus kalau client kena penalti. Kompensasi pembatalan masuk utuh ke helper tanpa komisi platform, karena kompensasi adalah ganti waktu helper, bukan transaksi jasa normal.

| Status saat client membatalkan | Dana client | Kompensasi helper | Voucher |
| --- | --- | --- | --- |
| `searching`, `awaiting_quote`, `pending_confirmation` | Belum ada dana tertahan | Tidak ada | Kembali |
| `accepted`, kurang dari 120 detik | Kembali penuh | Tidak ada | Kembali |
| `accepted`, 120 detik atau lebih | Kembali dikurangi Rp5.000 | Rp5.000 | Hangus |
| `on_the_way`, `arrived` | Kembali dikurangi 25% dari `total_amount` | 25% dari `total_amount` | Hangus |
| `in_progress` dan seterusnya | Client tidak bisa membatalkan sepihak | Tidak berlaku | Tidak berlaku |

Tim meminta satu kalimat di draf SA diperbaiki supaya tidak bentrok dengan T-05. Kalimat "talangan yang terbukti struk tetap dibayarkan" diganti menjadi: **jika pada Food run atau Laundry sudah terjadi pembelian, helper hanya dijamin penggantian pengeluaran yang sah dan berada dalam batas talangan yang sudah disetujui.** Tim sengaja menghindari rumusan "semua struk pasti dibayar", karena helper bisa saja belanja melewati batas tanpa persetujuan client.

**Dampak artefak:** FR-ORD-011, FR-WLT-014 baru, `04` bagian 5, API-ORD-09, Gherkin bagian 5, `CancellationPolicy` di class diagram.

### T-08. Menutup inkonsistensi DEC-10 pada Food run

**Keputusan.** Disetujui. Transisi `PRICE_ADJUSTMENT --> DISPUTED` dihapus. Kalau client menolak penyesuaian, status kembali ke `in_progress`. Helper boleh melanjutkan dengan batas talangan lama atau membatalkan pekerjaannya. Tim menambahkan aturan: pengeluaran di atas batas talangan tidak dijamin selama client belum menyetujuinya.

Contoh tim: helper minta tambahan Rp8.000, client menolak, order kembali `in_progress`, helper memilih lanjut dalam batas lama atau batal.

Tim mencatat bahwa SRS versi kasaran sudah menetapkan sengketa formal hanya untuk Delivery, sehingga penghapusan jalur ini menyelaraskan desain dengan DEC-10, bukan mengubahnya.

**Dampak artefak:** state machine `04`, catatan DEC-10 di README, FR-ORD-014, activity diagram penyesuaian talangan.

### T-09. Aturan detail direct booking

**Keputusan.** Seluruh usulan SA disetujui.

| No | Aturan final |
| --- | --- |
| a | Helper punya 300 detik untuk memberi quote. Lewat dari itu order `expired` |
| b | Konfirmasi ulang 60 detik tidak berlaku. Sistem menahan dana begitu client menyetujui quote |
| c | Quote berlaku 300 detik sejak masuk. Lewat dari itu offer `expired` dan order `expired` |
| d | Belum ada counter-offer dari client, masuk backlog |
| e | Kalau helper menolak atau quote kedaluwarsa, client melihat tombol "Siarkan ke helper lain" yang membuat pesanan broadcast baru dengan data yang sama |

Alur yang tim tulis: client memilih helper, helper punya 5 menit memberi quote, client punya 5 menit menyetujui, dana langsung ditahan, pekerjaan berjalan. Alasan menghapus konfirmasi 60 detik: pada broadcast, helper mungkin mengirim tawaran beberapa menit sebelumnya dan sudah tidak tersedia. Pada direct booking helper baru saja mengirim quote, jadi konfirmasi ulang redundan.

**Dampak artefak:** FR-ORD-003D ditulis ulang, FR-ORD-006B dibatasi ke broadcast, state machine `04`, API-ORD-04E dan API-ORD-10 baru di `06`, UC-ORD-09 baru di `03`, layar quote untuk UI/UX.

### T-10. Cakupan validasi suspend dan kewenangan admin membatalkan order

**Keputusan.** Daftar status penghalang suspend dan penanganan otomatis saat suspend berhasil disetujui. Admin boleh membatalkan order aktif, dengan dua koreksi dari tim terhadap usulan SA.

Koreksi pertama, admin cancel tidak boleh selalu mengembalikan dana penuh secara buta. Secara default, dana jasa kembali penuh ke client dan helper tidak mendapat kompensasi pembatalan. Tetapi pada Food run atau Laundry, talangan yang sah, berada dalam batas yang disetujui, dan punya bukti struk tetap diganti ke helper, baru sisanya kembali ke client. Tim memberi contoh: helper sudah belanja Rp40.000 dalam batas sah, admin membatalkan karena akun mau disuspend. Kalau client menerima refund penuh, Rp40.000 milik helper hilang dan itu bertentangan dengan T-05.

Koreksi kedua, pilihan `split` hanya dipakai pada keputusan sengketa, tidak pada admin cancel. Order Delivery yang sedang `disputed` diselesaikan lewat fitur resolve dispute, bukan admin cancel. Cancel dan resolve dispute punya fungsi berbeda.

Aturan final:
1. Status yang menghalangi suspend: `pending_confirmation`, `accepted`, `on_the_way`, `arrived`, `in_progress`, `price_adjustment`, `awaiting_confirmation`, dan `disputed`.
2. Saat suspend berhasil, sistem menarik semua tawaran helper berstatus `submitted` menjadi `withdrawn`, dan membatalkan order milik client yang masih `searching` atau `awaiting_quote`.
3. Admin boleh membatalkan order berstatus `pending_confirmation` sampai `awaiting_confirmation`, dengan alasan wajib diisi. Status akhir `cancelled_by_admin`. Order `disputed` tidak bisa dibatalkan admin.

**Dampak artefak:** FR-ADM-009 direvisi, FR-ADM-011 baru, `action_type` `cancel_order` di `ADMIN_ACTION_LOG`, galat `409 USER_HAS_ACTIVE_ORDER` di API-ADM-08, API-ADM-10 baru, Gherkin bagian 6B.

### T-11. Manajemen voucher oleh admin dan waktu evaluasi voucher

**Keputusan.** Validasi keras, bukan peringatan. Voucher nominal tetap ditolak kalau `min_order_amount` kurang dari `discount_value x 10`, contohnya voucher Rp10.000 wajib minimal pesanan Rp100.000. Voucher persentase ditolak kalau di atas 10%. Syarat voucher dicek terhadap quote atau tawaran yang benar-benar dipilih, bukan estimasi awal client, karena uang nyata transaksi berasal dari quote. Manajemen voucher oleh admin berprioritas Should, karena inti aplikasi adalah order, helper, dan escrow. Untuk demo, voucher boleh berasal dari data seed.

Alasan tim memilih validasi keras: kalau admin membuat voucher Rp20.000 dengan minimal pesanan Rp50.000, sistem hanya bisa memberi diskon Rp5.000, dan client akan merasa voucher itu menipu.

Usulan SA nomor 4 (klaim voucher lewat kode) tidak dibahas, lihat bagian 7.

**Dampak artefak:** FR-WLT-011 direvisi, FR-ADM-012 baru, API-ADM-11 sampai API-ADM-13 baru, layar panel admin untuk UI/UX, `VoucherService` di class diagram.

### T-12. Pembagian dana pada keputusan sengketa `split` dan foto penjemputan

**Keputusan.** Split 50:50 dari `held_amount`, tanpa isian persentase oleh admin. Admin cukup memilih "Split". Platform tidak mengambil komisi pada settlement split, sehingga `client_refund + helper_release = held_amount`. Kalau nominal ganjil, Backend menentukan penempatan selisih Rp1 secara deterministik. Foto penjemputan wajib untuk Delivery.

Alasan foto penjemputan: tanpa foto awal, admin hanya punya klaim client "waktu datang rusak" melawan klaim helper "dari awal memang rusak".

**Koreksi redaksi dari SA.** Draf agenda menulis foto penjemputan wajib "sebelum status berubah ke `on_the_way`". Rumusan itu keliru terhadap state machine, karena `on_the_way` berarti helper baru berangkat menuju titik jemput dan belum memegang barang. Di repo, SA menempatkan kewajiban foto pada transisi `arrived --> in_progress`, yaitu saat helper sudah di titik jemput dan menerima barang. Maksud keputusan tim tidak berubah.

**Dampak artefak:** FR-ADM-006, FR-ORD-021 baru, tabel `DISPUTE` di `05`, API-ADM-06, Gherkin bagian 6B.

## 5. Rangkuman keputusan

| Topik | Keputusan singkat | Diputuskan oleh | Nomor DEC |
| --- | --- | --- | --- |
| Bagian 2 | 12 jawaban Mentor atas pertanyaan terbuka ERD | Mentor | DEC-14 |
| T-01 | Empat konsep uang dipisah, voucher tidak menyentuh payout helper, komisi dibulatkan ke bawah | Backend, diperluas tim | DEC-15 |
| T-02 | Enum final termasuk `pending_confirmation`, `awaiting_quote`, `cancelled_by_admin`, `expired`, `declined`, `auto_approved` | Backend | DEC-16 |
| T-03 | Tabel `ORDER_ATTACHMENT` untuk semua foto | Backend | DEC-16 |
| T-04 | Tabel `orders`, pivot kategori helper, suspend hanya di `user`, label lokasi pengerjaan, contoh voucher diganti | Backend | DEC-16 |
| T-05 | Food run Opsi A, komisi hanya dari upah jasa, filter `max_advance_limit` | Mentor dan tim | DEC-17 |
| T-06 | Laundry jemput-antar bertalangan, lima larangan Personal | Mentor dan tim | DEC-18 |
| T-07 | Tabel pembatalan final, aturan voucher, kompensasi tanpa komisi | Mentor dan tim | DEC-19 |
| T-08 | Jalur sengketa Food run dihapus, penolakan kembali ke `in_progress` | Mentor dan tim | DEC-20 |
| T-09 | Direct booking dengan quote 300 detik, tanpa konfirmasi 60 detik | Mentor dan tim | DEC-21 |
| T-10 | Suspend diblokir order aktif, admin cancel dengan reimbursement talangan sah | Mentor dan tim | DEC-22 |
| T-11 | Validasi voucher keras, dicek terhadap quote terpilih, prioritas Should | Mentor dan tim | DEC-23 |
| T-12 | Split 50:50 tanpa komisi, foto penjemputan wajib untuk Delivery | Mentor dan tim | DEC-24 |

DEC-14 sekaligus menandai DEC-12 poin stacking dan cooldown sebagai digantikan, dan menjawab seluruh rekomendasi yang tercatat di DEC-13.

## 6. Turunan SA yang perlu dicek Backend

Saat menerapkan keputusan ke ERD dan API, SA menambahkan beberapa detail yang tidak dibahas eksplisit. Sebelas turunan di bawah berstatus `REVIEW` di repo sampai Kai mengonfirmasi.

| No | Turunan | Alasan |
| --- | --- | --- |
| 1 | Kolom `orders.client_estimated_price` terpisah dari `base_fee`, `distance_fee`, `service_fee` | Selama ini `orders.base_fee` menyimpan harga estimasi client dan namanya bentrok dengan `service_category.base_fee`. Tiga kolom biaya kini hanya berarti komponen rentang acuan FR-ORD-002 |
| 2 | `PRICE_ADJUSTMENT` mendapat kolom `recognized_amount` dan `additional_hold_amount` | `requested_amount` menyimpan nominal struk. `recognized_amount` menyimpan bagian yang diakui untuk reimbursement, sehingga pada struk yang ditolak, bagian dalam batas tetap diganti sesuai T-07 dan T-08. `advance_actual` adalah jumlah seluruh `recognized_amount` |
| 3 | Nilai `wallet_transaction.transaction_type` baru `advance_reimbursement` | Supaya FR-WLT-004 bisa menampilkan upah bersih dan penggantian belanja sebagai dua baris terpisah |
| 4 | Status `partially_refunded` untuk hasil `split` | `completed` dan `refunded` sama-sama tidak tepat untuk dana yang dibagi dua |
| 5 | Kolom `dispute.client_refund_amount` dan `dispute.helper_release_amount` | Mencatat hasil split, sudah ada di usulan T-12 |
| 6 | Pembatalan oleh helper dan admin cancel mengembalikan voucher ke client | Client tidak kena penalti pada dua kasus ini, jadi aturan "cancel gratis, voucher kembali" dari T-07 berlaku |
| 7 | Kompensasi pembatalan dibatasi maksimal `service_client_charge` | Mencegah biaya Rp5.000 melebihi tagihan jasa pada pesanan bernilai sangat kecil |
| 8 | Usulan selisih Rp1 pada split: `helper_release = floor(held_amount / 2)`, sisanya ke client | T-12 menyerahkan aturan ini ke Backend. Usulan ini hanya titik awal |
| 9 | Tombol "Pesan Langsung" nonaktif kalau `max_advance_limit` helper di bawah batas talangan yang client isi | Konsekuensi T-05 pada jalur direct booking |
| 10 | Kolom `service_category.requires_pickup_photo` dan `is_dispute_eligible` | Memindahkan aturan foto penjemputan (T-12) dan sengketa khusus Delivery (DEC-10) dari kode ke data, mengikuti pola kolom `requires_*` yang sudah ada |
| 11 | Kolom pelengkap: `orders.offer_window_expires_at`, `orders.helper_finished_at`, `orders.cancellation_fee`, `vouchers.is_active`, `created_at` pada `disputes` dan `verification_requests`, serta unique `(order_id, helper_id)` pada `order_offers` | Dibutuhkan untuk jendela 300 detik, hitungan 24 jam, rincian pembatalan, nonaktifkan voucher (T-11), dan urutan antrean admin |

## 7. Hal yang belum diputuskan

| No | Hal | Status |
| --- | --- | --- |
| 1 | Mekanisme pengguna mendapatkan voucher (usulan SA: klaim lewat kode di menu voucher) | Tidak dibahas. Untuk demo cukup data seed sesuai T-11 |
| 2 | Nasib voucher pada sengketa yang diputuskan `favor_client` atau `split` | Belum dibahas |
| 3 | Aturan pembulatan selisih Rp1 pada split | Diserahkan ke Backend, usulan di bagian 6 nomor 8 |

## 8. Tindak lanjut

| No | Tindakan | Penanggung jawab | Status |
| --- | --- | --- | --- |
| 1 | Memperbarui repo (`README`, `01`, `02`, `03`, `04`, `05`, `06`, `07`) sesuai keputusan | SA | Selesai 28 Sep 2026, menunggu commit |
| 2 | Menggabungkan `erd_final.md` ke `05_erd-draft.md` v0.3 dengan seluruh koreksi T-01 sampai T-04 | SA | Selesai 28 Sep 2026 |
| 3 | Menulis ulang `scenario.md` mengikuti Opsi A menjadi `11_skenario-pengguna.md` | SA | Selesai 28 Sep 2026 |
| 4 | Mengonfirmasi sebelas turunan SA di bagian 6 | Kai (BE) | Menunggu |
| 5 | Menggambar layar quote direct booking, manajemen voucher, larangan Personal, unggah foto penjemputan, dan label lokasi pengerjaan | Rafi (UI/UX) | Menunggu |
| 6 | Menyusun class diagram `12_class-diagram.md` | SA | Selesai 28 Sep 2026 |
| 7 | Menggambar use case diagram dan activity diagram versi UML untuk SRS | SA | Berikutnya |
| 8 | Menyusun SRS v0.3 dan mengajukannya untuk persetujuan Mentor | SA | Berikutnya |
