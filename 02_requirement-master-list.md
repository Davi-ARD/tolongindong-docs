# Requirement Master List

Status dokumen: `REVIEW` v0.3, 19 Oktober 2026. Ini satu satunya tempat FR dan NFR hidup. Tabel di SRS Word menyalin dari sini.

Perubahan v0.3 mengikuti DEC-25 sampai DEC-27 di `14_perubahan-model-marketplace.md`: kategori Moving, Food run, dan Laundry dihapus, pemesanan berubah menjadi model marketplace (client memilih helper langsung, tanpa broadcast), dan client bisa chat helper sebelum memesan. Akibatnya sebelas FR dinyatakan DEPRECATED (FR-ORD-003, 003B, 003C, 004, 005, 006B, 014, 019, 020, FR-CHT-001, FR-CHT-004), tujuh FR baru ditambahkan (FR-HLP-011, FR-ORD-024, FR-CHT-006 sampai FR-CHT-010), satu NFR baru (NFR-PERF-04), dan dua puluh lima FR direvisi. Penanda "turunan SA" berarti detail itu ditambahkan SA saat menerapkan keputusan dan masih menunggu konfirmasi Backend.

Perubahan v0.2 (28 September 2026) mengikuti DEC-14 sampai DEC-24 di `10_notulen-sinkronisasi-erd-final.md`.

Keterangan kolom sumber: `PROTO` berarti terlihat langsung di prototipe UI, `TURUNAN` berarti konsekuensi logis dari fitur yang terlihat atau dari keputusan tim, `ASUMSI` berarti belum ada buktinya dan wajib divalidasi ke tim atau Mentor sebelum naik ke `CONFIRMED`.

Status `CONFIRMED` di sini berarti isi requirement sudah disetujui pihak berwenang, belum berarti sudah dibangun.

Prioritas memakai MoSCoW. Untuk durasi lab, target realistis adalah menyelesaikan seluruh `Must Have` dan sebagian `Should Have`.

## 1. Modul AUTH, akun dan autentikasi

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-AUTH-001 | Sistem harus menampilkan form registrasi berisi nama lengkap, nomor telepon, email, dan kata sandi | Must | TURUNAN | DRAFT |
| FR-AUTH-002 | Sistem harus memverifikasi nomor telepon melalui kode OTP enam digit yang kedaluwarsa dalam 300 detik | Must | ASUMSI | DRAFT |
| FR-AUTH-003 | Sistem harus menolak registrasi jika nomor telepon atau email sudah terdaftar dan menampilkan pesan yang membedakan keduanya | Must | TURUNAN | DRAFT |
| FR-AUTH-004 | Sistem harus menerima login dengan kombinasi nomor telepon atau email dan kata sandi | Must | TURUNAN | DRAFT |
| FR-AUTH-005 | Sistem harus mengunci percobaan login selama 900 detik setelah lima kegagalan berturut turut dari satu akun | Should | ASUMSI | DRAFT |
| FR-AUTH-006 | Sistem harus menyediakan pemulihan kata sandi melalui OTP ke nomor telepon terdaftar | Should | ASUMSI | DRAFT |
| FR-AUTH-007 | Sistem harus menjaga sesi login dengan access token berumur 60 menit dan refresh token berumur 30 hari | Must | TURUNAN | DRAFT |
| FR-AUTH-008 | Sistem harus menampilkan tiga layar onboarding pengenalan sebelum registrasi dengan opsi lewati di setiap layar | Must | PROTO | DRAFT |

## 2. Modul VER, verifikasi identitas

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-VER-001 | Sistem harus menampilkan lencana ID verified pada profil pengguna yang identitasnya sudah lolos verifikasi | Must | PROTO | DRAFT |
| FR-VER-002 | Sistem harus meminta unggahan foto kartu identitas dan swafoto pemegang kartu saat pengguna mendaftar menjadi helper | Must | TURUNAN | DRAFT |
| FR-VER-003 | Sistem harus menyimpan status verifikasi dengan nilai belum diajukan, sedang ditinjau, disetujui, atau ditolak beserta alasan penolakan | Must | TURUNAN | DRAFT |
| FR-VER-004 | Sistem harus melarang akun yang belum berstatus disetujui untuk tampil di daftar helper maupun menerima permintaan pesanan apa pun | Must | TURUNAN, DIPERJELAS 19 Okt 2026 (DEC-26) | DRAFT |
| FR-VER-005 | Sistem harus menampilkan lencana helper terverifikasi pada kartu helper di daftar pencarian | Must | PROTO | DRAFT |
| FR-VER-006 | ~~Sistem harus menahan pengajuan ulang verifikasi selama 2 menit setelah pengajuan sebelumnya ditolak~~ | Must | TURUNAN | DEPRECATED, dihapus 28 Sep 2026 (DEC-14 nomor 7). Digantikan FR-VER-007. Anti spam ditangani rate limit API, bukan aturan bisnis |
| FR-VER-007 | Sistem harus mengizinkan pengguna mengajukan ulang verifikasi segera setelah pengajuan sebelumnya ditolak, dan menolak pengajuan baru selama pengguna masih memiliki satu pengajuan berstatus sedang ditinjau | Must | TURUNAN, BARU 28 Sep 2026 (DEC-14 nomor 7) | CONFIRMED |

## 3. Modul PRF, profil dan alamat

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-PRF-001 | Sistem harus menampilkan halaman profil berisi nama, kota, tanggal bergabung, dan status verifikasi | Must | PROTO | DRAFT |
| FR-PRF-002 | Sistem harus mengizinkan pengguna menyimpan maksimal sepuluh alamat dengan label, koordinat, dan catatan patokan | Must | TURUNAN, batas sepuluh ditetapkan sebagai business rule yang disengaja (DEC-14 nomor 9) | CONFIRMED |
| FR-PRF-003 | Sistem harus menandai satu alamat sebagai alamat utama yang terpilih otomatis saat membuat pesanan | Should | TURUNAN | DRAFT |
| FR-PRF-004 | Sistem harus menampilkan lokasi aktif pengguna pada bagian atas beranda | Must | PROTO | DRAFT |
| FR-PRF-005 | Sistem harus menyediakan menu ubah profil untuk nama, foto, dan nomor telepon dengan verifikasi OTP ulang saat nomor diubah | Should | TURUNAN | DRAFT |

