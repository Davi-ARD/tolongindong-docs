# Skenario Alur Pengguna

Status dokumen: `REVIEW` v1.0, 28 September 2026. Naskah awal disusun Backend sebagai `scenario.md`, lalu SA menulis ulang mengikuti DEC-14 sampai DEC-24, terutama model talangan Food run yang memisahkan upah jasa dari uang belanja (DEC-17). Dokumen ini adalah narasi untuk UI/UX dan QA. Aturan resminya tetap hidup di `02_requirement-master-list.md` dan `04_dual-role-transaction-analysis.md`. Kalau narasi ini berbeda dengan dua file itu, dua file itulah yang benar.

Semua angka uang sudah diuji terhadap rumus di `05_erd-draft.md` bagian 2.2.

## 1. Skenario utama, Food run dari awal sampai selesai

Budi adalah client yang ingin dibelikan makan siang lewat Food run. Agus adalah helper terverifikasi yang mengerjakannya.

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

### Tahap 3, membuat pesanan Food run

1. Budi memilih kategori Food run di beranda.
2. Budi mengisi deskripsi: "Tolong belikan Nasi Padang rendang 1 porsi di RM Sederhana, sambal dipisah."
3. Titik jemput adalah lokasi RM Sederhana. Titik tujuan adalah alamat utama Budi, "Kos Budi, Kamar 12".
4. Sistem menampilkan rentang acuan upah jasa, misalnya Rp 12.000 sampai Rp 25.000.
5. Budi mengisi dua angka terpisah:
   - Estimasi upah jasa: Rp 15.000.
   - Batas talangan (uang belanja maksimal): Rp 50.000.
6. Budi memasang voucher HEMAT2K, potongan Rp 2.000 dengan minimal pesanan Rp 20.000.
7. Budi menekan "Pesan Sekarang". Status pesanan menjadi `searching` dan jendela penawaran terbuka 300 detik. Belum ada uang yang ditahan.

### Tahap 4, helper menawar

1. Agus sedang mengaktifkan ketersediaan dan punya batas talangan maksimal Rp 100.000, jadi dia menerima notifikasi. Helper lain dengan batas talangan maksimal Rp 30.000 tidak menerima notifikasi pesanan ini (FR-ORD-020).
2. Agus membuka pesanan dan mengajukan upah jasa Rp 20.000. Sebelum mengirim, aplikasi menunjukkan:
   - Upah jasa: Rp 20.000
   - Komisi platform 10 persen: Rp 2.000
   - Upah bersih: Rp 18.000
   - Uang belanja sampai Rp 50.000 diganti penuh di luar upah
3. Agus menekan "Kirim Penawaran".
4. Dedi, helper lain, mengajukan upah Rp 23.000.

### Tahap 5, Budi memilih dan dana ditahan

1. Budi melihat dua tawaran beserta profil dan rating.
2. Budi memilih tawaran Agus. Sistem mengecek voucher terhadap Rp 20.000. Syarat minimal Rp 20.000 terpenuhi, jadi voucher berlaku.
3. Layar Budi menunjukkan rincian:
   - Upah jasa: Rp 20.000
   - Voucher: minus Rp 2.000
   - Jasa yang Budi bayar: Rp 18.000
   - Batas talangan: Rp 50.000
   - Total yang akan ditahan: Rp 68.000
4. Status pesanan menjadi `pending_confirmation`. Agus menerima permintaan konfirmasi dengan batas 60 detik.
5. Agus menekan "Saya Siap Berangkat".
6. Sistem menahan Rp 68.000 dari saldo Budi. Saldo tersedia Budi tinggal Rp 32.000, saldo tertahan Rp 68.000.
7. Tawaran Dedi otomatis ditutup. Status pesanan menjadi `accepted`, dan ruang chat Budi dengan Agus terbuka.

### Tahap 6, pelaksanaan

1. Agus menekan "Menuju Lokasi Pembelian", status menjadi `on_the_way`.
2. Di RM Sederhana, Agus menekan "Tiba di Lokasi" lalu "Mulai Mengerjakan", status menjadi `in_progress`.
3. Agus bertanya lewat chat soal kuah, Budi menjawab.
4. Agus membayar Rp 43.000 di kasir, memotret struk, dan mengunggahnya. Karena Rp 43.000 masih di bawah batas Rp 50.000, sistem langsung mengakui struk itu (`auto_approved`) tanpa menunggu Budi. Status pesanan tetap `in_progress`.

