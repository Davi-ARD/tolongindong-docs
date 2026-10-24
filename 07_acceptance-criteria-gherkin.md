# Acceptance Criteria untuk QA

Status dokumen: `CONFIRMED` v0.3, 19 Oktober 2026. Naik ke `CONFIRMED` pada 24 Oktober 2026 bersama baseline SRS v1.0 dan SDD v1.0 yang disetujui PIC, setelah seluruh turunan SA dikonfirmasi Backend. QA menurunkan 68 test case FE dari dokumen ini (Test Plan v3.0). Versi ini disesuaikan dengan DEC-25 sampai DEC-27 (`14_perubahan-model-marketplace.md`). Ditulis dalam format Given When Then supaya bisa diturunkan langsung menjadi test case. Setiap skenario menyebut kode FR yang diuji agar keterlacakan terjaga.

Perubahan dari v0.2: skenario broadcast, konfirmasi 60 detik, dan struk talangan dihapus karena FR-nya sudah DEPRECATED. Skenario baru mencakup daftar helper, percakapan sebelum pesanan, dan alur permintaan sampai quote. Angka memakai contoh baku di `05_erd-draft.md` bagian 2.2: Household dengan quote Rp 50.000 dan voucher HEMAT2K Rp 2.000 (minimal pesanan Rp 20.000).

## 1. Registrasi dan verifikasi

```gherkin
Fitur: Registrasi akun
  Latar: Pengguna belum memiliki akun

  Skenario: Registrasi berhasil dengan data valid
    # FR-AUTH-001, FR-AUTH-002
    Diberikan pengguna berada pada layar registrasi
    Ketika pengguna mengisi nama, nomor telepon, email, dan kata sandi yang valid
    Dan menekan tombol daftar
    Dan memasukkan kode OTP yang benar sebelum 300 detik
    Maka sistem membuat akun dengan status verifikasi identitas belum terverifikasi
    Dan sistem membuat dompet dengan saldo nol untuk akun tersebut
    Dan pengguna diarahkan ke beranda

  Skenario: Registrasi ditolak karena nomor sudah terdaftar
    # FR-AUTH-003
    Diberikan nomor 081234567890 sudah terdaftar
    Ketika pengguna mendaftar dengan nomor yang sama
    Maka sistem menolak dengan pesan bahwa nomor sudah digunakan
    Dan sistem tidak mengirim kode OTP

  Skenario: Kode OTP kedaluwarsa
    # FR-AUTH-002
    Diberikan pengguna menerima kode OTP
    Ketika pengguna memasukkan kode setelah lewat 300 detik
    Maka sistem menolak kode tersebut
    Dan menyediakan tombol kirim ulang kode
```

```gherkin
Fitur: Menjadi helper
  Skenario: Akun belum terverifikasi tidak tampil dan tidak bisa dipesan
    # FR-VER-004
    Diberikan Rian sudah mendaftar sebagai helper Household tetapi status verifikasinya sedang ditinjau
    Ketika client di area yang sama membuka daftar helper Household
    Maka Rian tidak muncul di daftar
    Dan permintaan pesanan ke Rian lewat API ditolak dengan kode HELPER_NOT_AVAILABLE

  Skenario: Calon helper memilih paling banyak tiga kategori
    # FR-HLP-008
    Diberikan Rian berada di layar pendaftaran helper
    Ketika Rian memilih Delivery, Personal, dan Household
    Maka sistem menerima pilihan tersebut
    Dan layar tidak menampilkan kategori lain untuk dipilih
```