## 4. Modul HLP, pencarian dan pendaftaran helper

Sejak 19 Oktober 2026 (DEC-26) modul ini menjadi pintu masuk pemesanan. Client tidak lagi menyiarkan pesanan, tetapi memilih helper yang relevan dari daftar di modul ini, lalu chat (modul CHT) atau langsung mengirim permintaan pesanan (modul ORD).

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-HLP-001 | Sistem harus menampilkan tiga kategori layanan pada beranda yaitu Delivery, Personal, dan Household, dan membuka daftar helper yang tersaring kategori tersebut saat salah satu kategori dipilih | Must | PROTO, DIREVISI 19 Okt 2026 (DEC-25, DEC-26). Prototipe masih menampilkan enam kategori | CONFIRMED |
| FR-HLP-002 | Sistem harus menampilkan daftar helper terverifikasi yang mengaktifkan ketersediaan beserta nama, rating rata rata, jumlah ulasan, jarak dalam kilometer, tarif per jam, dan tag keahlian. Tarif per jam hanya informasi profil dan tidak dipakai menghitung harga pesanan, harga final tetap berasal dari quote helper | Must | PROTO, DIPERJELAS 28 Sep 2026 (DEC-14 nomor 4) dan 19 Okt 2026 (DEC-26) | DRAFT |
| FR-HLP-003 | Sistem harus menyediakan penyaringan daftar helper berdasarkan kategori layanan | Must | PROTO | DRAFT |
| FR-HLP-004 | Sistem harus menyediakan pengurutan daftar helper berdasarkan jarak terdekat, rating tertinggi, dan tarif terendah | Must | PROTO | DRAFT |
| FR-HLP-005 | Sistem harus membatasi daftar helper pada radius maksimal 10 kilometer dari lokasi aktif client | Should | ASUMSI, DIPERJELAS 19 Okt 2026: titik acuan radius pindah dari lokasi pesanan ke lokasi aktif client karena daftar helper dibuka sebelum pesanan dibuat | DRAFT |
| FR-HLP-006 | Sistem harus menampilkan jumlah helper yang tersedia di sekitar lokasi pengguna | Must | PROTO | DRAFT |
| FR-HLP-007 | Sistem harus menyediakan menu daftar menjadi helper pada halaman profil | Must | PROTO | DRAFT |
| FR-HLP-008 | Sistem harus meminta calon helper memilih minimal satu dan maksimal tiga kategori layanan yang dikuasai | Must | TURUNAN, DIREVISI 19 Okt 2026 (DEC-25), batas atas turun dari enam menjadi tiga | DRAFT |
| FR-HLP-009 | Sistem harus menyediakan sakelar ketersediaan bagi helper untuk berhenti tampil di daftar helper dan berhenti menerima permintaan pesanan baru. Permintaan yang sudah masuk sebelum sakelar dimatikan tetap bisa dijawab | Must | TURUNAN, DIPERJELAS 19 Okt 2026 (DEC-26). Kalimat kedua adalah turunan SA | DRAFT |
| FR-HLP-010 | Sistem harus menampilkan halaman detail helper berisi profil, kategori yang dikuasai, ulasan, jumlah pekerjaan selesai, tombol Chat, dan tombol Pesan | Must | TURUNAN, DIREVISI 19 Okt 2026 (DEC-26, DEC-27), prioritas naik dari Should karena halaman ini menjadi titik awal chat dan pemesanan | DRAFT |
| FR-HLP-011 | Sistem harus menampilkan label sedang sibuk dan menonaktifkan tombol Pesan pada kartu dan halaman detail helper yang memegang pesanan berstatus accepted sampai awaiting_confirmation, tanpa menonaktifkan tombol Chat | Should | TURUNAN, BARU 19 Okt 2026 (DEC-26), turunan SA | DRAFT |

## 5. Modul ORD, siklus hidup pesanan