### Tahap 7, pengantaran

1. Agus mengantar makanan ke kos Budi dan menyerahkannya bersama struk fisik.
2. Agus memotret makanan di depan kamar sebagai bukti penyelesaian. Tanpa foto ini, tombol selesai ditolak sistem (FR-ORD-013).
3. Agus menekan "Selesaikan Pekerjaan". Status menjadi `awaiting_confirmation`.

### Tahap 8, konfirmasi dan settlement

1. Budi menerima notifikasi untuk memeriksa pesanan.
2. Budi menekan "Konfirmasi Selesai".
3. Sistem menjalankan settlement dari dana tertahan Rp 68.000:

| Komponen | Nominal |
| --- | --- |
| Upah bersih Agus | Rp 18.000 |
| Penggantian talangan Agus | Rp 43.000 |
| Total ke Agus | Rp 61.000 |
| Komisi kotor platform | Rp 2.000 |
| Dipakai menanggung voucher | Rp 2.000 |
| Komisi bersih platform | Rp 0 |
| Sisa batas talangan kembali ke Budi | Rp 7.000 |
| Total | Rp 68.000 |

4. Saldo Agus bertambah Rp 61.000, tercatat sebagai dua baris mutasi: upah bersih dan penggantian talangan.
5. Saldo tersedia Budi menjadi Rp 39.000. Budi membayar total Rp 61.000, yaitu Rp 18.000 jasa dan Rp 43.000 makanan.
6. Status pesanan menjadi `completed`.

Voucher ditanggung platform dari komisinya, bukan dari upah Agus. Upah bersih Agus tetap Rp 18.000 dengan atau tanpa voucher.

### Tahap 9, penilaian

1. Budi memberi bintang 5 dan ulasan singkat.
2. Rating rata rata Agus langsung diperbarui.
3. Ruang chat menjadi baca saja tujuh hari kemudian.

## 2. Skenario alternatif dan kondisi khusus

### Skenario A, mendaftar menjadi helper

Rian ingin mendapat penghasilan tambahan.

1. Rian membuka Profil, lalu "Daftar Menjadi Helper".
2. Rian memilih kategori Delivery dan Household.
3. Rian mengisi batas talangan maksimal yang bersedia dia tanggung, misalnya Rp 100.000.
4. Rian mengunggah foto KTP dan swafoto memegang KTP. Status pengajuan `pending`.

**Kondisi A.1, ditolak karena foto buram.** Admin menolak dengan catatan "Foto KTP silau dan NIK tidak terbaca". Rian langsung memotret ulang dan mengirim lagi saat itu juga, tanpa masa tunggu (FR-VER-007). Sistem hanya menolak kalau Rian mencoba mengirim pengajuan baru selagi pengajuan sebelumnya masih `pending`.

**Kondisi A.2, disetujui.** Status Rian menjadi `verified`, lencana terverifikasi muncul, dan sakelar "Terima Order" bisa dinyalakan.

### Skenario B, direct booking

Budi puas dengan Agus dan ingin memesan Agus lagi untuk mengantar dokumen (Delivery).

1. Budi membuka profil Agus dan menekan "Pesan Langsung". Tombol ini hanya aktif kalau Agus tersedia, tidak sedang memegang pesanan aktif, dan batas talangan maksimalnya cukup.
2. Budi mengisi detail dan mengirim. Status pesanan `awaiting_quote`. Pesanan tidak disiarkan ke helper lain.
3. Agus punya 300 detik untuk memberi quote. Agus mengirim quote upah Rp 30.000 dan melihat upah bersihnya Rp 27.000.
4. Budi melihat quote dan punya 300 detik untuk menyetujui.
5. Budi menekan "Setuju dan Bayar". Sistem langsung menahan Rp 30.000 tanpa meminta Agus mengonfirmasi ulang. Status menjadi `accepted`.

**Kalau gagal.** Kalau Agus tidak memberi quote dalam 300 detik, menolak permintaan, atau Budi tidak menyetujui quote dalam 300 detik, pesanan menjadi `expired`. Budi melihat tombol "Siarkan ke helper lain" yang membuat pesanan broadcast baru dengan data yang sama. Budi tidak bisa menawar balik quote Agus pada versi ini.

### Skenario C, belanja melewati batas talangan

Budi memesan Food run dengan upah jasa Rp 15.000 dan batas talangan Rp 30.000. Di toko, total belanja ternyata Rp 42.000.

