# Stakeholder Register dan Glosarium

Status dokumen: `CONFIRMED` v0.2, 28 September 2026. Glosarium diperbarui mengikuti DEC-14 nomor 3 dan DEC-15 sampai DEC-21.

## 1. Stakeholder pengguna

Ini pihak yang memakai sistem secara langsung dan menjadi aktor di use case.

| ID | Stakeholder | Peran dalam sistem | Kebutuhan utama | Kekhawatiran utama |
| --- | --- | --- | --- | --- |
| ACT-01 | Client (pemesan) | Membuat permintaan bantuan, membayar, menilai helper | Bantuan datang cepat, harga jelas di depan, orang yang datang terverifikasi | Uang hilang, helper kabur, harga membengkak setelah pesan |
| ACT-02 | Helper (penolong) | Menerima pesanan, mengerjakan, menerima bayaran | Pesanan yang layak, bayaran pasti cair, jarak masuk akal | Sudah kerja tetapi tidak dibayar, dibatalkan sepihak setelah jalan, talangan tidak diganti |
| ACT-03 | Sistem | Pencocokan, penahanan dana, notifikasi, pelepasan dana otomatis | Konsistensi status dan saldo | Kondisi balapan data dan saldo ganda |
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
| Kategori layanan | Enam kategori yang terlihat di prototipe: Food run, Delivery, Moving, Personal, Household, Laundry |
| Food run | Helper membeli barang atas nama client memakai talangan, lalu mengantarkannya. Punya batas talangan, struk, dan penyesuaian talangan |
| Delivery | Helper mengambil barang di satu lokasi lalu mengantarkannya ke lokasi lain. Helper tidak membeli barang. Satu satunya kategori dengan kanal sengketa formal |
| Moving | Helper membantu memindahkan atau mengangkat barang, misalnya pindahan kamar, kos, atau furnitur |
| Personal | Bantuan pribadi non profesional seperti antre, mengambil dokumen, atau menemani keperluan. Tidak mencakup jasa bersertifikasi, mengangkut penumpang, menangani uang tunai pihak lain, aktivitas melanggar hukum, dan pekerjaan berbahaya atau berisiko tinggi |
| Household | Pekerjaan rumah tangga sederhana di lokasi client yang tidak membutuhkan sertifikasi khusus |
| Laundry | Jasa jemput dan antar ke penyedia laundry. Helper membayar laundry dengan talangan, lalu mengantar pakaian kembali. Helper tidak mencuci sendiri |
| Talangan | Uang yang dikeluarkan helper terlebih dahulu untuk pembelian atas nama client, berlaku pada Food run dan Laundry. Diganti penuh di luar upah jasa dan tidak dipotong komisi |
| Batas talangan | Nominal maksimal belanja yang client sediakan dan sistem tahan saat pesanan diterima. Belanja di atasnya hanya dijamin kalau client menyetujui |
| Upah jasa | Harga jasa yang helper tawarkan, tidak termasuk uang belanja. Dasar perhitungan komisi platform |
| Upah bersih | Upah jasa dikurangi komisi platform 10 persen, yang diterima helper |
| Komisi bersih | Komisi platform setelah dipakai menanggung voucher, tidak pernah negatif |
| Tawaran | Harga upah jasa yang helper ajukan pada pesanan broadcast |
| Quote | Harga upah jasa yang helper ajukan pada direct booking, berlaku 300 detik |
| TD-Wallet | Dompet internal aplikasi tempat saldo client dan pendapatan helper disimpan |
| Escrow | Kondisi dana client ditahan sistem, tidak bisa dipakai client dan belum menjadi milik helper |
| Hold | Aksi menahan dana ke escrow |
| Release | Aksi melepas dana escrow menjadi saldo helper, terdiri atas upah bersih dan penggantian talangan |
| Refund | Aksi mengembalikan dana escrow ke saldo client |
| Payout | Penarikan saldo helper ke rekening atau dompet eksternal |
| Broadcast | Penawaran pesanan ke banyak helper yang memenuhi syarat secara serentak |
| Direct booking | Pemesanan langsung ke satu helper tertentu yang dipilih client dari daftar. Helper tetap memberi quote, dan dana ditahan saat client menyetujui quote |
| Ledger | Catatan mutasi saldo yang bersifat tambah saja dan tidak boleh diubah |
| Idempotency key | Penanda unik agar satu permintaan yang dikirim ulang tidak menghasilkan transaksi ganda |
| Verifikasi identitas | Proses pemeriksaan dokumen identitas sebelum akun boleh menjadi helper |
| Sengketa | Kondisi ketika client dan helper berbeda pendapat soal penyelesaian pesanan. Kanal formal hanya untuk Delivery, diputuskan admin dengan pilihan berpihak client, berpihak helper, atau split 50:50 |
| SLA penerimaan | Batas waktu helper merespons: 60 detik untuk konfirmasi setelah dipilih pada broadcast, 300 detik untuk memberi quote pada direct booking |
| Suspend | Penonaktifan akun oleh admin. Hanya bisa dilakukan kalau akun tidak memiliki pesanan aktif atau sedang disengketakan |

## 5. Kaitan dengan SDG 8

Proyek ini mengangkat SDG 8 tentang pekerjaan layak dan pertumbuhan ekonomi. Supaya klaim itu mempunyai justifikasi yang kuat, usulan tiga hal berikut diturunkan menjadi requirement yang bisa diuji, bukan sekadar narasi.

Pertama, transparansi pendapatan. Helper harus bisa melihat rincian berapa yang dia terima dan berapa potongan platform sebelum menerima pesanan, bukan setelah pekerjaan selesai. Ini menjadi `FR-ORD-006` dan `FR-WLT-004`.

Kedua, kepastian pembayaran. Dana ditahan sistem sejak pesanan diterima sehingga helper punya jaminan bahwa client benar benar punya saldo. Ini menjadi `FR-WLT-002` dan pembatalan sepihak setelah helper berangkat tetap memberi kompensasi, diatur di `FR-ORD-011`.

Ketiga, perlindungan dari penilaian sepihak. Poin ini direvisi 28 Agustus 2026 mengikuti `DEC-09`. Argumen awal di sini mengandalkan penilaian dua arah yang tertutup sampai keduanya mengisi, sehingga Helper tidak bisa ditekan dengan ancaman bintang satu. Mekanisme itu sudah dihapus dari lingkup, `FR-RTG-002` dinyatakan DEPRECATED, sehingga argumen ini tidak lagi punya requirement pendukung.

Ini bukan sekadar detail administratif, ini kehilangan satu dari tiga argumen SDG 8 yang tadinya dipakai membenarkan desain sistem. Kalau tim ingin bab pendahuluan SRS tetap mengklaim "perlindungan dari penilaian sepihak" sebagai bagian dari kontribusi SDG 8, klaim itu perlu argumen pengganti, atau dihapus saja dari narasi dan cukup mengandalkan dua poin pertama. Saya tidak mengarang argumen pengganti sendiri di sini karena itu keputusan yang sebaiknya diambil sadar oleh tim, bukan ditutupi diam diam supaya narasinya tetap kelihatan lengkap.

Dua yang pertama tetap berlaku dan sudah masuk daftar requirement.