Modul ini adalah inti sistem. Detail transisi status ada di `04_dual-role-transaction-analysis.md`. Sejak 19 Oktober 2026 hanya ada satu jalur pemesanan: client mengirim permintaan ke helper yang dia pilih, helper memberi quote, client menyetujui, lalu dana ditahan (DEC-26, mengikuti aturan quote DEC-21).

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-ORD-001 | Sistem harus menyediakan form permintaan pesanan yang dibuka dari halaman detail helper atau ruang chat dengan helper tujuan sudah terisi, berisi kategori yang dikuasai helper tersebut, deskripsi tugas, alamat penjemputan, alamat tujuan atau lokasi pengerjaan, waktu pelaksanaan, dan harga estimasi upah jasa dari client. Field alamat yang tampil mengikuti konfigurasi kategori | Must | TURUNAN, DIREVISI 28 Sep 2026 (DEC-16) dan 19 Okt 2026 (DEC-25, DEC-26). Field batas talangan dihapus | DRAFT |
| FR-ORD-002 | Sistem harus menghitung rentang biaya acuan sebelum permintaan dikirim, terdiri atas tarif dasar, biaya jarak, biaya layanan, dan potongan voucher, sebagai panduan bagi client menentukan harga estimasi dan bagi helper menyusun quote. Biaya layanan bernilai flat sama dengan tarif dasar kategori, tidak dikalikan durasi | Must | TURUNAN, DIKONFIRMASI 28 Sep 2026 (DEC-14 nomor 4), istilah tawaran diganti quote 19 Okt 2026 | CONFIRMED |
| FR-ORD-003 | ~~Sistem harus menyiarkan pesanan broadcast ke helper yang memenuhi syarat kategori, radius, dan batas talangan, dan mengizinkan setiap helper yang berminat mengajukan tawaran upah jasa masing masing berbeda dari harga estimasi client~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Broadcast tidak ada lagi, digantikan FR-ORD-003D |
| FR-ORD-003B | ~~Sistem harus menampilkan seluruh tawaran harga yang masuk kepada client, masing masing beserta profil helper, rating, dan nominal yang diajukan, agar client dapat membandingkan sebelum memilih~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Client membandingkan helper di daftar helper (FR-HLP-002, FR-HLP-004) dan lewat chat (FR-CHT-006) |
| FR-ORD-003C | ~~Sistem harus mengizinkan client memilih satu tawaran dari daftar yang masuk, dan pilihan ini yang memicu penahanan dana serta penolakan otomatis terhadap tawaran lain~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Penahanan dana dipicu persetujuan quote di FR-ORD-003D |
| FR-ORD-003D | Sistem harus mengirim permintaan pesanan hanya kepada helper yang dipilih client. Helper memberi quote upah jasa dalam 300 detik atau menolak, client menyetujui quote dalam 300 detik sejak quote masuk, dan dana ditahan saat client menyetujui. Client tidak dapat menawar balik. Kalau helper menolak atau batas waktu lewat, pesanan kedaluwarsa dan client ditawari tombol Pilih helper lain yang mengirim data pesanan yang sama ke helper lain sebagai pesanan baru | Must | TURUNAN, DITULIS ULANG 28 Sep 2026 (DEC-14 nomor 4, DEC-21) dan 19 Okt 2026 (DEC-26). Kini satu satunya jalur pemesanan, prioritas naik dari Should. Tombol siarkan diganti Pilih helper lain | CONFIRMED |
| FR-ORD-004 | ~~Sistem harus menutup jendela penawaran 300 detik setelah pesanan disiarkan, dan mengizinkan client memilih dari tawaran yang sudah masuk meskipun jendela belum ditutup penuh jika sudah ada minimal satu tawaran~~ | Must | ASUMSI | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Batas waktu quote diatur FR-ORD-003D |
| FR-ORD-005 | ~~Sistem harus mengubah status pesanan broadcast menjadi kedaluwarsa jika tidak ada satu pun tawaran masuk sampai jendela penawaran berakhir, dan memberi tahu client dengan saran menaikkan harga estimasi~~ | Must | ASUMSI | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Kedaluwarsa permintaan diatur FR-ORD-003D |
| FR-ORD-006 | Sistem harus menampilkan kepada helper upah bersih yang akan diterima berdasarkan quote yang dia isi, yaitu upah dikurangi komisi platform 10 persen, dihitung sebelum quote dikirim | Must | TURUNAN, DIREVISI 19 Agu 2026 (DEC-05), 28 Sep 2026 (DEC-17), dan 19 Okt 2026 (DEC-25, DEC-26). Kalimat soal talangan dihapus | DRAFT |
| FR-ORD-006B | ~~Sistem harus meminta konfirmasi ulang dari helper terpilih dalam 60 detik setelah client memilih tawarannya, sebelum dana ditahan, khusus untuk pesanan broadcast~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-26). Konfirmasi 60 detik hanya ada pada broadcast |
| FR-ORD-007 | Sistem harus menampilkan status pesanan berjalan pada beranda client dalam bentuk kartu ringkas berisi nama helper, aktivitas, estimasi waktu, dan nomor pesanan | Must | PROTO | DRAFT |
| FR-ORD-008 | Sistem harus menampilkan halaman pelacakan berisi peta, posisi helper, estimasi waktu tiba, tahapan progres, identitas helper, nomor pesanan, dan total biaya | Must | PROTO | DRAFT |
| FR-ORD-009 | Sistem harus menampilkan tahapan progres pesanan minimal empat tahap yaitu diterima, menuju lokasi, sedang dikerjakan, dan selesai | Must | PROTO | DRAFT |
| FR-ORD-010 | Sistem harus menyediakan tombol selesaikan pesanan bagi client sebagai konfirmasi pekerjaan diterima | Must | PROTO | DRAFT |
| FR-ORD-011 | Sistem harus menerapkan biaya pembatalan oleh client sesuai status: gratis pada awaiting_quote dan accepted kurang dari 120 detik. Rp 5.000 pada accepted 120 detik atau lebih. 25 persen dari harga jasa pada on_the_way dan arrived. Client tidak dapat membatalkan sepihak mulai in_progress. Kompensasi masuk utuh ke helper tanpa komisi platform, dan client melihat rincian potongan sebelum mengonfirmasi batal | Must | PROTO, DIREVISI 28 Sep 2026 (DEC-19) dan 19 Okt 2026 (status searching dan pending_confirmation hilang, DEC-26) | CONFIRMED |
| FR-ORD-012 | Sistem harus mengonfirmasi pesanan selesai secara otomatis dalam 24 jam setelah helper menandai pekerjaan selesai jika client tidak merespons | Must | ASUMSI | DRAFT |
| FR-ORD-013 | Sistem harus mewajibkan helper mengunggah foto bukti penyelesaian sebelum menandai selesai untuk kategori yang konfigurasinya mewajibkan foto bukti, pada versi ini kategori Delivery | Should | TURUNAN, DIPERJELAS 28 Sep 2026 (foto disimpan di ORDER_ATTACHMENT, DEC-16), DIREVISI 19 Okt 2026 (Food run dan Laundry dihapus, DEC-25) | DRAFT |
| FR-ORD-014 | ~~Sistem harus menerima struk belanja dari helper pada kategori Food run dan Laundry, mengakui struk dalam batas talangan, dan meminta persetujuan client untuk struk yang melewati batas~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-25). Tidak ada lagi kategori bertalangan |
| FR-ORD-015 | Sistem harus menyimpan riwayat pesanan client dan helper beserta status akhir dan rincian biaya | Must | PROTO | DRAFT |
| FR-ORD-016 | Sistem harus melarang seorang pengguna memesan dirinya sendiri sebagai helper, dan tidak menampilkan profil helper miliknya di daftar helper yang dia lihat sebagai client | Must | TURUNAN, DIPERJELAS 19 Okt 2026 (DEC-26). Bagian menyembunyikan profil sendiri adalah turunan SA | DRAFT |
| FR-ORD-017 | Sistem harus membatasi jumlah pesanan berstatus aktif menjadi maksimal satu untuk setiap helper pada satu waktu | Should | ASUMSI | DRAFT |
| FR-ORD-018 | Sistem harus menyediakan kanal sengketa bagi client dan helper dalam 24 jam setelah helper menandai pesanan selesai, khusus untuk pesanan berkategori Delivery. Kategori lain tidak memiliki jalur sengketa formal pada versi ini | Should | TURUNAN, DIREVISI 28 Agu 2026 (DEC-10). Catatan Food run gugur 19 Okt 2026 (DEC-25) | CONFIRMED |
| FR-ORD-019 | ~~Sistem harus menyimpan setiap struk dan penyesuaian talangan sebagai riwayat terpisah, dan menolak struk baru selama pesanan masih memiliki satu penyesuaian berstatus menunggu persetujuan~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-25) |
| FR-ORD-020 | ~~Sistem harus menyembunyikan pesanan bertalangan dari helper yang batas talangan maksimalnya lebih kecil dari batas talangan pesanan~~ | Must | TURUNAN | DEPRECATED, dihapus 19 Okt 2026 (DEC-25) |
| FR-ORD-021 | Sistem harus mewajibkan helper mengunggah foto kondisi barang saat penjemputan sebelum status pesanan Delivery berubah dari arrived ke in_progress | Must | TURUNAN, BARU 28 Sep 2026 (DEC-24) | CONFIRMED |
| FR-ORD-022 | Sistem harus menampilkan daftar pekerjaan yang tidak termasuk kategori Personal pada layar pembuatan pesanan Personal, yaitu jasa yang butuh sertifikasi atau izin, mengangkut penumpang, membelikan barang atau menangani uang tunai atas nama client, barang atau aktivitas melanggar hukum, dan pekerjaan berbahaya atau berisiko tinggi terhadap keselamatan | Should | TURUNAN, BARU 28 Sep 2026 (DEC-18), DIREVISI 19 Okt 2026: larangan ketiga diperluas karena talangan tidak ada lagi, turunan SA | REVIEW |
| FR-ORD-023 | Sistem harus mengizinkan helper membatalkan pesanan berstatus accepted sampai in_progress dengan mengembalikan dana jasa penuh ke client, tidak membayar upah, dan menambah catatan pembatalan helper | Must | TURUNAN, BARU 28 Sep 2026 (DEC-19), DIREVISI 19 Okt 2026 (klausul talangan dihapus, DEC-25) | CONFIRMED |
| FR-ORD-024 | Sistem harus mengubah seluruh permintaan pesanan lain yang ditujukan ke seorang helper dan masih berstatus awaiting_quote menjadi expired begitu helper itu memiliki pesanan berstatus accepted, lalu memberi tahu client masing masing dengan tombol Pilih helper lain | Should | TURUNAN, BARU 19 Okt 2026 (DEC-26), turunan SA untuk menjaga INV-06 | DRAFT |

