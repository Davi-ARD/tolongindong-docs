# Acceptance Criteria untuk QA

Status dokumen: `REVIEW` v0.2, 28 September 2026, disesuaikan dengan DEC-14 sampai DEC-24. Ditulis dalam format Given When Then supaya bisa diturunkan langsung menjadi test case. Setiap skenario menyebut kode FR yang diuji agar keterlacakan terjaga.

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
  Skenario: Akun belum terverifikasi tidak bisa menerima pesanan
    # FR-VER-004
    Diberikan pengguna sudah mendaftar sebagai helper tetapi status verifikasinya sedang ditinjau
    Ketika sistem mencari helper untuk sebuah pesanan baru di area yang sama
    Maka pengguna tersebut tidak menerima tawaran pesanan
```

```gherkin
Fitur: Pengajuan ulang verifikasi
  # Baru 28 September 2026, DEC-14 nomor 7. Menggantikan skenario cooldown 2 menit

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

## 2. Pembuatan pesanan, tawar harga, dan penahanan dana

Direvisi 28 September 2026. Angka memakai contoh baku di `05_erd-draft.md` bagian 2.2: upah jasa Rp 20.000, voucher Rp 2.000, batas talangan Rp 50.000.

```gherkin
Fitur: Membuat pesanan
  Skenario: Rentang biaya acuan tampil sebelum pesanan dikirim
    # FR-ORD-002
    Diberikan client memilih kategori Food run dan mengisi alamat
    Ketika client membuka ringkasan pesanan
    Maka sistem menampilkan tarif dasar, biaya jarak, biaya layanan flat, dan rentang acuan
    Dan menampilkan field harga estimasi upah jasa dan field batas talangan

  Skenario: Batas talangan wajib untuk kategori bertalangan
    # FR-ORD-001, DEC-17
    Diberikan client memilih kategori Laundry
    Ketika client mengirim pesanan tanpa batas talangan
    Maka sistem menolak dengan kode ADVANCE_LIMIT_REQUIRED

  Skenario: Label lokasi pengerjaan untuk Household
    # FR-ORD-001, DEC-16
    Diberikan client memilih kategori Household
    Ketika form pembuatan pesanan tampil
    Maka form hanya memuat satu alamat berlabel lokasi pengerjaan

  Skenario: Daftar larangan tampil untuk kategori Personal
    # FR-ORD-022
    Diberikan client memilih kategori Personal
    Ketika form pembuatan pesanan tampil
    Maka sistem menampilkan lima jenis pekerjaan yang tidak termasuk Personal
    Dan daftar itu memuat pekerjaan berbahaya atau berisiko tinggi terhadap keselamatan

  Skenario: Tidak ada tawaran sampai jendela berakhir
    # FR-ORD-005
    Diberikan pesanan broadcast berstatus searching
    Ketika 300 detik berlalu tanpa ada tawaran
    Maka status pesanan menjadi expired
    Dan saldo client tidak pernah berubah sejak pesanan dibuat
    Dan client menerima notifikasi dengan saran menaikkan harga estimasi
```