```gherkin
Fitur: Pengajuan ulang verifikasi
  Skenario: Pengajuan ulang langsung setelah ditolak
    # FR-VER-007
    Diberikan pengajuan verifikasi Rian ditolak admin dengan alasan foto KTP buram
    Ketika Rian mengunggah foto baru satu menit setelah penolakan
    Maka sistem menerima pengajuan baru dengan status pending
    Dan tidak ada masa tunggu yang harus dilewati

  Skenario: Pengajuan kedua ditolak selama pengajuan pertama masih ditinjau
    # FR-VER-007, INV-13
    Diberikan Rian memiliki satu pengajuan verifikasi berstatus pending
    Ketika Rian mencoba mengirim pengajuan lain
    Maka sistem menolak dengan pesan bahwa pengajuan sebelumnya masih ditinjau
```

## 2. Mencari helper

Baru 19 Oktober 2026, DEC-25 dan DEC-26.

```gherkin
Fitur: Daftar helper yang relevan
  Skenario: Beranda menampilkan tiga kategori
    # FR-HLP-001
    Diberikan client membuka beranda
    Maka sistem menampilkan kategori Delivery, Personal, dan Household
    Dan tidak menampilkan Food run, Moving, atau Laundry

  Skenario: Memilih kategori membuka daftar helper tersaring
    # FR-HLP-001, FR-HLP-002, FR-HLP-003
    Diberikan helper Agus menguasai Household dan Delivery
    Dan helper Dedi hanya menguasai Delivery
    Ketika client menekan kategori Household
    Maka sistem membuka daftar helper berisi Agus
    Dan Dedi tidak muncul di daftar tersebut
    Dan kartu Agus menampilkan rating, jumlah ulasan, jarak, tarif per jam, dan tag keahlian

  Skenario: Helper yang mematikan ketersediaan tidak tampil
    # FR-HLP-009
    Diberikan helper Agus mematikan sakelar ketersediaan
    Ketika client membuka daftar helper Household
    Maka Agus tidak muncul di daftar

  Skenario: Helper di luar radius tidak tampil
    # FR-HLP-005
    Diberikan helper Sari menguasai Household dan berada 12 kilometer dari lokasi aktif client
    Ketika client membuka daftar helper Household
    Maka Sari tidak muncul di daftar

  Skenario: Helper yang sibuk tetap tampil tetapi tidak bisa dipesan
    # FR-HLP-011, FR-ORD-017
    Diberikan helper Agus sedang memegang pesanan berstatus in_progress
    Ketika client membuka daftar helper Household
    Maka kartu Agus menampilkan label sedang sibuk
    Dan tombol Pesan pada halaman detail Agus tidak aktif
    Dan tombol Chat pada halaman detail Agus tetap aktif

  Skenario: Pengguna tidak melihat profil helper miliknya sendiri
    # FR-ORD-016, INV-05
    Diberikan Agus adalah helper terverifikasi yang sedang memakai aplikasi sebagai client
    Ketika Agus membuka daftar helper Household
    Maka profil helper milik Agus tidak muncul di daftar
```

## 3. Percakapan sebelum dan selama pesanan

Baru 19 Oktober 2026, DEC-27.