## 6. Modul WLT, dompet dan pembayaran

Catatan lingkup, hasil keputusan tim 19 Agustus 2026, `DEC-04`. Seluruh transaksi pada modul ini berjalan sebagai saldo simulasi, tidak ada uang sungguhan yang berpindah. Integrasi payment gateway sungguhan seperti Midtrans atau Xendit dicatat sebagai arah pengembangan lanjutan, tidak masuk lingkup pengerjaan lab saat ini, dan disebutkan eksplisit di bagian batasan SRS supaya tidak jadi ekspektasi keliru saat presentasi.

Catatan komisi, `DEC-08`, dikonfirmasi 28 Agustus 2026, komisi platform ditetapkan 10 persen, dipotong dari upah jasa helper, bukan ditambahkan ke tagihan client. Voucher dibiayai dari komisi platform, bukan dari upah helper (DEC-15). Sejak 19 Oktober 2026 (DEC-25) tidak ada lagi uang talangan, sehingga dana yang ditahan selalu sama dengan harga jasa setelah voucher. Rumus lengkap ada di `05_erd-draft.md` bagian 2.2.

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-WLT-001 | Sistem harus menampilkan saldo TD-Wallet pada halaman profil beserta tombol isi saldo dan riwayat | Must | PROTO | DRAFT |
| FR-WLT-002 | Sistem harus menahan dana sebesar harga jasa setelah voucher pada saat client menyetujui quote, dan menolak penahanan jika saldo client tidak mencukupi | Must | TURUNAN, DIREVISI 28 Sep 2026 (DEC-15, DEC-21) dan 19 Okt 2026 (DEC-25, DEC-26) | CONFIRMED |
| FR-WLT-003 | Sistem harus menyelesaikan dana tertahan paling lambat lima detik setelah pesanan berstatus selesai: melepas upah bersih ke helper dan mencatat komisi bersih platform | Must | TURUNAN, DIREVISI 28 Sep 2026 (DEC-15) dan 19 Okt 2026 (bagian talangan dihapus, DEC-25) | CONFIRMED |
| FR-WLT-004 | Sistem harus menampilkan rincian pada setiap transaksi helper, terdiri atas upah jasa, potongan platform, dan upah bersih sebagai baris terpisah | Must | TURUNAN, DIPERJELAS 28 Sep 2026, DIREVISI 19 Okt 2026 (baris penggantian talangan dihapus) | DRAFT |
| FR-WLT-005 | Sistem harus menampilkan riwayat mutasi saldo berisi tanggal, jenis transaksi, nominal, nomor pesanan terkait, dan saldo akhir | Must | PROTO | DRAFT |
| FR-WLT-006 | Sistem harus menyediakan pengisian saldo dengan nominal minimal sepuluh ribu rupiah | Must | PROTO | DRAFT |
| FR-WLT-007 | Sistem harus menyediakan penarikan saldo helper dengan jadwal pencairan mingguan | Should | PROTO | DRAFT |
| FR-WLT-008 | Sistem harus mencegah saldo bernilai negatif pada kondisi apa pun | Must | TURUNAN | DRAFT |
| FR-WLT-009 | Sistem harus menolak permintaan transaksi berulang dengan idempotency key yang sama dan mengembalikan hasil transaksi pertama | Must | TURUNAN | DRAFT |
| FR-WLT-010 | Sistem harus menampilkan voucher yang dimiliki pengguna beserta syarat dan tanggal kedaluwarsa | Should | PROTO | DRAFT |
| FR-WLT-011 | Sistem harus memeriksa syarat voucher terhadap quote yang disetujui client, bukan harga estimasi. Jika tidak memenuhi syarat, sistem tetap memproses persetujuan tanpa voucher, memberi tahu client, dan menyimpan voucher untuk pesanan lain | Should | PROTO, DIREVISI 28 Sep 2026 (DEC-23) dan 19 Okt 2026 (tawaran dihapus, DEC-26) | CONFIRMED |
| FR-WLT-012 | Sistem harus membatasi pemakaian voucher maksimal satu voucher per pesanan | Should | TURUNAN, BARU 28 Sep 2026 (DEC-14 nomor 6). Voucher bertumpuk masuk backlog | CONFIRMED |
| FR-WLT-013 | Sistem harus membatasi potongan voucher pada satu pesanan agar tidak melebihi komisi platform 10 persen dari harga jasa, dan tidak pernah mengurangi upah bersih helper | Must | TURUNAN, BARU 28 Sep 2026 (DEC-14 nomor 5, DEC-15) | CONFIRMED |
| FR-WLT-014 | Sistem harus mengembalikan voucher ke client ketika pesanan batal tanpa biaya pembatalan, dan menghanguskan voucher ketika client dikenai biaya pembatalan | Should | TURUNAN, BARU 28 Sep 2026 (DEC-19) | CONFIRMED |