```gherkin
Fitur: Tawar harga dan pemilihan helper pada broadcast
  Skenario: Beberapa helper mengajukan tawaran berbeda
    # FR-ORD-003, FR-ORD-003B, FR-ORD-006
    Diberikan pesanan Food run berstatus searching dengan estimasi upah Rp 15.000 dan batas talangan Rp 50.000
    Ketika helper A mengajukan upah Rp 23.000
    Dan helper B mengajukan upah Rp 20.000
    Maka client melihat dua tawaran dengan nominal dan profil masing masing
    Dan helper B melihat upah bersih Rp 18.000 sebelum mengirim tawaran
    Dan belum ada dana yang tertahan

  Skenario: Helper dengan batas talangan terlalu kecil tidak melihat pesanan
    # FR-ORD-020, DEC-17
    Diberikan helper C memiliki batas talangan maksimal Rp 30.000
    Dan pesanan Food run memiliki batas talangan Rp 50.000
    Ketika sistem menyiarkan pesanan
    Maka helper C tidak menerima notifikasi pesanan tersebut
    Dan tawaran dari helper C lewat API ditolak dengan kode HELPER_ADVANCE_LIMIT_TOO_LOW

  Skenario: Konfirmasi helper menahan jasa setelah voucher ditambah batas talangan
    # FR-ORD-003C, FR-ORD-006B, FR-WLT-002, FR-WLT-011
    Diberikan client memakai voucher Rp 2.000 dengan minimal pesanan Rp 20.000
    Dan client memilih tawaran helper B senilai Rp 20.000
    Ketika helper B mengonfirmasi ketersediaan dalam 60 detik
    Maka saldo tersedia client berkurang Rp 68.000
    Dan saldo tertahan client bertambah Rp 68.000
    Dan status pesanan menjadi accepted

  Skenario: Voucher tidak berlaku terhadap tawaran yang dipilih
    # FR-WLT-011, DEC-23
    Diberikan client memakai voucher Rp 2.000 dengan minimal pesanan Rp 20.000
    Ketika client memilih tawaran senilai Rp 18.000
    Maka pemilihan tetap berjalan tanpa voucher
    Dan client menerima pemberitahuan bahwa voucher tidak memenuhi syarat
    Dan voucher tetap tersimpan untuk pesanan lain

  Skenario: Potongan voucher tidak melebihi komisi platform
    # FR-WLT-013, INV-11
    Diberikan voucher bernilai Rp 5.000 lolos syarat minimal pesanan
    Ketika harga jasa tawaran terpilih Rp 50.000
    Maka potongan voucher Rp 5.000
    Dan upah bersih helper tetap Rp 45.000

  Skenario: Helper terpilih tidak merespons konfirmasi
    # FR-ORD-006B
    Diberikan client memilih tawaran helper B
    Ketika 60 detik berlalu tanpa konfirmasi dari helper B
    Maka status pesanan kembali menjadi searching
    Dan status tawaran helper B menjadi expired
    Dan tidak ada dana yang tertahan

  Skenario: Saldo client tidak cukup saat helper mengonfirmasi
    # FR-WLT-002, FR-WLT-008
    Diberikan saldo tersedia client Rp 21.500
    Dan dana yang harus ditahan Rp 68.000
    Ketika helper B mengonfirmasi ketersediaan
    Maka sistem menolak dengan kode CLIENT_INSUFFICIENT_BALANCE
    Dan status pesanan kembali menjadi searching
    Dan client diberi tahu kekurangan saldo Rp 46.500
    Dan tidak ada baris mutasi dompet yang tercipta

  Skenario: Helper terpilih sudah mengambil pesanan lain
    # INV-06
    Diberikan client memilih tawaran helper B
    Dan helper B baru saja dikonfirmasi pada pesanan lain
    Ketika helper B mengonfirmasi pesanan ini
    Maka sistem menolak dengan kode HELPER_HAS_ACTIVE_ORDER
    Dan status pesanan kembali menjadi searching

  Skenario: Pengguna tidak bisa menawar pesanannya sendiri
    # FR-ORD-016, INV-05
    Diberikan pengguna yang juga helper terverifikasi membuat pesanan sebagai client
    Ketika pengguna tersebut mengajukan tawaran pada pesanannya sendiri
    Maka sistem menolak dengan kode SELF_ORDER_NOT_ALLOWED

  Skenario: Konfirmasi dikirim ulang karena jaringan buruk
    # FR-WLT-009, INV-04
    Diberikan helper menekan konfirmasi dan permintaan gagal karena waktu habis di jaringan
    Ketika aplikasi mengirim ulang dengan idempotency key yang sama
    Maka sistem mengembalikan hasil konfirmasi yang pertama
    Dan tidak tercipta penahanan dana kedua
```

```gherkin
Fitur: Direct booking
  # Baru 28 September 2026, DEC-21

  Skenario: Helper memberi quote dan client menyetujui
    # FR-ORD-003D, FR-WLT-002
    Diberikan client memesan langsung Delivery ke helper Agus
    Dan status pesanan awaiting_quote
    Ketika Agus mengirim quote Rp 50.000 dalam 300 detik
    Dan client menyetujui quote dalam 300 detik sejak quote masuk
    Maka dana client langsung ditahan tanpa konfirmasi ulang dari Agus
    Dan status pesanan menjadi accepted

  Skenario: Helper tidak memberi quote
    # FR-ORD-003D
    Diberikan pesanan direct berstatus awaiting_quote
    Ketika 300 detik berlalu tanpa quote
    Maka status pesanan menjadi expired
    Dan client melihat tombol siarkan ke helper lain

  Skenario: Quote kedaluwarsa sebelum disetujui
    # FR-ORD-003D
    Diberikan Agus mengirim quote pada pukul 10.00.00
    Ketika client menekan setuju pada pukul 10.05.01
    Maka sistem menolak dengan kode QUOTE_EXPIRED
    Dan status pesanan menjadi expired

  Skenario: Helper menolak permintaan quote
    # FR-ORD-003D
    Diberikan pesanan direct berstatus awaiting_quote
    Ketika Agus menolak permintaan
    Maka status tawaran menjadi declined dan status pesanan menjadi expired

  Skenario: Siarkan ulang membuat pesanan broadcast baru
    # FR-ORD-003D
    Diberikan pesanan direct berstatus expired
    Ketika client menekan siarkan ke helper lain
    Maka sistem membuat pesanan baru berjenis broadcast dengan kategori, deskripsi, alamat, dan batas talangan yang sama
    Dan pesanan lama tetap berstatus expired

  Skenario: Tombol pesan langsung nonaktif untuk helper yang sibuk
    # FR-ORD-003D, FR-ORD-017
    Diberikan helper Agus sedang memegang pesanan berstatus in_progress
    Ketika client membuka profil Agus
    Maka tombol pesan langsung tidak aktif
```