```gherkin
Fitur: Chat dengan helper sebelum memesan
  Skenario: Client memulai chat dari halaman detail helper
    # FR-CHT-006, FR-CHT-007
    Diberikan client Budi belum pernah menghubungi helper Agus
    Ketika Budi menekan tombol Chat pada halaman detail Agus
    Maka sistem membuat satu ruang percakapan untuk pasangan Budi dan Agus
    Dan Budi bisa mengirim pesan tanpa membuat pesanan lebih dulu
    Dan Agus menerima notifikasi pesan baru

  Skenario: Ruang percakapan yang sama dipakai lagi
    # FR-CHT-007, INV-16
    Diberikan Budi sudah punya ruang percakapan dengan Agus
    Ketika Budi menekan tombol Chat pada halaman detail Agus sekali lagi
    Maka sistem membuka ruang percakapan yang sama
    Dan tidak membuat ruang baru

  Skenario: Helper tidak bisa membuka ruang percakapan baru
    # FR-CHT-007
    Diberikan helper Agus belum pernah dihubungi client Citra
    Ketika Agus mencoba membuka ruang percakapan dengan Citra lewat API
    Maka sistem menolak dengan kode CHAT_START_CLIENT_ONLY

  Skenario: Permintaan pesanan tanpa chat membuat ruang percakapan otomatis
    # FR-CHT-007
    Diberikan Budi belum pernah chat dengan helper Dedi
    Ketika Budi mengirim permintaan pesanan ke Dedi
    Maka sistem membuat ruang percakapan Budi dan Dedi
    Dan Dedi bisa bertanya lewat ruang itu sebelum mengirim quote

  Skenario: Tombol Pesan helper ini di ruang percakapan
    # FR-CHT-008, FR-ORD-001
    Diberikan Budi sedang berada di ruang percakapan dengan Agus
    Dan Agus tersedia dan tidak memegang pesanan aktif
    Ketika Budi menekan tombol Pesan helper ini
    Maka sistem membuka form permintaan pesanan dengan Agus sudah terisi sebagai helper tujuan

  Skenario: Pemberitahuan pembayaran di luar aplikasi
    # FR-CHT-009, NFR-SEC-04
    Ketika Budi membuka ruang percakapan dengan Agus
    Maka ruang percakapan menampilkan pemberitahuan bahwa pembayaran di luar TD-Wallet tidak dilindungi
    Dan nomor telepon Agus tampil tersamar

  Skenario: Batas ruang percakapan baru per hari
    # FR-CHT-010
    Diberikan Budi sudah membuka 20 ruang percakapan baru hari ini
    Ketika Budi mencoba membuka ruang percakapan dengan helper ke-21
    Maka sistem menolak dengan kode CHAT_ROOM_LIMIT_REACHED
    Dan Budi tetap bisa mengirim pesan di ruang yang sudah ada

  Skenario: Pesan selama pesanan ditandai nomor pesanan
    # FR-ADM-005, turunan SA nomor 14
    Diberikan Budi dan Agus punya pesanan TD-2942 berstatus in_progress
    Ketika Agus mengirim pesan di ruang percakapan mereka
    Maka pesan tersebut tersimpan dengan referensi pesanan TD-2942

  Skenario: Pesan tampil cepat di perangkat lawan bicara
    # NFR-PERF-04
    Diberikan perangkat Budi dan perangkat Agus terhubung internet
    Ketika Budi mengirim sepuluh pesan berturut turut
    Maka setiap pesan tampil di perangkat Agus tidak lebih dari 5 detik setelah dikirim
```

## 4. Permintaan pesanan, quote, dan penahanan dana

Direvisi 19 Oktober 2026 mengikuti DEC-26. Menggantikan fitur "Tawar harga dan pemilihan helper pada broadcast" dan fitur "Direct booking" versi v0.2.

```gherkin
Fitur: Mengirim permintaan pesanan
  Skenario: Rentang biaya acuan tampil sebelum permintaan dikirim
    # FR-ORD-002
    Diberikan client membuka form permintaan pesanan ke Agus untuk kategori Household
    Ketika client mengisi lokasi pengerjaan
    Maka sistem menampilkan tarif dasar, biaya jarak, biaya layanan flat, dan rentang acuan
    Dan form tidak memuat field batas talangan

  Skenario: Label lokasi pengerjaan untuk Household
    # FR-ORD-001, DEC-16
    Diberikan client memilih kategori Household
    Ketika form permintaan pesanan tampil
    Maka form hanya memuat satu alamat berlabel lokasi pengerjaan

  Skenario: Daftar larangan tampil untuk kategori Personal
    # FR-ORD-022
    Diberikan client memilih kategori Personal
    Ketika form permintaan pesanan tampil
    Maka sistem menampilkan lima jenis pekerjaan yang tidak termasuk Personal
    Dan daftar itu memuat membelikan barang atau menangani uang tunai atas nama client

  Skenario: Permintaan terkirim hanya ke helper pilihan
    # FR-ORD-003D
    Diberikan client memilih Agus dari daftar helper Household
    Ketika client mengirim permintaan pesanan
    Maka status pesanan menjadi awaiting_quote
    Dan hanya Agus yang menerima notifikasi permintaan
    Dan saldo client tidak berubah

  Skenario: Permintaan ke helper dengan kategori yang tidak dikuasai ditolak
    # FR-ORD-001
    Diberikan Dedi hanya menguasai Delivery
    Ketika client mengirim permintaan Household ke Dedi lewat API
    Maka sistem menolak dengan kode HELPER_CATEGORY_MISMATCH

  Skenario: Pengguna tidak bisa memesan dirinya sendiri
    # FR-ORD-016, INV-05
    Diberikan Agus adalah helper terverifikasi
    Ketika Agus sebagai client mengirim permintaan pesanan ke profil helper miliknya lewat API
    Maka sistem menolak dengan kode SELF_ORDER_NOT_ALLOWED
```