1. Aplikasi mendorong Agus mengajukan penyesuaian sebelum membayar di kasir, karena belanja di atas batas tidak dijamin tanpa persetujuan Budi.
2. Agus memotret daftar harga atau struk sementara, lalu mengunggah nominal Rp 42.000. Status pesanan menjadi `price_adjustment`.
3. Agus tidak bisa mengunggah struk lain selama pengajuan ini belum dijawab (FR-ORD-019).

**Kalau Budi menyetujui.** Sistem menahan tambahan Rp 12.000. Dana tertahan menjadi Rp 57.000 dan status kembali `in_progress`. Saat selesai, Agus menerima Rp 13.500 upah bersih ditambah Rp 42.000 talangan, total Rp 55.500. Komisi platform Rp 1.500. Tidak ada sisa yang kembali ke Budi.

**Kalau Budi menolak.** Status kembali `in_progress`, bukan `disputed` (DEC-20). Agus hanya dijamin penggantian sampai batas Rp 30.000. Agus bisa berdiskusi lewat chat lalu belanja dalam batas lama, atau membatalkan pesanan. Kalau Agus sudah terlanjur membayar Rp 42.000, selisih Rp 12.000 menjadi risikonya.

### Skenario D, pembatalan

**Kasus 1, Budi batal saat mencari helper.** Status `searching`, belum ada dana tertahan. Batal gratis dan voucher kembali.

**Kasus 2, tidak ada tawaran selama 300 detik.** Status menjadi `expired`. Sistem menyarankan Budi menaikkan estimasi upah.

**Kasus 3, Budi batal tak lama setelah Agus menerima.** Kalau kurang dari 120 detik sejak `accepted`, batal gratis dan voucher kembali. Kalau sudah 3 menit, pada contoh Tahap 5 dengan dana tertahan Rp 68.000, Agus menerima kompensasi Rp 5.000 utuh tanpa potongan komisi, Budi menerima kembali Rp 63.000, dan voucher hangus.

**Kasus 4, Budi batal saat Agus sudah jalan.** Pada Delivery dengan upah Rp 40.000 dan voucher Rp 4.000, dana tertahan Rp 36.000. Agus sedang `on_the_way`. Sebelum menekan batal, Budi melihat pratinjau: Agus menerima 25 persen dari Rp 40.000, yaitu Rp 10.000, dan Budi menerima kembali Rp 26.000. Voucher hangus.

**Kasus 5, Budi ingin batal saat pekerjaan berjalan.** Status `in_progress`. Tombol batal tidak tersedia untuk client.

**Kasus 6, Agus membatalkan.** Motor Agus mogok setelah menerima pesanan. Dana jasa Budi kembali penuh, voucher kembali, dan catatan pembatalan Agus bertambah satu. Kalau Agus sudah belanja dalam batas dan struknya sudah diakui, misalnya Rp 30.000, Agus tetap menerima penggantian Rp 30.000, tetapi tidak menerima upah.

### Skenario E, Budi lupa menekan konfirmasi

Agus sudah menandai selesai, tetapi Budi tidak membuka aplikasi. Setelah 24 jam tanpa konfirmasi dan tanpa sengketa, sistem mengonfirmasi otomatis dan settlement berjalan seperti Tahap 8.

### Skenario F, helper terpilih tidak merespons

Budi memilih tawaran Helper C, tetapi Helper C tidak menekan konfirmasi dalam 60 detik. Tawaran Helper C menjadi `expired`, status pesanan kembali `searching`, dan dana Budi belum dipotong sama sekali. Budi memilih tawaran lain yang masih berlaku.

### Skenario G, sengketa Delivery

Budi memakai Delivery untuk mengirim kue tart. Upah jasa Rp 60.000 tanpa voucher, dana tertahan Rp 60.000.

1. Saat menjemput kue, Agus wajib memotret kondisi kue sebelum menekan "Mulai Mengerjakan". Tanpa foto ini, sistem menolak perubahan status (FR-ORD-021).
2. Penerima mendapati kue hancur. Dalam 24 jam sejak Agus menandai selesai, Budi menekan "Ajukan Sengketa", menulis alasan, dan melampirkan foto.
3. Status menjadi `disputed` dan dana tetap tertahan.
4. Admin membandingkan foto penjemputan dengan foto dari Budi, lalu membaca riwayat chat.
5. Admin memilih satu dari tiga keputusan dan wajib mengisi catatan:

| Keputusan | Ke Agus | Ke Budi | Komisi platform | Status akhir |
| --- | --- | --- | --- | --- |
| Berpihak helper | Rp 54.000 | Rp 0 | Rp 6.000 | `completed` |
| Berpihak client | Rp 0 | Rp 60.000 | Rp 0 | `refunded` |
| Split | Rp 30.000 | Rp 30.000 | Rp 0 | `partially_refunded` |

Admin tidak mengetik persentase pembagian. Pilihan split selalu 50:50 dari dana tertahan, tanpa komisi.

Sengketa hanya tersedia untuk Delivery. Pada kategori lain, tombol "Ajukan Sengketa" tidak muncul.

### Skenario H, suspend akun

Seorang pengguna dilaporkan berkata kasar. Admin membuka profilnya dan menekan "Bekukan Akun".

**Kalau masih ada pesanan aktif.** Sistem memeriksa apakah pengguna punya pesanan berstatus `pending_confirmation` sampai `awaiting_confirmation`, atau `disputed`, baik sebagai client maupun helper. Kalau ada, sistem menolak dan menampilkan daftar nomor pesanan itu. Admin harus menunggu pesanan selesai, memutuskan sengketanya, atau membatalkannya lewat Skenario J.

**Kalau tidak ada pesanan aktif.** Akun dibekukan. Sistem menarik semua tawaran pengguna yang masih diajukan, membatalkan pesanan client miliknya yang masih mencari helper atau menunggu quote, dan mencabut semua sesi login.

### Skenario I, admin menerbitkan voucher

1. Admin membuka "Manajemen Voucher" dan menekan "Buat Voucher Baru".
2. Admin mengisi kode HEMAT5K, potongan tetap Rp 5.000.
3. Admin mencoba minimal pesanan Rp 45.000. Sistem menolak penyimpanan, bukan hanya memberi peringatan: "Minimal pesanan harus Rp 50.000 agar potongan tidak melebihi komisi platform."
4. Admin mengubah minimal pesanan menjadi Rp 50.000 dengan kuota 30 klaim. Voucher tersimpan.
5. Kalau admin membuat voucher persentase, sistem menolak persentase di atas 10.

Manajemen voucher berprioritas Should. Untuk demo, voucher boleh berasal dari data seed.

### Skenario J, admin membatalkan pesanan Food run

Admin perlu membekukan akun Agus, tetapi Agus sedang mengerjakan Food run milik Budi. Upah jasa Rp 20.000, voucher Rp 2.000, batas talangan Rp 50.000, dana tertahan Rp 68.000, dan Agus sudah belanja Rp 40.000 dengan struk yang diakui.

1. Admin membuka pesanan dan menekan "Batalkan Pesanan", lalu mengisi alasan.
2. Sistem mengganti talangan sah Agus Rp 40.000, tanpa kompensasi pembatalan dan tanpa upah.
3. Budi menerima kembali Rp 28.000, dan voucher Budi kembali.
4. Status pesanan menjadi `cancelled_by_admin`. Tindakan tercatat di log admin.
5. Setelah itu admin bisa membekukan akun Agus.

Admin tidak bisa memakai tombol ini untuk pesanan `disputed`. Pesanan sengketa diselesaikan lewat keputusan sengketa di Skenario G.

### Skenario K, Laundry dan Personal

**Laundry.** Budi memesan Laundry dengan upah Rp 15.000 dan batas talangan Rp 40.000. Agus menjemput pakaian, membawanya ke penyedia laundry kiloan, membayar Rp 28.000 dengan talangan, mengunggah struk, lalu mengambil dan mengantar pakaian kembali. Settlement mengikuti pola Food run: Agus menerima upah bersih Rp 13.500 ditambah talangan Rp 28.000, dan sisa batas Rp 12.000 kembali ke Budi. Agus tidak mencuci sendiri.

**Personal.** Saat Budi memilih Personal, form menampilkan lima jenis pekerjaan yang tidak termasuk kategori ini: jasa yang butuh sertifikasi atau izin, mengangkut penumpang, menangani uang tunai pihak lain di luar talangan tercatat, aktivitas melanggar hukum, dan pekerjaan berbahaya atau berisiko tinggi seperti bekerja di atap. Form hanya meminta satu alamat berlabel "Lokasi pengerjaan".