## 3. Talangan pada Food run dan Laundry

Direvisi 28 September 2026 mengikuti DEC-17, DEC-18, dan DEC-20. Skenario lama "client menolak penyesuaian, status menjadi disputed" dihapus.

```gherkin
Fitur: Struk talangan
  Latar:
    Diberikan pesanan Food run berstatus in_progress
    Dan upah jasa Rp 20.000 tanpa voucher
    Dan batas talangan Rp 50.000

  Skenario: Struk dalam batas langsung diakui
    # FR-ORD-014
    Ketika helper mengunggah struk Rp 43.000
    Maka status struk menjadi auto_approved
    Dan status pesanan tetap in_progress
    Dan client tidak perlu menyetujui apa pun

  Skenario: Beberapa struk dalam batas
    # FR-ORD-014, FR-ORD-019
    Ketika helper mengunggah struk Rp 30.000 lalu struk Rp 13.000
    Maka kedua struk berstatus auto_approved
    Dan total talangan yang diakui Rp 43.000

  Skenario: Struk melewati batas dan client menyetujui
    # FR-ORD-014
    Ketika helper mengunggah struk Rp 58.000
    Maka status pesanan menjadi price_adjustment
    Dan ketika client menyetujui
    Maka sistem menahan dana tambahan Rp 8.000
    Dan status pesanan kembali in_progress

  Skenario: Struk melewati batas dan client menolak
    # FR-ORD-014, FR-ORD-018, DEC-20
    Ketika helper mengunggah struk Rp 58.000
    Dan client menolak
    Maka status pesanan kembali in_progress, bukan disputed
    Dan talangan yang diakui hanya Rp 50.000
    Dan helper dapat memilih melanjutkan atau membatalkan

  Skenario: Struk kedua ditolak selama struk pertama menunggu
    # FR-ORD-019, INV-13
    Diberikan satu struk berstatus pending
    Ketika helper mengunggah struk lain
    Maka sistem menolak dengan kode ADJUSTMENT_PENDING_EXISTS

  Skenario: Kategori tanpa talangan tidak menerima struk
    # FR-ORD-014, DEC-18
    Diberikan pesanan Moving berstatus in_progress
    Ketika helper mengunggah struk
    Maka sistem menolak dengan kode CATEGORY_HAS_NO_ADVANCE
```

## 4. Penyelesaian dan settlement