```gherkin
Fitur: Quote dan persetujuan
  Skenario: Helper melihat upah bersih sebelum mengirim quote
    # FR-ORD-006
    Diberikan Agus menerima permintaan pesanan Household
    Ketika Agus mengisi quote Rp 50.000
    Maka aplikasi menampilkan komisi platform Rp 5.000 dan upah bersih Rp 45.000 sebelum quote dikirim

  Skenario: Client menyetujui quote dan dana ditahan
    # FR-ORD-003D, FR-WLT-002, FR-WLT-011
    Diberikan client memakai voucher HEMAT2K Rp 2.000 dengan minimal pesanan Rp 20.000
    Dan Agus mengirim quote Rp 50.000 dalam 300 detik
    Ketika client menyetujui quote dalam 300 detik sejak quote masuk
    Maka saldo tersedia client berkurang Rp 48.000
    Dan saldo tertahan client bertambah Rp 48.000
    Dan status pesanan menjadi accepted tanpa konfirmasi ulang dari Agus

  Skenario: Voucher tidak berlaku terhadap quote
    # FR-WLT-011, DEC-23
    Diberikan client memakai voucher Rp 2.000 dengan minimal pesanan Rp 20.000
    Ketika client menyetujui quote senilai Rp 18.000
    Maka persetujuan tetap berjalan tanpa voucher
    Dan client menerima pemberitahuan bahwa voucher tidak memenuhi syarat
    Dan voucher tetap tersimpan untuk pesanan lain

  Skenario: Potongan voucher tidak melebihi komisi platform
    # FR-WLT-013, INV-11
    Diberikan voucher bernilai Rp 5.000 lolos syarat minimal pesanan
    Ketika client menyetujui quote Delivery Rp 50.000
    Maka potongan voucher Rp 5.000
    Dan upah bersih helper tetap Rp 45.000

  Skenario: Helper tidak memberi quote
    # FR-ORD-003D
    Diberikan pesanan berstatus awaiting_quote
    Ketika 300 detik berlalu tanpa quote dari helper
    Maka status pesanan menjadi expired
    Dan client melihat tombol Pilih helper lain

  Skenario: Helper menolak permintaan
    # FR-ORD-003D
    Diberikan pesanan berstatus awaiting_quote tanpa quote
    Ketika Agus menolak permintaan dengan alasan sedang di luar kota
    Maka status pesanan menjadi expired
    Dan riwayat status mencatat alasan penolakan
    Dan jumlah pembatalan Agus tidak bertambah

  Skenario: Quote kedaluwarsa sebelum disetujui
    # FR-ORD-003D
    Diberikan Agus mengirim quote pada pukul 10.00.00
    Ketika client menekan setuju pada pukul 10.05.01
    Maka sistem menolak dengan kode QUOTE_EXPIRED
    Dan status pesanan menjadi expired

  Skenario: Saldo client tidak cukup saat menyetujui
    # FR-WLT-002, FR-WLT-008
    Diberikan saldo tersedia client Rp 21.500
    Dan dana yang harus ditahan Rp 48.000
    Ketika client menyetujui quote
    Maka sistem menolak dengan kode WALLET_INSUFFICIENT_BALANCE
    Dan client diberi tahu kekurangan saldo Rp 26.500
    Dan quote tetap berlaku sampai masa berlakunya habis
    Dan tidak ada baris mutasi dompet yang tercipta

  Skenario: Pilih helper lain membuat pesanan baru
    # FR-ORD-003D, turunan SA nomor 20
    Diberikan pesanan TD-2942 ke Agus berstatus expired
    Ketika client menekan Pilih helper lain dan memilih Dedi
    Maka sistem membuat pesanan baru ke Dedi dengan kategori, deskripsi, alamat, waktu, dan voucher yang sama
    Dan status pesanan baru awaiting_quote
    Dan pesanan TD-2942 tetap berstatus expired

  Skenario: Permintaan lain ke helper kedaluwarsa saat helper menerima satu pesanan
    # FR-ORD-024, INV-06
    Diberikan Agus mengirim quote ke Budi dan ke Citra
    Ketika Budi menyetujui quote lebih dulu
    Maka pesanan Budi berstatus accepted
    Dan pesanan Citra berubah menjadi expired
    Dan Citra menerima notifikasi dengan tombol Pilih helper lain

  Skenario: Persetujuan bersamaan ke helper yang sama
    # INV-06
    Diberikan Agus mengirim quote ke Budi dan ke Citra
    Ketika Budi dan Citra menyetujui pada detik yang hampir bersamaan
    Maka hanya satu pesanan yang berstatus accepted
    Dan persetujuan yang lain ditolak dengan kode HELPER_HAS_ACTIVE_ORDER tanpa dana tertahan

  Skenario: Persetujuan dikirim ulang karena jaringan buruk
    # FR-WLT-009, INV-04
    Diberikan client menekan setuju dan permintaan gagal karena waktu habis di jaringan
    Ketika aplikasi mengirim ulang dengan idempotency key yang sama
    Maka sistem mengembalikan hasil persetujuan yang pertama
    Dan tidak tercipta penahanan dana kedua
```