## 7. Modul CHT, percakapan

Direvisi 19 Oktober 2026 mengikuti DEC-27. Chat tidak lagi terikat pada satu pesanan. Client bisa menghubungi helper sebelum memesan, misalnya untuk menanyakan ketersediaan atau menyepakati kisaran harga, dan percakapan yang sama berlanjut saat pesanan berjalan.

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-CHT-001 | ~~Sistem harus menyediakan ruang percakapan antara client dan helper yang terikat pada satu pesanan, dibuka saat pesanan berstatus accepted. Tidak ada percakapan sebelum pesanan terbentuk~~ | Must | PROTO | DEPRECATED, dihapus 19 Okt 2026 (DEC-27, menggantikan DEC-14 nomor 10). Digantikan FR-CHT-006 dan FR-CHT-007 |
| FR-CHT-002 | Sistem harus menampilkan daftar percakapan berisi nama lawan bicara, cuplikan pesan terakhir, waktu, dan penanda pesan belum dibaca | Must | PROTO | DRAFT |
| FR-CHT-003 | Sistem harus menyediakan balasan cepat berupa frasa siap pakai untuk mempercepat komunikasi | Should | PROTO | DRAFT |
| FR-CHT-004 | ~~Sistem harus menutup ruang percakapan tujuh hari setelah pesanan selesai dan menjadikannya hanya bisa dibaca~~ | Could | ASUMSI | DEPRECATED, dihapus 19 Okt 2026 (DEC-27). Ruang percakapan kini milik pasangan client dan helper dan tetap terbuka lintas pesanan |
| FR-CHT-005 | Sistem harus menyediakan tombol panggilan telepon ke helper pada halaman pelacakan | Should | PROTO | DRAFT |
| FR-CHT-006 | Sistem harus mengizinkan client memulai percakapan dengan helper terverifikasi dari halaman detail helper sebelum membuat pesanan, dan mengizinkan helper membalasnya | Must | TURUNAN, BARU 19 Okt 2026 (DEC-27) | CONFIRMED |
| FR-CHT-007 | Sistem harus menyimpan satu ruang percakapan untuk setiap pasangan client dan helper, memakai ruang yang sama sebelum, selama, dan sesudah pesanan, membuat ruang itu otomatis saat client mengirim permintaan pesanan kalau belum ada, dan hanya mengizinkan client membuka ruang baru | Must | TURUNAN, BARU 19 Okt 2026 (DEC-27), turunan SA | REVIEW |
| FR-CHT-008 | Sistem harus menampilkan tombol Pesan helper ini di ruang percakapan, yang membuka form permintaan pesanan dengan helper tersebut sudah terisi, dan menonaktifkannya kalau helper sedang tidak dapat dipesan | Should | TURUNAN, BARU 19 Okt 2026 (DEC-26, DEC-27), turunan SA | DRAFT |
| FR-CHT-009 | Sistem harus menampilkan pemberitahuan tetap di ruang percakapan bahwa pembayaran di luar TD-Wallet tidak dilindungi penahanan dana dan tidak dapat dibantu lewat kanal sengketa, dan tetap menyamarkan nomor telepon kedua pihak sesuai NFR-SEC-04 | Should | TURUNAN, BARU 19 Okt 2026 (DEC-27), turunan SA untuk risiko RSK-05 | DRAFT |
| FR-CHT-010 | Sistem harus membatasi setiap client membuka maksimal 20 ruang percakapan baru per hari | Could | ASUMSI, BARU 19 Okt 2026, angka 20 belum divalidasi tim | DRAFT |

## 8. Modul RTG, penilaian satu arah

