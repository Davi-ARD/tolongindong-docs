# Skenario Alur Pengguna

Status dokumen: `REVIEW` v2.0, 19 Oktober 2026. Versi 1.0 (28 September 2026) adalah penulisan ulang `scenario.md` dari Backend dengan Food run sebagai skenario utama. Versi 2.0 ditulis ulang mengikuti DEC-25 sampai DEC-27 (`14_perubahan-model-marketplace.md`): Food run, Moving, dan Laundry dihapus, client memilih helper sendiri dari daftar, dan client bisa chat helper sebelum memesan. Dokumen ini adalah narasi untuk UI/UX dan QA. Aturan resminya tetap hidup di `02_requirement-master-list.md` dan `04_dual-role-transaction-analysis.md`. Kalau narasi ini berbeda dengan dua file itu, dua file itulah yang benar.

Semua angka uang sudah diuji terhadap rumus di `05_erd-draft.md` bagian 2.2.

## 1. Skenario utama, Household dari mencari helper sampai selesai

Budi adalah client yang ingin kamar kosnya dibersihkan. Agus adalah helper terverifikasi yang menguasai Household dan Delivery.

### Tahap 1, pendaftaran dan masuk aplikasi

1. Budi membuka aplikasi dan melihat tiga layar onboarding, lalu menekan "Mulai".
2. Budi mengisi nama lengkap, nomor telepon, email, dan kata sandi, lalu menekan "Daftar".
3. Sistem mengirim kode OTP enam digit lewat SMS. Budi memasukkannya sebelum 300 detik habis.
4. Akun aktif. Sistem membuat TD-Wallet dengan saldo Rp 0.
5. Budi masuk memakai nomor telepon dan kata sandi, lalu tiba di beranda.

### Tahap 2, isi saldo

1. Budi membuka Profil, lalu kartu TD-Wallet, lalu "Isi Saldo".
2. Budi memilih Rp 100.000. Karena dompet berjalan dalam mode simulasi, sistem langsung mengonfirmasi tanpa pembayaran sungguhan.
3. Saldo tersedia Budi menjadi Rp 100.000 dan tercatat di riwayat mutasi.

### Tahap 3, mencari helper

1. Di beranda, Budi melihat tiga kategori: Delivery, Personal, dan Household. Budi menekan Household.
2. Sistem membuka daftar helper Household. Isinya hanya helper terverifikasi yang sedang menyalakan ketersediaan, menguasai Household, dan berada dalam 10 km dari lokasi aktif Budi. Urutan bawaan adalah jarak terdekat.
3. Setiap kartu menampilkan nama, rating, jumlah ulasan, jarak, tarif per jam, dan tag keahlian. Tarif per jam hanya gambaran, bukan harga pesanan.
4. Satu helper bertanda "Sedang sibuk" karena sedang mengerjakan pesanan lain. Kartunya tetap muncul, tetapi tombol Pesan di halaman detailnya nonaktif.
5. Budi membuka profil Agus: rating 4,8 dari 32 ulasan, 1,4 km, tarif Rp 25.000 per jam, menguasai Household dan Delivery.

### Tahap 4, chat sebelum memesan

1. Budi menekan "Chat". Sistem membuat ruang percakapan untuk pasangan Budi dan Agus.
2. Di bagian atas ruang tampil pemberitahuan: pembayaran di luar TD-Wallet tidak dilindungi penahanan dana dan tidak bisa dibantu lewat sengketa. Nomor telepon Agus tampil tersamar.
3. Budi menulis: "Mas, besok jam 9 bisa bersihkan kamar kos 3x4 m dan cuci piring? Kira kira berapa?"
4. Agus membalas: "Bisa Mas. Sekitar Rp 50.000 ya, sudah termasuk cuci piring."
5. Belum ada pesanan dan belum ada uang yang ditahan.

### Tahap 5, mengirim permintaan pesanan

1. Budi menekan "Pesan Agus" di ruang percakapan. Form permintaan terbuka dengan Agus dan kategori Household sudah terisi.
2. Karena Household, form hanya meminta satu alamat berlabel "Lokasi pengerjaan". Budi memilih "Kos Budi, Kamar 12".
3. Budi mengisi deskripsi "Bersihkan kamar kos 3x4 m dan cuci piring", waktu besok pukul 09.00, dan estimasi upah Rp 50.000 sesuai obrolan.
4. Sistem menampilkan rentang biaya acuan sebagai pembanding.
5. Budi memasang voucher HEMAT2K, potongan tetap Rp 2.000 dengan minimal pesanan Rp 20.000.
6. Budi menekan "Kirim Permintaan". Status pesanan `awaiting_quote`. Agus punya 300 detik untuk memberi quote atau menolak. Belum ada uang yang ditahan.