## 5. Penyelesaian dan settlement

```gherkin
Fitur: Menyelesaikan pesanan
  Skenario: Settlement Household dengan voucher
    # FR-ORD-010, FR-WLT-003, FR-WLT-013, INV-09, INV-12
    Diberikan pesanan Household berstatus awaiting_confirmation
    Dan quote Rp 50.000 dengan voucher Rp 2.000
    Dan dana tertahan Rp 48.000
    Ketika client menekan tombol selesaikan pesanan
    Maka helper menerima upah bersih Rp 45.000
    Dan komisi bersih platform Rp 3.000
    Dan client tidak menerima pengembalian
    Dan penjumlahan dana ke helper, komisi bersih, dan pengembalian sama dengan Rp 48.000
    Dan status pesanan menjadi completed dalam waktu tidak lebih dari 5 detik

  Skenario: Settlement tanpa voucher
    # FR-WLT-003, INV-09
    Diberikan pesanan Delivery dengan harga jasa Rp 38.000 tanpa voucher
    Ketika pesanan dikonfirmasi selesai
    Maka helper menerima Rp 34.200
    Dan komisi bersih platform Rp 3.800

  Skenario: Delivery wajib foto bukti sebelum ditandai selesai
    # FR-ORD-013
    Diberikan pesanan Delivery berstatus in_progress tanpa foto bukti penyelesaian
    Ketika helper menandai pekerjaan selesai
    Maka sistem menolak dengan kode COMPLETION_PROOF_REQUIRED

  Skenario: Household tidak wajib foto bukti
    # FR-ORD-013
    Diberikan pesanan Household berstatus in_progress tanpa foto bukti
    Ketika helper menandai pekerjaan selesai
    Maka status pesanan menjadi awaiting_confirmation

  Skenario: Client tidak merespons sampai batas waktu
    # FR-ORD-012
    Diberikan pesanan berstatus awaiting_confirmation selama 24 jam sejak helper menandai selesai
    Ketika batas waktu terlampaui
    Maka sistem mengonfirmasi pesanan secara otomatis
    Dan settlement berjalan sama seperti konfirmasi client
    Dan riwayat status mencatat pelaku sebagai sistem

  Skenario: Pelepasan dana tidak boleh berulang
    # INV-04
    Diberikan pesanan sudah berstatus completed
    Ketika permintaan konfirmasi dikirim ulang
    Maka sistem mengembalikan hasil yang sama dengan konfirmasi pertama
    Dan saldo helper tidak bertambah untuk kedua kalinya
```

