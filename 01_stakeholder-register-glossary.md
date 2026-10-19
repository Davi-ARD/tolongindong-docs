# Stakeholder Register dan Glosarium

Status dokumen: `CONFIRMED` v0.3, 19 Oktober 2026. Glosarium diperbarui mengikuti DEC-25 sampai DEC-27 (`14_perubahan-model-marketplace.md`): tiga kategori tersisa, istilah talangan dan broadcast dihapus, istilah permintaan pesanan dan ruang percakapan ditambah. Versi v0.2 (28 September 2026) mengikuti DEC-14 nomor 3 dan DEC-15 sampai DEC-21.

## 1. Stakeholder pengguna

Ini pihak yang memakai sistem secara langsung dan menjadi aktor di use case.

| ID | Stakeholder | Peran dalam sistem | Kebutuhan utama | Kekhawatiran utama |
| --- | --- | --- | --- | --- |
| ACT-01 | Client (pemesan) | Memilih helper, chat sebelum memesan, mengirim permintaan pesanan, menyetujui quote, membayar, menilai helper | Bisa memilih sendiri helper yang cocok, harga jelas sebelum dana ditahan, orang yang datang terverifikasi | Uang hilang, helper kabur, harga berbeda dari yang disepakati di chat |
| ACT-02 | Helper (penolong) | Membalas chat, memberi quote atas permintaan pesanan, mengerjakan, menerima bayaran | Permintaan yang layak, bayaran pasti cair, jarak masuk akal | Sudah kerja tetapi tidak dibayar, dibatalkan sepihak setelah jalan, dihubungi client tanpa niat memesan |
| ACT-03 | Sistem | Penyaringan daftar helper, batas waktu quote, penahanan dana, notifikasi, pelepasan dana otomatis | Konsistensi status dan saldo | Kondisi balapan data dan saldo ganda |
| ACT-04 | Admin operasional | Verifikasi identitas helper, menangani sengketa, menonaktifkan akun, membatalkan pesanan bermasalah, mengelola voucher | Bukti yang cukup untuk memutuskan | Keputusan sengketa tanpa jejak audit |

## 2. Stakeholder proyek

| Pihak | Kepentingan | Yang mereka butuhkan dari SA | Ritme komunikasi |
| --- | --- | --- | --- |
| Mentor | Kelulusan proyek dan kualitas dokumen | SRS dan SDD tepat waktu, lingkup terkendali | Laporan progres mingguan |
| ASE Lab | Kesesuaian dengan format ASE Lab 2026 | Dokumen mengikuti template resmi, tidak ada bagian yang kosong | Sesi asistensi (mungkin) |
| UI/UX Designer | Kejelasan logika sebelum menggambar | Aturan bisnis per layar, daftar state termasuk error | Sebelum sprint desain dimulai |
| Backend Developer | Skema stabil dan kontrak jelas | ERD, kamus data, kontrak API, aturan invarian | Sebelum sprint backend dimulai |
| Mobile Developer | Alur navigasi dan konsumsi API | State machine pesanan, daftar kode error | Setelah kontrak API disepakati |
| QA | Basis pengujian | Kriteria penerimaan Gherkin, daftar edge case | Menjelang integrasi |

## 3. Stakeholder tidak langsung

Penyedia peta, penyedia pembayaran, dan penyedia notifikasi tidak memakai aplikasi tetapi membatasi desain. Untuk konteks lab, ketiganya diperlakukan sebagai dependensi eksternal yang boleh disimulasikan. Keputusan ini harus tertulis di bagian Asumsi dan Dependensi pada SRS supaya penguji tidak menuntut integrasi sungguhan.

## 4. Glosarium

Istilah di bawah ini dipakai konsisten di seluruh dokumen, kode, dan nama tabel. Kalau tim mau mengganti salah satunya, ganti di sini dulu lalu turunkan ke artefak lain.