Catatan revisi 28 Agustus 2026, `DEC-09`. Modul ini semula bernama "penilaian dua arah". Hasil diskusi dengan UI/UX, penilaian balik dari Helper ke Client dihapus dari lingkup, bukan sekadar digabung ke layar lain. Penilaian sekarang satu arah, hanya Client menilai Helper, dan langsung terlihat tanpa mekanisme tunda atau sembunyikan.

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-RTG-001 | Sistem harus meminta client memberi penilaian satu sampai lima bintang dan ulasan opsional setelah pesanan selesai | Must | TURUNAN | DRAFT |
| FR-RTG-002 | ~~Sistem harus meminta helper memberi penilaian terhadap client dan menyembunyikan kedua penilaian sampai keduanya mengisi atau sampai lewat 72 jam~~ | Must | TURUNAN | DEPRECATED, dihapus 28 Agu 2026 (DEC-09). Digantikan oleh model satu arah, tidak ada requirement pengganti karena memang tidak ada lagi penilaian dari sisi Helper |
| FR-RTG-003 | Sistem harus menghitung rating rata rata helper dari seluruh penilaian client dan menampilkannya dengan satu angka desimal, ditampilkan langsung tanpa masa tunda | Must | PROTO, DIREVISI 28 Agu 2026 (DEC-09) | DRAFT |
| FR-RTG-004 | Sistem harus menolak penilaian pada pesanan yang berstatus batal | Must | TURUNAN | DRAFT |

## 9. Modul NOT, notifikasi

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-NOT-001 | Sistem harus menampilkan lonceng notifikasi dengan penanda jika ada notifikasi belum dibaca | Must | PROTO | DRAFT |
| FR-NOT-002 | Sistem harus mengirim notifikasi pada setiap perubahan status pesanan kepada pihak yang berkepentingan | Must | TURUNAN | DRAFT |
| FR-NOT-003 | Sistem harus mengirim notifikasi permintaan pesanan baru kepada helper yang dipilih client, notifikasi quote masuk kepada client, dan notifikasi pesan chat baru kepada penerimanya | Must | TURUNAN, DIREVISI 19 Okt 2026 (DEC-26, DEC-27) | DRAFT |
| FR-NOT-004 | Sistem harus mengirim notifikasi mutasi saldo untuk setiap penahanan, pelepasan, dan pengembalian dana | Should | TURUNAN | DRAFT |
| FR-NOT-005 | Sistem harus menyediakan halaman daftar notifikasi dengan penyimpanan riwayat 30 hari terakhir | Should | PROTO | DRAFT |

## 9B. Modul ADM, panel admin

Modul baru, ditambahkan 28 Agustus 2026 mengikuti `DEC-07` yang sudah final, Mentor mengakses lewat panel admin sungguhan di dalam sistem, bukan proses manual. Ini menambah lingkup baru yang sebelumnya tidak ada di manapun pada workspace, jadi seluruh isi modul ini berstatus `DRAFT` dan `ASUMSI` sampai ditinjau ulang bersama BE, karena rancangannya baru pertama kali ditulis di sini.

Cakupan modul ini mencakup verifikasi identitas Helper, penyelesaian sengketa, ringkasan agregat komisi platform, suspend akun (`FR-ADM-009`, `FR-ADM-010`), pembatalan pesanan oleh admin (`FR-ADM-011`), dan pengelolaan voucher (`FR-ADM-012`). Aturan suspend dan pembatalan admin disesuaikan 19 Oktober 2026 karena status `pending_confirmation` dan tawaran broadcast sudah tidak ada (DEC-26).

| Kode | Functional Requirement | Prioritas | Sumber | Status |
| --- | --- | --- | --- | --- |
| FR-ADM-001 | Sistem harus menyediakan login terpisah untuk akun admin, tidak memakai jalur registrasi dan login yang sama dengan Client dan Helper | Must | ASUMSI | DRAFT |
| FR-ADM-002 | Sistem harus menolak akses ke seluruh endpoint dan halaman admin bagi akun yang tidak bertanda admin, termasuk akun Client dan Helper yang mencoba mengakses langsung lewat URL | Must | TURUNAN | DRAFT |
| FR-ADM-003 | Sistem harus menampilkan daftar pengajuan verifikasi identitas Helper yang berstatus menunggu peninjauan, beserta foto KTP dan swafoto yang diunggah | Must | TURUNAN | DRAFT |
| FR-ADM-004 | Sistem harus mengizinkan admin menyetujui atau menolak pengajuan verifikasi, dan mewajibkan alasan penolakan diisi jika ditolak | Must | TURUNAN | DRAFT |
| FR-ADM-005 | Sistem harus menampilkan daftar sengketa yang masuk beserta detail pesanan, bukti terkait, dan riwayat percakapan yang terkait pesanan itu, dibatasi hanya pesanan berkategori Delivery sesuai FR-ORD-018 | Must | TURUNAN, DIPERJELAS 19 Okt 2026 (DEC-27), bagian riwayat percakapan adalah turunan SA | DRAFT |
| FR-ADM-006 | Sistem harus mengizinkan admin memutuskan hasil sengketa dengan tiga pilihan dan catatan keputusan wajib diisi. Berpihak helper: settlement normal dengan komisi. Berpihak client: seluruh dana tertahan kembali ke client. Dibagi: dana tertahan dibagi 50:50 tanpa komisi platform, tanpa isian persentase oleh admin | Must | TURUNAN, DIREVISI 28 Sep 2026 (DEC-24) | CONFIRMED |
| FR-ADM-007 | Sistem harus menampilkan ringkasan agregat komisi platform yang terkumpul kepada admin, dapat disaring berdasarkan rentang tanggal | Should | TURUNAN | DRAFT |
| FR-ADM-008 | Sistem harus mencatat setiap tindakan admin (keputusan verifikasi, keputusan sengketa, suspend, reaktivasi, pembatalan pesanan, dan pengelolaan voucher) beserta identitas admin dan waktu tindakan | Must | TURUNAN, DIPERLUAS 28 Sep 2026 | DRAFT |
| FR-ADM-009 | Sistem harus mengizinkan admin menonaktifkan (suspend) akun Client atau Helper dengan alasan wajib diisi, menolak suspend jika akun masih memiliki pesanan berstatus accepted sampai awaiting_confirmation atau disputed, dan saat suspend berhasil membatalkan seluruh pesanan berstatus awaiting_quote yang dibuat akun itu sebagai client maupun yang ditujukan kepadanya sebagai helper, serta mencabut seluruh sesi login | Must | TURUNAN, BARU 6 Sep 2026, DIREVISI 28 Sep 2026 (DEC-14 nomor 12, DEC-22) dan 19 Okt 2026 (DEC-26) | CONFIRMED |
| FR-ADM-010 | Sistem harus mengizinkan admin mengaktifkan kembali akun yang di-suspend | Must | TURUNAN, BARU 6 Sep 2026 hasil sesi BE | DRAFT |
| FR-ADM-011 | Sistem harus mengizinkan admin membatalkan pesanan berstatus accepted sampai awaiting_confirmation dengan alasan wajib diisi. Dana jasa kembali penuh ke client, helper tidak menerima kompensasi, dan status akhir menjadi cancelled_by_admin. Pesanan berstatus disputed tidak dapat dibatalkan admin | Must | TURUNAN, BARU 28 Sep 2026 (DEC-22), DIREVISI 19 Okt 2026 (status pending_confirmation dan klausul talangan dihapus, DEC-25, DEC-26) | CONFIRMED |
| FR-ADM-012 | Sistem harus mengizinkan admin membuat, mengubah, dan menonaktifkan voucher, dengan validasi keras: voucher nominal tetap ditolak jika minimal pesanan kurang dari sepuluh kali nilai diskon, dan voucher persentase ditolak jika melebihi 10 persen | Should | TURUNAN, BARU 28 Sep 2026 (DEC-23). Untuk demo, voucher boleh berasal dari data seed | CONFIRMED |