## 6. Pembatalan

Direvisi 19 Oktober 2026: status `searching` dan `pending_confirmation` hilang, skenario talangan dihapus.

```gherkin
Fitur: Pembatalan pesanan
  Skenario: Client membatalkan saat menunggu quote
    # FR-ORD-011, FR-WLT-014
    Diberikan pesanan berstatus awaiting_quote dengan voucher terpasang dan quote berstatus submitted
    Ketika client membatalkan
    Maka tidak ada biaya pembatalan
    Dan voucher kembali ke client
    Dan quote berubah menjadi withdrawn

  Skenario: Client membatalkan dalam masa tenggang
    # FR-ORD-011
    Diberikan pesanan diterima helper 60 detik yang lalu
    Ketika client membatalkan
    Maka seluruh dana tertahan kembali ke client
    Dan helper tidak menerima kompensasi
    Dan status pesanan menjadi cancelled_by_client

  Skenario: Client membatalkan setelah masa tenggang
    # FR-ORD-011, FR-WLT-014
    Diberikan pesanan Household diterima 3 menit yang lalu dengan dana tertahan Rp 48.000 dan voucher Rp 2.000
    Ketika client membatalkan
    Maka helper menerima kompensasi Rp 5.000 tanpa potongan komisi
    Dan client menerima kembali Rp 43.000
    Dan voucher hangus

  Skenario: Client membatalkan setelah helper berangkat
    # FR-ORD-011
    Diberikan pesanan Delivery berstatus on_the_way dengan harga jasa Rp 40.000, voucher Rp 4.000, dan dana tertahan Rp 36.000
    Ketika client membuka pratinjau pembatalan
    Maka client melihat kompensasi helper Rp 10.000 dan pengembalian Rp 26.000
    Dan ketika client mengonfirmasi batal
    Maka helper menerima Rp 10.000 dan client menerima Rp 26.000

  Skenario: Client tidak bisa membatalkan pesanan yang sedang dikerjakan
    # FR-ORD-011
    Diberikan pesanan berstatus in_progress
    Ketika client menekan batalkan
    Maka sistem menolak dengan kode CANCELLATION_NOT_ALLOWED

  Skenario: Helper membatalkan setelah menerima
    # FR-ORD-023, FR-WLT-014
    Diberikan pesanan Household berstatus in_progress dengan dana tertahan Rp 48.000 dan voucher terpasang
    Ketika helper membatalkan
    Maka client menerima kembali Rp 48.000
    Dan voucher kembali ke client
    Dan helper tidak menerima upah
    Dan jumlah pembatalan helper bertambah satu

  Skenario: Sengketa ditolak untuk kategori selain Delivery
    # FR-ORD-018, DEC-10
    Diberikan pesanan Household berstatus awaiting_confirmation
    Ketika client mengajukan sengketa
    Maka sistem menolak dengan kode DISPUTE_NOT_ELIGIBLE
```

## 7. Foto penjemputan dan sengketa Delivery