| Istilah | Definisi |
| --- | --- |
| Client | Pengguna yang membuat pesanan bantuan dan membayar |
| Helper | Pengguna terverifikasi yang menerima dan mengerjakan pesanan |
| Order | Satu unit permintaan bantuan dengan satu client, satu helper, satu kategori, dan satu siklus hidup status |
| Kategori layanan | Tiga kategori sejak 19 Oktober 2026 (DEC-25): Delivery, Personal, Household. Prototipe masih menampilkan enam kategori. Moving, Food run, dan Laundry sudah dihapus dari lingkup |
| Delivery | Helper mengambil barang di satu lokasi lalu mengantarkannya ke lokasi lain. Helper tidak membeli barang. Satu satunya kategori dengan kanal sengketa formal |
| Personal | Bantuan pribadi non profesional seperti antre, mengambil dokumen, atau menemani keperluan. Tidak mencakup jasa bersertifikasi, mengangkut penumpang, membelikan barang atau menangani uang tunai atas nama client, aktivitas melanggar hukum, dan pekerjaan berbahaya atau berisiko tinggi |
| Household | Pekerjaan rumah tangga sederhana di lokasi client yang tidak membutuhkan sertifikasi khusus |
| Upah jasa | Harga jasa yang helper ajukan lewat quote. Dasar perhitungan komisi platform |
| Upah bersih | Upah jasa dikurangi komisi platform 10 persen, yang diterima helper |
| Komisi bersih | Komisi platform setelah dipakai menanggung voucher, tidak pernah negatif |
| Daftar helper | Halaman yang menampilkan helper terverifikasi yang sedang tersedia, menguasai kategori yang client pilih, dan berada dalam radius 10 km dari lokasi aktif client. Titik awal pemesanan sejak 19 Oktober 2026 |
| Permintaan pesanan | Pesanan yang client kirim ke satu helper pilihannya, berstatus `awaiting_quote` sampai helper memberi quote dan client menyetujuinya |
| Quote | Harga upah jasa yang helper ajukan atas permintaan pesanan, dikirim dalam 300 detik sejak permintaan masuk dan berlaku 300 detik sejak dikirim |
| Ruang percakapan | Ruang chat milik satu pasangan client dan helper. Client membukanya sebelum memesan atau otomatis saat mengirim permintaan pesanan, dan ruang yang sama dipakai selama dan sesudah pesanan |
| TD-Wallet | Dompet internal aplikasi tempat saldo client dan pendapatan helper disimpan |
| Escrow | Kondisi dana client ditahan sistem, tidak bisa dipakai client dan belum menjadi milik helper |
| Hold | Aksi menahan dana ke escrow |
| Release | Aksi melepas dana escrow menjadi saldo helper sebesar upah bersih |
| Refund | Aksi mengembalikan dana escrow ke saldo client |
| Payout | Penarikan saldo helper ke rekening atau dompet eksternal |
| Direct booking | Pemesanan langsung ke satu helper tertentu yang dipilih client dari daftar. Sejak 19 Oktober 2026 (DEC-26) ini satu satunya cara memesan, sehingga dokumen lain cukup menyebutnya pemesanan. Broadcast ke banyak helper sudah dihapus |
| Ledger | Catatan mutasi saldo yang bersifat tambah saja dan tidak boleh diubah |
| Idempotency key | Penanda unik agar satu permintaan yang dikirim ulang tidak menghasilkan transaksi ganda |
| Verifikasi identitas | Proses pemeriksaan dokumen identitas sebelum akun boleh menjadi helper |
| Sengketa | Kondisi ketika client dan helper berbeda pendapat soal penyelesaian pesanan. Kanal formal hanya untuk Delivery, diputuskan admin dengan pilihan berpihak client, berpihak helper, atau split 50:50 |
| SLA penerimaan | Batas waktu helper merespons permintaan pesanan, yaitu 300 detik untuk memberi quote atau menolak |
| Suspend | Penonaktifan akun oleh admin. Hanya bisa dilakukan kalau akun tidak memiliki pesanan berstatus `accepted` sampai `awaiting_confirmation` atau `disputed`. Permintaan pesanan yang masih `awaiting_quote` ikut dibatalkan |

## 5. Kaitan dengan SDG 8

Proyek ini mengangkat SDG 8 tentang pekerjaan layak dan pertumbuhan ekonomi. Supaya klaim itu mempunyai justifikasi yang kuat, usulan tiga hal berikut diturunkan menjadi requirement yang bisa diuji, bukan sekadar narasi.

Pertama, transparansi pendapatan. Helper harus bisa melihat rincian berapa yang dia terima dan berapa potongan platform sebelum menerima pesanan, bukan setelah pekerjaan selesai. Ini menjadi `FR-ORD-006` dan `FR-WLT-004`.

Kedua, kepastian pembayaran. Dana ditahan sistem sejak client menyetujui quote sehingga helper punya jaminan bahwa client benar benar punya saldo. Ini menjadi `FR-WLT-002` dan pembatalan sepihak setelah helper berangkat tetap memberi kompensasi, diatur di `FR-ORD-011`.

Ketiga, perlindungan dari penilaian sepihak. Poin ini direvisi 28 Agustus 2026 mengikuti `DEC-09`. Argumen awal di sini mengandalkan penilaian dua arah yang tertutup sampai keduanya mengisi, sehingga Helper tidak bisa ditekan dengan ancaman bintang satu. Mekanisme itu sudah dihapus dari lingkup, `FR-RTG-002` dinyatakan DEPRECATED, sehingga argumen ini tidak lagi punya requirement pendukung.

Ini bukan sekadar detail administratif, ini kehilangan satu dari tiga argumen SDG 8 yang tadinya dipakai membenarkan desain sistem. Kalau tim ingin bab pendahuluan SRS tetap mengklaim "perlindungan dari penilaian sepihak" sebagai bagian dari kontribusi SDG 8, klaim itu perlu argumen pengganti, atau dihapus saja dari narasi dan cukup mengandalkan dua poin pertama. Saya tidak mengarang argumen pengganti sendiri di sini karena itu keputusan yang sebaiknya diambil sadar oleh tim, bukan ditutupi diam diam supaya narasinya tetap kelihatan lengkap.

Dua yang pertama tetap berlaku dan sudah masuk daftar requirement.