### Tahap 6, helper memberi quote

1. Agus menerima notifikasi permintaan dan membuka detailnya.
2. Agus mengisi quote Rp 50.000. Sebelum mengirim, aplikasi menunjukkan:
   - Upah jasa: Rp 50.000
   - Komisi platform 10 persen: Rp 5.000
   - Upah bersih: Rp 45.000
3. Agus menekan "Kirim Quote". Quote berlaku 300 detik.

### Tahap 7, Budi menyetujui dan dana ditahan

1. Budi melihat quote. Sistem mengecek voucher terhadap Rp 50.000. Syarat minimal Rp 20.000 terpenuhi, jadi voucher berlaku.
2. Layar Budi menunjukkan rincian:
   - Upah jasa: Rp 50.000
   - Voucher: minus Rp 2.000
   - Total yang akan ditahan: Rp 48.000
3. Budi menekan "Setuju dan Bayar". Sistem menahan Rp 48.000 tanpa meminta Agus mengonfirmasi ulang. Saldo tersedia Budi tinggal Rp 52.000, saldo tertahan Rp 48.000.
4. Status pesanan menjadi `accepted`. Kalau saat itu Agus juga sedang menunggu jawaban quote dari client lain, permintaan itu otomatis `expired` dan client tersebut diberi tahu untuk memilih helper lain (FR-ORD-024).

### Tahap 8, pelaksanaan

1. Esok paginya Agus menekan "Berangkat", status menjadi `on_the_way`.
2. Di kos Budi, Agus menekan "Tiba di Lokasi" lalu "Mulai Mengerjakan", status menjadi `in_progress`. Household tidak mewajibkan foto penjemputan.
3. Agus bertanya lewat ruang percakapan yang sama soal tempat sampah. Pesan ini otomatis ditandai dengan nomor pesanan.
4. Setelah selesai, Agus menekan "Selesaikan Pekerjaan". Status menjadi `awaiting_confirmation`. Household tidak mewajibkan foto bukti, berbeda dengan Delivery.

### Tahap 9, konfirmasi dan settlement

1. Budi menerima notifikasi untuk memeriksa pekerjaan.
2. Budi menekan "Konfirmasi Selesai".
3. Sistem menjalankan settlement dari dana tertahan Rp 48.000:

| Komponen | Nominal |
| --- | --- |
| Upah bersih Agus | Rp 45.000 |
| Komisi kotor platform | Rp 5.000 |
| Dipakai menanggung voucher | Rp 2.000 |
| Komisi bersih platform | Rp 3.000 |
| Kembali ke Budi | Rp 0 |
| Total | Rp 48.000 |

4. Saldo Agus bertambah Rp 45.000. Saldo tersedia Budi tetap Rp 52.000, karena seluruh dana tertahan terpakai.
5. Status pesanan menjadi `completed`.

Voucher ditanggung platform dari komisinya, bukan dari upah Agus. Upah bersih Agus tetap Rp 45.000 dengan atau tanpa voucher.

### Tahap 10, penilaian

1. Budi memberi bintang 5 dan ulasan singkat.
2. Rating rata rata Agus langsung diperbarui.
3. Ruang percakapan Budi dan Agus tetap terbuka. Kalau bulan depan Budi butuh Agus lagi, dia cukup membuka ruang yang sama.

## 2. Skenario alternatif dan kondisi khusus

### Skenario A, mendaftar menjadi helper

Rian ingin mendapat penghasilan tambahan.

1. Rian membuka Profil, lalu "Daftar Menjadi Helper".
2. Rian memilih kategori Delivery dan Household. Pilihan maksimal tiga kategori.
3. Rian mengunggah foto KTP dan swafoto memegang KTP. Status pengajuan `pending`. Selama status ini, Rian belum muncul di daftar helper.

**Kondisi A.1, ditolak karena foto buram.** Admin menolak dengan catatan "Foto KTP silau dan NIK tidak terbaca". Rian langsung memotret ulang dan mengirim lagi saat itu juga, tanpa masa tunggu (FR-VER-007). Sistem hanya menolak kalau Rian mencoba mengirim pengajuan baru selagi pengajuan sebelumnya masih `pending`.

**Kondisi A.2, disetujui.** Status Rian menjadi `verified`, lencana terverifikasi muncul, dan sakelar "Terima Pesanan" bisa dinyalakan. Begitu sakelar menyala, Rian muncul di daftar helper Delivery dan Household.

### Skenario B, helper tidak merespons atau menolak

Budi mengirim permintaan Delivery ke Agus untuk mengantar dokumen, tetapi Agus sedang di luar kota.