## 10. Non-Functional Requirement

Kategori mengacu pada ISO 25010. Setiap NFR ditulis dengan angka supaya QA bisa mengujinya. Perubahan 19 Oktober 2026: NFR-PERF-04 baru untuk percakapan, NFR-SEC-04 dan NFR-USE-01 disesuaikan dengan alur pemesanan baru.

| Kode | Quality Requirement | Quality Factor | Cara pengukuran |
| --- | --- | --- | --- |
| NFR-PERF-01 | Waktu respons API untuk operasi baca tidak lebih dari 1500 milidetik pada persentil 95 dengan jaringan 4G | Performance Efficiency | Uji beban dengan 100 pengguna serentak |
| NFR-PERF-02 | Waktu buka dingin aplikasi tidak lebih dari 3 detik pada perangkat Android dengan RAM 4 GB | Performance Efficiency | Pengukuran manual pada perangkat acuan sebanyak sepuluh kali |
| NFR-PERF-03 | Perubahan status pesanan tampil di perangkat lawan bicara dalam waktu tidak lebih dari 5 detik | Performance Efficiency | Uji dua perangkat berdampingan |
| NFR-PERF-04 | Pesan chat tampil di perangkat lawan bicara dalam waktu tidak lebih dari 5 detik ketika kedua perangkat terhubung internet | Performance Efficiency | Uji dua perangkat berdampingan, sepuluh pesan berturut turut. Baru 19 Okt 2026 |
| NFR-SEC-01 | Kata sandi disimpan dalam bentuk hash dengan algoritma bcrypt faktor kerja minimal 12 | Security | Inspeksi basis data |
| NFR-SEC-02 | Seluruh komunikasi antara aplikasi dan server memakai HTTPS dengan TLS 1.2 atau lebih baru | Security | Inspeksi konfigurasi server |
| NFR-SEC-03 | Foto kartu identitas disimpan di bucket privat (Cloudflare R2) dan hanya dapat diakses lewat signed URL berumur pendek yang dibuat backend untuk admin operasional, tidak pernah dikembalikan pada endpoint publik. Tidak ada enkripsi at-rest tambahan (DEC-14 nomor 8) | Security | Uji akses tidak sah |
| NFR-SEC-04 | Nomor telepon disimpan tanpa enkripsi kolom, akses basis data dibatasi pada akun layanan backend, dan aplikasi menampilkan nomor lawan bicara tersamar (contoh 0812xxxx9012) di daftar helper, di ruang percakapan, dan di halaman pesanan, kecuali pada pesanan yang sedang berjalan (DEC-14 nomor 11, diperjelas 19 Okt 2026) | Security | Uji tampilan dan peninjauan hak akses basis data |
| NFR-REL-01 | Ketersediaan layanan minimal 99 persen dalam periode pengujian | Reliability | Pemantauan uptime |
| NFR-REL-02 | Tidak boleh ada selisih antara total saldo pengguna dan total mutasi ledger pada rekonsiliasi harian | Reliability | Skrip rekonsiliasi otomatis |
| NFR-REL-03 | Tingkat sesi bebas gagal aplikasi minimal 99 persen | Reliability | Laporan crash reporting |
| NFR-USE-01 | Client dapat mengirim permintaan pesanan dari beranda dalam maksimal lima ketukan, dihitung dari memilih kategori sampai menekan kirim, tidak termasuk ketukan untuk mengetik deskripsi tugas | Usability | Penelusuran alur bersama UI/UX. Diperjelas 19 Okt 2026 mengikuti alur pilih helper |
| NFR-USE-02 | Setiap pesan kesalahan menyebutkan penyebab dan tindakan yang bisa dilakukan pengguna, bukan kode teknis | Usability | Peninjauan seluruh pesan kesalahan |
| NFR-USE-03 | Seluruh teks antarmuka memakai bahasa Indonesia yang konsisten dengan glosarium | Usability | Peninjauan salinan teks |
| NFR-COMP-01 | Aplikasi berjalan pada Android 9 ke atas dan iOS 14 ke atas | Compatibility | Uji perangkat |
| NFR-MNT-01 | Kode mengikuti satu panduan gaya yang disepakati dan lolos pemeriksaan linter tanpa galat | Maintainability | Pemeriksaan otomatis pada integrasi |
| NFR-PORT-01 | Konfigurasi lingkungan dipisahkan dari kode sehingga aplikasi dapat dipindah antar lingkungan tanpa mengubah kode | Portability | Peninjauan konfigurasi |