```gherkin
Fitur: Menyelesaikan pesanan
  Skenario: Settlement Food run dengan voucher
    # FR-ORD-010, FR-WLT-003, FR-WLT-013, INV-09, INV-12
    Diberikan pesanan Food run berstatus awaiting_confirmation
    Dan upah jasa Rp 20.000, voucher Rp 2.000, batas talangan Rp 50.000
    Dan dana tertahan Rp 68.000
    Dan talangan yang diakui Rp 43.000
    Ketika client menekan tombol selesaikan pesanan
    Maka helper menerima Rp 18.000 upah bersih dan Rp 43.000 penggantian talangan
    Dan client menerima pengembalian Rp 7.000
    Dan komisi bersih platform Rp 0
    Dan penjumlahan dana ke helper, komisi bersih, dan pengembalian sama dengan Rp 68.000
    Dan riwayat mutasi helper memuat upah bersih dan penggantian talangan sebagai dua baris terpisah
    Dan status pesanan menjadi completed dalam waktu tidak lebih dari 5 detik

  Skenario: Settlement tanpa voucher
    # FR-WLT-003, INV-09
    Diberikan pesanan Delivery dengan harga jasa Rp 38.000 tanpa voucher
    Ketika pesanan dikonfirmasi selesai
    Maka helper menerima Rp 34.200
    Dan komisi bersih platform Rp 3.800

  Skenario: Pesanan wajib foto bukti sebelum ditandai selesai
    # FR-ORD-013
    Diberikan pesanan Laundry berstatus in_progress tanpa foto bukti penyelesaian
    Ketika helper menandai pekerjaan selesai
    Maka sistem menolak dengan kode COMPLETION_PROOF_REQUIRED

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

## 5. Pembatalan

Direvisi 28 September 2026 mengikuti DEC-19. Skenario lama dengan "30 persen tarif dasar" diganti.

```gherkin
Fitur: Pembatalan pesanan
  Skenario: Client membatalkan saat menunggu konfirmasi helper
    # FR-ORD-011, FR-WLT-014
    Diberikan pesanan berstatus pending_confirmation dengan voucher terpasang
    Ketika client membatalkan
    Maka tidak ada biaya pembatalan
    Dan voucher kembali ke client

  Skenario: Client membatalkan dalam masa tenggang
    # FR-ORD-011
    Diberikan pesanan diterima helper 60 detik yang lalu
    Ketika client membatalkan
    Maka seluruh dana tertahan kembali ke client
    Dan helper tidak menerima kompensasi
    Dan status pesanan menjadi cancelled_by_client

  Skenario: Client membatalkan setelah masa tenggang
    # FR-ORD-011, FR-WLT-014
    Diberikan pesanan Food run diterima 3 menit yang lalu dengan dana tertahan Rp 68.000 dan voucher Rp 2.000
    Ketika client membatalkan
    Maka helper menerima kompensasi Rp 5.000 tanpa potongan komisi
    Dan client menerima kembali Rp 63.000
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
    Diberikan pesanan berstatus accepted dengan voucher terpasang
    Ketika helper membatalkan
    Maka seluruh dana kembali ke client
    Dan voucher kembali ke client
    Dan jumlah pembatalan helper bertambah satu

  Skenario: Helper membatalkan Food run setelah belanja dalam batas
    # FR-ORD-023, DEC-19
    Diberikan pesanan Food run berstatus in_progress dengan dana tertahan Rp 70.000
    Dan struk Rp 30.000 sudah diakui
    Ketika helper membatalkan
    Maka helper menerima penggantian talangan Rp 30.000 dan tidak menerima upah
    Dan client menerima kembali Rp 40.000

  Skenario: Sengketa ditolak untuk kategori selain Delivery
    # FR-ORD-018, DEC-10
    Diberikan pesanan Household berstatus awaiting_confirmation
    Ketika client mengajukan sengketa
    Maka sistem menolak dengan kode DISPUTE_NOT_ELIGIBLE
```

## 5B. Foto penjemputan dan sengketa Delivery

Baru 28 September 2026, DEC-24.

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

## 6. Penilaian satu arah

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

## 6B. Panel admin

Direvisi 28 September 2026 mengikuti DEC-22, DEC-23, dan DEC-24.

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

  Skenario: Suspend berhasil membersihkan tawaran dan pesanan yang belum berjalan
    # FR-ADM-009
    Diberikan helper tidak memiliki pesanan aktif tetapi punya dua tawaran berstatus submitted
    Ketika admin men-suspend helper tersebut
    Maka kedua tawaran menjadi withdrawn
    Dan seluruh sesi login helper dicabut

  Skenario: Admin membatalkan Food run yang sudah belanja
    # FR-ADM-011, DEC-22
    Diberikan pesanan Food run berstatus in_progress dengan dana tertahan Rp 68.000
    Dan talangan sah yang diakui Rp 40.000
    Ketika admin membatalkan pesanan dengan alasan
    Maka helper menerima penggantian Rp 40.000 tanpa kompensasi pembatalan
    Dan client menerima kembali Rp 28.000
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

## 7. Catatan pengujian tambahan untuk QA

Data uji minimum yang perlu disiapkan mencakup satu akun client bersaldo besar, satu akun client bersaldo nol, dua akun helper terverifikasi dengan batas talangan maksimal berbeda (misalnya Rp 30.000 dan Rp 100.000), satu akun helper yang masih menunggu peninjauan, satu akun yang memegang dua peran sekaligus untuk menguji `INV-05`, dan satu pesanan pada setiap status untuk menguji transisi yang tidak sah.

Pengujian dompet sebaiknya selalu diakhiri dengan pemeriksaan rekonsiliasi, yaitu memastikan jumlah saldo tersedia ditambah saldo tertahan sama dengan hasil penjumlahan seluruh mutasi. Kalau angkanya meleset satu rupiah pun, ada cacat pada logika transaksi meskipun antarmuka terlihat normal.

Pengujian waktu seperti konfirmasi helper 60 detik, jendela penawaran dan quote 300 detik, masa tenggang pembatalan 120 detik, dan konfirmasi otomatis 24 jam sebaiknya dilakukan dengan kemampuan memajukan waktu server di lingkungan pengujian, bukan dengan menunggu sungguhan. Ini perlu diminta ke BE sejak awal supaya tidak menjadi hambatan menjelang UAT.