1. Agus menekan "Tolak" dan menulis alasan "Sedang di luar kota". Status pesanan menjadi `expired`. Penolakan ini tidak menambah catatan pembatalan Agus.
2. Kalau Agus tidak membuka aplikasi sama sekali, hasilnya sama setelah 300 detik lewat.
3. Budi melihat tombol "Pilih helper lain". Sistem membuka daftar helper Delivery.
4. Budi memilih Dedi. Sistem membuat pesanan baru ke Dedi dengan deskripsi, alamat, waktu, dan voucher yang sama. Pesanan lama ke Agus tetap `expired`.

### Skenario C, quote tidak sesuai obrolan

Di chat, Agus menyebut sekitar Rp 50.000. Saat quote masuk, angkanya Rp 70.000.

1. Budi tidak bisa menawar balik di layar quote.
2. Budi bisa bertanya lewat ruang percakapan yang sama, misalnya "Kok jadi 70 ribu, Mas?".
3. Kalau tetap tidak cocok, Budi membiarkan quote kedaluwarsa setelah 300 detik atau membatalkan permintaan. Pembatalan pada `awaiting_quote` gratis dan voucher kembali.
4. Budi lalu memilih helper lain lewat tombol "Pilih helper lain".

### Skenario D, satu helper dipesan dua client

Agus menerima permintaan dari Budi dan dari Citra pada pagi yang sama, dan mengirim quote ke keduanya.

1. Budi menyetujui lebih dulu. Dana Budi ditahan dan pesanan Budi menjadi `accepted`.
2. Pada transaksi yang sama, pesanan Citra berubah menjadi `expired`. Citra menerima notifikasi bahwa helper sudah menerima pesanan lain, lengkap dengan tombol "Pilih helper lain".
3. Kalau Citra menekan setuju pada detik yang hampir bersamaan, sistem menolak dengan `HELPER_HAS_ACTIVE_ORDER` dan tidak menahan dana Citra.

### Skenario E, pembatalan

**Kasus 1, Budi batal saat menunggu quote.** Status `awaiting_quote`, belum ada dana tertahan. Batal gratis, voucher kembali, dan quote yang sudah masuk berubah menjadi `withdrawn`.

**Kasus 2, Budi batal tak lama setelah menyetujui.** Kalau kurang dari 120 detik sejak `accepted`, batal gratis dan voucher kembali. Kalau sudah 3 menit, pada contoh Tahap 7 dengan dana tertahan Rp 48.000, Agus menerima kompensasi Rp 5.000 utuh tanpa potongan komisi, Budi menerima kembali Rp 43.000, dan voucher hangus.

**Kasus 3, Budi batal saat helper sudah jalan.** Pada Delivery dengan quote Rp 40.000 dan voucher Rp 4.000, dana tertahan Rp 36.000. Helper sedang `on_the_way`. Sebelum menekan batal, Budi melihat pratinjau: helper menerima 25 persen dari Rp 40.000, yaitu Rp 10.000, dan Budi menerima kembali Rp 26.000. Voucher hangus.

**Kasus 4, Budi ingin batal saat pekerjaan berjalan.** Status `in_progress`. Tombol batal tidak tersedia untuk client.

**Kasus 5, Agus membatalkan.** Agus mendadak sakit setelah mulai bekerja. Dana jasa Budi Rp 48.000 kembali penuh, voucher kembali, dan catatan pembatalan Agus bertambah satu. Agus tidak menerima upah.

### Skenario F, Budi lupa menekan konfirmasi

Agus sudah menandai selesai, tetapi Budi tidak membuka aplikasi. Setelah 24 jam tanpa konfirmasi dan tanpa sengketa, sistem mengonfirmasi otomatis dan settlement berjalan seperti Tahap 9.

### Skenario G, sengketa Delivery

Budi memakai Delivery untuk mengirim kue tart. Quote Rp 60.000 tanpa voucher, dana tertahan Rp 60.000.

1. Saat menjemput kue, Agus wajib memotret kondisi kue sebelum menekan "Mulai Mengerjakan". Tanpa foto ini, sistem menolak perubahan status (FR-ORD-021).
2. Setelah mengantar, Agus wajib memotret kue di tujuan sebelum menekan "Selesaikan Pekerjaan" (FR-ORD-013).
3. Penerima mendapati kue hancur. Dalam 24 jam sejak Agus menandai selesai, Budi menekan "Ajukan Sengketa", menulis alasan, dan melampirkan foto.
4. Status menjadi `disputed` dan dana tetap tertahan.
5. Admin membandingkan foto penjemputan dengan foto dari Budi, lalu membaca pesan chat yang bertanda nomor pesanan itu.
6. Admin memilih satu dari tiga keputusan dan wajib mengisi catatan:

| Keputusan | Ke Agus | Ke Budi | Komisi platform | Status akhir |
| --- | --- | --- | --- | --- |
| Berpihak helper | Rp 54.000 | Rp 0 | Rp 6.000 | `completed` |
| Berpihak client | Rp 0 | Rp 60.000 | Rp 0 | `refunded` |
| Split | Rp 30.000 | Rp 30.000 | Rp 0 | `partially_refunded` |

Admin tidak mengetik persentase pembagian. Pilihan split selalu 50:50 dari dana tertahan, tanpa komisi.

Sengketa hanya tersedia untuk Delivery. Pada Personal dan Household, tombol "Ajukan Sengketa" tidak muncul.

### Skenario H, suspend akun

Seorang helper dilaporkan berkata kasar. Admin membuka profilnya dan menekan "Bekukan Akun".

**Kalau masih ada pesanan aktif.** Sistem memeriksa apakah pengguna punya pesanan berstatus `accepted` sampai `awaiting_confirmation`, atau `disputed`, baik sebagai client maupun helper. Kalau ada, sistem menolak dan menampilkan daftar nomor pesanan itu. Admin harus menunggu pesanan selesai, memutuskan sengketanya, atau membatalkannya lewat Skenario J.

**Kalau tidak ada pesanan aktif.** Akun dibekukan. Sistem membatalkan seluruh permintaan `awaiting_quote` yang ditujukan ke helper itu maupun yang dia buat sebagai client, memberi tahu client terkait, dan mencabut semua sesi login. Ruang percakapan tidak dihapus, tetapi tidak bisa dipakai sampai akun diaktifkan kembali.

### Skenario I, admin menerbitkan voucher

1. Admin membuka "Manajemen Voucher" dan menekan "Buat Voucher Baru".
2. Admin mengisi kode HEMAT5K, potongan tetap Rp 5.000.
3. Admin mencoba minimal pesanan Rp 45.000. Sistem menolak penyimpanan, bukan hanya memberi peringatan: "Minimal pesanan harus Rp 50.000 agar potongan tidak melebihi komisi platform."
4. Admin mengubah minimal pesanan menjadi Rp 50.000 dengan kuota 30 klaim. Voucher tersimpan.
5. Kalau admin membuat voucher persentase, sistem menolak persentase di atas 10.

Manajemen voucher berprioritas Should. Untuk demo, voucher boleh berasal dari data seed.

### Skenario J, admin membatalkan pesanan yang sedang berjalan

Admin perlu membekukan akun Agus, tetapi Agus sedang mengerjakan pesanan Household milik Budi dengan quote Rp 50.000, voucher Rp 2.000, dan dana tertahan Rp 48.000.

1. Admin membuka pesanan dan menekan "Batalkan Pesanan", lalu mengisi alasan.
2. Budi menerima kembali Rp 48.000 dan voucher Budi kembali. Agus tidak menerima kompensasi maupun upah.
3. Status pesanan menjadi `cancelled_by_admin`. Tindakan tercatat di log admin.
4. Setelah itu admin bisa membekukan akun Agus.

Admin tidak bisa memakai tombol ini untuk pesanan `disputed`. Pesanan sengketa diselesaikan lewat keputusan sengketa di Skenario G.

### Skenario K, Personal

Saat Budi memilih Personal, form menampilkan lima jenis pekerjaan yang tidak termasuk kategori ini: jasa yang butuh sertifikasi atau izin, mengangkut penumpang, membelikan barang atau menangani uang tunai atas nama client, aktivitas melanggar hukum, dan pekerjaan berbahaya atau berisiko tinggi seperti bekerja di atap. Form hanya meminta satu alamat berlabel "Lokasi pengerjaan".

Larangan ketiga penting sejak Food run dihapus. Tanpa larangan itu, client bisa memakai Personal untuk minta dibelikan makanan, padahal sistem tidak lagi punya mekanisme talangan dan struk untuk melindungi uang helper.

### Skenario L, chat tanpa jadi memesan

Budi ingin membandingkan tiga helper Household sebelum memilih.

1. Budi membuka ruang percakapan dengan Agus, Sari, dan Dedi, lalu menanyakan hal yang sama ke ketiganya.
2. Budi memesan Agus. Ruang percakapan dengan Sari dan Dedi tetap ada tanpa pesanan.
3. Sistem membatasi Budi membuka maksimal 20 ruang percakapan baru per hari, supaya helper tidak dibanjiri chat tanpa niat memesan.
4. Sari tetap bisa mengirim pesan di ruang yang sudah Budi buka, tetapi Sari tidak bisa membuka ruang baru ke client lain yang belum pernah menghubunginya.