```gherkin
Fitur: Bukti Delivery
  Skenario: Delivery wajib foto penjemputan
    # FR-ORD-021
    Diberikan pesanan Delivery berstatus arrived
    Ketika helper memulai pekerjaan tanpa foto penjemputan
    Maka sistem menolak dengan kode PICKUP_PROOF_REQUIRED

  Skenario: Sengketa Delivery dalam 24 jam
    # FR-ORD-018
    Diberikan pesanan Delivery berstatus awaiting_confirmation
    Dan helper menandai selesai 3 jam yang lalu
    Ketika client mengajukan sengketa dengan foto barang rusak
    Maka status pesanan menjadi disputed
    Dan dana tetap tertahan
```

## 8. Penilaian satu arah

Tidak berubah sejak 28 Agustus 2026 (DEC-09).

```gherkin
Fitur: Penilaian satu arah
  Skenario: Client memberi penilaian dan langsung terlihat
    # FR-RTG-001, FR-RTG-003
    Diberikan pesanan berstatus completed
    Ketika client memberi penilaian empat bintang dengan ulasan
    Maka penilaian langsung tercatat dan terlihat pada profil helper
    Dan rating rata rata helper diperbarui tanpa masa tunda

  Skenario: Helper tidak memiliki jalur memberi penilaian
    # DEC-09
    Diberikan pesanan berstatus completed
    Ketika helper membuka aplikasi
    Maka tidak ada menu atau tombol untuk menilai client

  Skenario: Pesanan batal tidak bisa dinilai
    # FR-RTG-004
    Diberikan pesanan berstatus cancelled_by_helper
    Ketika client mencoba memberi penilaian
    Maka sistem menolak permintaan tersebut
```

## 9. Panel admin

Direvisi 19 Oktober 2026 mengikuti DEC-26 dan DEC-27.

```gherkin
Fitur: Panel admin
  Skenario: Akun biasa tidak bisa akses panel admin
    # FR-ADM-002
    Diberikan akun Client atau Helper yang tidak bertanda admin
    Ketika akun tersebut mengakses endpoint admin
    Maka sistem menolak dengan kode 403

  Skenario: Admin menyetujui verifikasi
    # FR-ADM-003, FR-ADM-004, FR-ADM-008
    Diberikan pengajuan verifikasi Helper berstatus pending
    Ketika admin menyetujui pengajuan tersebut
    Maka status verifikasi Helper menjadi verified
    Dan tercatat satu baris baru di admin_action_log

  Skenario: Admin menolak verifikasi wajib mengisi alasan
    # FR-ADM-004
    Diberikan pengajuan verifikasi Helper berstatus pending
    Ketika admin menolak tanpa alasan
    Maka sistem menolak permintaan tersebut

  Skenario: Admin melihat percakapan terkait sengketa
    # FR-ADM-005
    Diberikan sengketa pada pesanan Delivery TD-3110
    Ketika admin membuka detail sengketa
    Maka admin melihat foto penjemputan, foto bukti dari client, dan pesan chat yang bertanda TD-3110

  Skenario: Sengketa diputuskan berpihak pada helper
    # FR-ADM-006
    Diberikan sengketa Delivery dengan harga jasa Rp 60.000 tanpa voucher
    Ketika admin memilih berpihak pada helper
    Maka helper menerima Rp 54.000 dan komisi bersih platform Rp 6.000
    Dan status pesanan menjadi completed

  Skenario: Sengketa diputuskan split
    # FR-ADM-006, DEC-24
    Diberikan sengketa Delivery dengan dana tertahan Rp 60.000
    Ketika admin memilih split
    Maka helper menerima Rp 30.000 dan client menerima Rp 30.000
    Dan platform tidak mengambil komisi
    Dan status pesanan menjadi partially_refunded
    Dan form keputusan tidak memuat isian persentase

  Skenario: Suspend ditolak karena pesanan aktif
    # FR-ADM-009, INV-14
    Diberikan helper memegang pesanan berstatus in_progress
    Ketika admin men-suspend helper tersebut
    Maka sistem menolak dengan kode USER_HAS_ACTIVE_ORDER beserta nomor pesanannya

  Skenario: Suspend ditolak karena pesanan sedang disengketakan
    # FR-ADM-009, DEC-22
    Diberikan client memiliki pesanan berstatus disputed
    Ketika admin men-suspend client tersebut
    Maka sistem menolak dengan kode USER_HAS_ACTIVE_ORDER

  Skenario: Suspend berhasil membatalkan permintaan yang belum berjalan
    # FR-ADM-009
    Diberikan helper tidak memiliki pesanan aktif tetapi menerima dua permintaan berstatus awaiting_quote
    Ketika admin men-suspend helper tersebut
    Maka kedua permintaan menjadi cancelled_by_admin
    Dan client kedua permintaan itu menerima notifikasi
    Dan seluruh sesi login helper dicabut
    Dan ruang percakapan helper tetap tersimpan

  Skenario: Admin membatalkan pesanan yang sedang berjalan
    # FR-ADM-011, DEC-22
    Diberikan pesanan Household berstatus in_progress dengan dana tertahan Rp 48.000 dan voucher terpasang
    Ketika admin membatalkan pesanan dengan alasan
    Maka client menerima kembali Rp 48.000
    Dan helper tidak menerima kompensasi
    Dan voucher kembali ke client
    Dan status pesanan menjadi cancelled_by_admin

  Skenario: Admin tidak bisa membatalkan pesanan yang disengketakan
    # FR-ADM-011
    Diberikan pesanan Delivery berstatus disputed
    Ketika admin mencoba membatalkan pesanan
    Maka sistem menolak dengan kode ORDER_IN_DISPUTE

  Skenario: Voucher nominal tetap ditolak kalau minimal pesanan terlalu kecil
    # FR-ADM-012, DEC-23
    Ketika admin membuat voucher Rp 5.000 dengan minimal pesanan Rp 45.000
    Maka sistem menolak dengan kode VOUCHER_EXCEEDS_COMMISSION
    Dan menampilkan minimal pesanan yang dibutuhkan Rp 50.000

  Skenario: Voucher persentase di atas 10 persen ditolak
    # FR-ADM-012, DEC-23
    Ketika admin membuat voucher persentase 15 persen
    Maka sistem menolak dengan kode VOUCHER_EXCEEDS_COMMISSION
```