## 11. Matriks keterlacakan

Kolom diisi bertahap seiring artefak lain jadi. Kolom kosong adalah pengingat pekerjaan yang belum selesai, bukan kelalaian. Diperbarui 19 Oktober 2026 mengikuti endpoint di `06_api-contract-draft.md` v0.3. Baris untuk FR yang sudah DEPRECATED dihapus dari matriks ini.

| Kode FR | Use Case | Layar | Endpoint | Test Case |
| --- | --- | --- | --- | --- |
| FR-AUTH-001 | UC-AUTH-01 | Registrasi | `POST /auth/register` | TC-FR-AUTH-001-01 |
| FR-VER-004 | UC-HLP-01, UC-ORD-09B | Cari helper | API-HLP-01 | TC-FR-VER-004-01 |
| FR-VER-007 | UC-VER-01 | Unggah verifikasi | `POST /verifications` | TC-FR-VER-007-01 |
| FR-HLP-001 | UC-HLP-01 | Beranda, Cari helper | API-HLP-01 | TC-FR-HLP-001-01 |
| FR-HLP-002 | UC-HLP-01 | Cari helper | API-HLP-01 | TC-FR-HLP-002-01 |
| FR-HLP-010 | UC-HLP-01 | Detail helper | API-HLP-02 | TC-FR-HLP-010-01 |
| FR-HLP-011 | UC-HLP-01 | Cari helper, Detail helper | API-HLP-01, API-HLP-02 | TC-FR-HLP-011-01 |
| FR-ORD-001 | UC-ORD-09 | Form permintaan pesanan | API-ORD-02 | TC-FR-ORD-001-01 |
| FR-ORD-003D | UC-ORD-09, UC-ORD-09B | Detail helper, Form permintaan, Quote | API-ORD-02, API-ORD-04A, API-ORD-04C, API-ORD-04E, API-ORD-13 | TC-FR-ORD-003D-01 |
| FR-ORD-006 | UC-ORD-09B | Permintaan masuk helper | API-ORD-04A | TC-FR-ORD-006-01 |
| FR-ORD-008 | UC-ORD-04 | Pelacakan | API-ORD-03 | TC-FR-ORD-008-01 |
| FR-ORD-010 | UC-ORD-05 | Pelacakan | API-ORD-08 | TC-FR-ORD-010-01 |
| FR-ORD-011 | UC-ORD-07 | Konfirmasi batal | API-ORD-09A, API-ORD-09 | TC-FR-ORD-011-01 |
| FR-ORD-013 | UC-ORD-03 | Unggah bukti | API-ORD-11, API-ORD-05 | TC-FR-ORD-013-01 |
| FR-ORD-016 | UC-HLP-01, UC-ORD-09 | Cari helper | API-HLP-01, API-ORD-02 | TC-FR-ORD-016-01 |
| FR-ORD-018 | UC-ORD-08 | Ajukan sengketa | API-ORD-12 | TC-FR-ORD-018-01 |
| FR-ORD-021 | UC-ORD-03 | Unggah bukti | API-ORD-11, API-ORD-05 | TC-FR-ORD-021-01 |
| FR-ORD-022 | UC-ORD-09 | Form permintaan Personal | tidak ada, konten statis | TC-FR-ORD-022-01 |
| FR-ORD-023 | UC-ORD-07 | Pekerjaan berjalan helper | API-ORD-09 | TC-FR-ORD-023-01 |
| FR-ORD-024 | UC-ORD-09 | Proses otomatis | API-ORD-04C | TC-FR-ORD-024-01 |
| FR-WLT-002 | UC-ORD-09 | Quote | API-ORD-04C | TC-FR-WLT-002-01 |
| FR-WLT-003 | UC-ORD-05 | Proses otomatis | API-ORD-08 | TC-FR-WLT-003-01 |
| FR-WLT-011 | UC-ORD-09 | Quote | API-ORD-04C | TC-FR-WLT-011-01 |
| FR-WLT-013 | UC-ORD-09 | Quote | API-ORD-04C | TC-FR-WLT-013-01 |
| FR-WLT-014 | UC-ORD-07 | Konfirmasi batal | API-ORD-09 | TC-FR-WLT-014-01 |
| FR-CHT-006 | UC-CHT-02 | Detail helper, Percakapan | API-CHT-02 | TC-FR-CHT-006-01 |
| FR-CHT-007 | UC-CHT-02, UC-CHT-01 | Percakapan | API-CHT-02, API-ORD-02 | TC-FR-CHT-007-01 |
| FR-CHT-008 | UC-CHT-01 | Percakapan | API-HLP-02 | TC-FR-CHT-008-01 |
| FR-CHT-009 | UC-CHT-01 | Percakapan | API-CHT-03 | TC-FR-CHT-009-01 |
| FR-CHT-010 | UC-CHT-02 | Percakapan | API-CHT-02 | TC-FR-CHT-010-01 |
| FR-RTG-001 | UC-RTG-01 | Penilaian | `POST /orders/{id}/rating` | TC-FR-RTG-001-01 |
| FR-ADM-004 | UC-ADM-01 | Panel admin, verifikasi | API-ADM-03, API-ADM-04 | TC-FR-ADM-004-01 |
| FR-ADM-005 | UC-ADM-02 | Panel admin, sengketa | API-ADM-05 | TC-FR-ADM-005-01 |
| FR-ADM-006 | UC-ADM-02 | Panel admin, sengketa | API-ADM-06 | TC-FR-ADM-006-01 |
| FR-ADM-009 | UC-ADM-04 | Panel admin, pengguna | API-ADM-08 | TC-FR-ADM-009-01 |
| FR-ADM-011 | UC-ADM-05 | Panel admin, pesanan | API-ADM-10 | TC-FR-ADM-011-01 |
| FR-ADM-012 | UC-ADM-06 | Panel admin, voucher | API-ADM-11, API-ADM-12, API-ADM-13 | TC-FR-ADM-012-01 |