## 10. Catatan pengujian tambahan untuk QA

Data uji minimum yang perlu disiapkan mencakup satu akun client bersaldo besar, satu akun client bersaldo nol, tiga akun helper terverifikasi dengan kategori berbeda (misalnya Agus menguasai Household dan Delivery, Dedi hanya Delivery, Sari hanya Personal), satu helper yang sedang memegang pesanan aktif, satu helper di luar radius 10 km, satu akun helper yang masih menunggu peninjauan, satu akun yang memegang dua peran sekaligus untuk menguji `INV-05`, dan satu pesanan pada setiap status untuk menguji transisi yang tidak sah.

Pengujian dompet sebaiknya selalu diakhiri dengan pemeriksaan rekonsiliasi, yaitu memastikan jumlah saldo tersedia ditambah saldo tertahan sama dengan hasil penjumlahan seluruh mutasi. Kalau angkanya meleset satu rupiah pun, ada cacat pada logika transaksi meskipun antarmuka terlihat normal.

Pengujian waktu seperti batas quote 300 detik, masa berlaku quote 300 detik, masa tenggang pembatalan 120 detik, dan konfirmasi otomatis 24 jam sebaiknya dilakukan dengan kemampuan memajukan waktu server di lingkungan pengujian, bukan dengan menunggu sungguhan. Ini perlu diminta ke BE sejak awal supaya tidak menjadi hambatan menjelang UAT.

Untuk percakapan, uji juga skenario dua peran: Agus memakai aplikasi sebagai client dan menghubungi helper lain, sementara Agus juga menerima chat sebagai helper. Kedua ruang harus terpisah karena pasangan client dan helper-nya berbeda.
