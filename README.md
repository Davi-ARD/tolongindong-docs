# Workspace: TolonginDong

Workspace artefak untuk proyek aplikasi mobile marketplace jasa harian (SDG 8: Decent Work and Economic Growth). Dokumen ini adalah pintu masuk. Semua orang di tim yang butuh tahu "artefak SA yang mana yang jadi acuan" harus mulai dari sini.

## 1. Prinsip kerja

Pertama, satu sumber kebenaran per topik. Kalau ada dua tempat yang menyebut daftar Functional Requirement, salah satunya harus dihapus, bukan disinkronkan manual. File `02_requirement-master-list.md` adalah satu-satunya tempat FR dan NFR hidup. SRS versi Word menyalin dari sana saat mau dikumpulkan, bukan sebaliknya.

Kedua, setiap artefak punya ID yang bisa dilacak. Requirement menunjuk ke use case, use case menunjuk ke layar dan endpoint, endpoint menunjuk ke test case. Kalau satu FR berubah, kita bisa tahu dalam hitungan detik siapa yang kena dampak.

Ketiga, asumsi ditulis sebagai asumsi. Selama belum divalidasi ke Mentor, statusnya `ASSUMED`, bukan `CONFIRMED`. Ini yang membedakan dokumen analis dari dokumen karangan.

## 2. Struktur folder

```
tolongindong-docs/
├── README.md                                 <- kamu di sini
├── 01_stakeholder-register-glossary.md       <- siapa yang terlibat, istilah apa yang dipakai
├── 02_requirement-master-list.md             <- FR + NFR + matriks keterlacakan
├── 03_use-case-inventory.md                  <- aktor, use case, inventaris layar, gap prototipe
├── 04_dual-role-transaction-analysis.md      <- analisis dua peran dan validasi transaksi
├── 05_erd-draft.md                           <- ERD dan catatan skema untuk BE
├── 06_api-contract-draft.md                  <- kontrak API untuk BE dan Mobile
├── 07_acceptance-criteria-gherkin.md         <- kriteria penerimaan untuk QA
├── 08_alur-kerja-dan-pemetaan-divisi.md       <- alur kerja SA dan siapa kasih input ke siapa, untuk disebar ke tim
├── 09_dokumentasi-keputusan-minggu-1.md       <- catatan lima keputusan hasil sesi 19 Agustus dan dampaknya
├── 10_notulen-sinkronisasi-erd-final.md      <- 12 jawaban Mentor dan keputusan T-01 sampai T-12 (DEC-14 sampai DEC-24)
├── 11_skenario-pengguna.md                   <- naskah alur pengguna end to end untuk UI/UX dan QA
├── 12_class-diagram.md                       <- class diagram model domain dan service layer
├── 13_use-case-dan-activity-diagram.md       <- 3 use case diagram dan 7 activity diagram aktif, XML draw.io siap tempel
├── 14_perubahan-model-marketplace.md         <- perubahan 19 Oktober: tiga kategori, pilih helper langsung, chat sebelum pesanan (DEC-25 sampai DEC-27)
├── assets/13/                                <- gambar pratinjau dan file .drawio untuk dokumen 13
└── assets/sdd/                               <- sumber .mmd dan .dot serta gambar diagram SDD v0.1 (arsitektur, ERD konseptual, tiga sequence diagram)
```

Struktur penyimpanan tim di Google Drive:

```
/00-Admin            berita acara, notulen, jadwal
/01-SA               isi workspace ini, plus SRS dan SDD versi Word
/02-UIUX             file Figma, export flow, design system
/03-BE               skema database, koleksi Postman, dokumentasi API final
/04-Mobile           dokumen teknis mobile
/05-QA               test plan, test case, laporan bug
/99-Archive          versi lama yang sudah tidak dipakai
```

## 3. Konvensi penamaan

Nama file dokumen resmi mengikuti pola `<JENIS>_<NamaProyek>_v<major>.<minor>.<ekstensi>`, contohnya `SRS_TolonginDong_v1.0.docx` dan `SDD_TolonginDong_v0.3.docx`. Tanggal tidak perlu masuk nama file karena sudah tercatat di version history di dalam dokumen.

Aturan versi: `v0.x` berarti draf yang masih boleh berubah tanpa pemberitahuan, `v1.0` berarti sudah menjadi baseline, kenaikan minor untuk perbaikan redaksi atau penambahan detail, kenaikan major untuk perubahan yang mengubah lingkup atau memaksa divisi lain mengerjakan ulang.

## 4. Konvensi ID artefak

| Jenis artefak | Pola | Contoh |
| --- | --- | --- |
| Functional Requirement | `FR-<MODUL>-<NNN>` | `FR-ORD-004` |
| Non-Functional Requirement | `NFR-<KATEGORI>-<NN>` | `NFR-PERF-02` |
| Use Case | `UC-<MODUL>-<NN>` | `UC-ORD-03` |
| Aktor | `ACT-<NN>` | `ACT-02` |
| Entitas basis data | Diagram: huruf besar tunggal. Tabel fisik: `snake_case` jamak mengikuti Laravel (DEC-16) | `ORDER_OFFER` menjadi `order_offers` |
| Endpoint API | `API-<MODUL>-<NN>` | `API-WLT-05` |
| Test Case | `TC-<KODE FR>-<NN>` | `TC-FR-ORD-004-01` |
| Keputusan desain | `DEC-<NN>` | `DEC-01` |
| Risiko | `RSK-<NN>` | `RSK-03` |

Kode modul yang dipakai: `AUTH` autentikasi dan akun, `VER` verifikasi identitas, `PRF` profil dan alamat, `HLP` pencarian serta pendaftaran helper, `ORD` siklus hidup pesanan termasuk pengajuan sengketa, `WLT` dompet, pembayaran, dan voucher, `CHT` percakapan, `RTG` penilaian satu arah, `NOT` notifikasi, `ADM` panel admin. Kode `DSP` tidak dipakai, sengketa tercakup di `ORD` (pengajuan) dan `ADM` (keputusan).

Nomor urut tidak pernah dipakai ulang. Kalau satu FR dihapus, nomornya dipensiunkan dan ditandai `DEPRECATED`, tidak diberikan ke requirement baru. Ini mencegah kekacauan waktu QA membuka test case lama.

## 5. Status artefak

Setiap baris requirement dan setiap dokumen punya kolom status dengan nilai yang terbatas pada lima ini.

`DRAFT` baru ditulis SA, belum dibaca siapa pun. `REVIEW` sudah dikirim ke divisi terkait dan menunggu tanggapan. `CONFIRMED` sudah disetujui pihak yang berwenang, biasanya Mentor untuk lingkup dan BE untuk kelayakan teknis. `BUILT` sudah diimplementasikan dan masuk build. `DEPRECATED` sudah tidak berlaku tetapi sengaja disimpan sebagai jejak.

Sumber setiap requirement juga ditulis, apakah berasal dari prototipe UI, dari diskusi tim, dari kebutuhan lab, atau dari asumsi SA yang belum divalidasi.

## 6. Definition of Done per artefak

Sebuah artefak SA baru boleh dinyatakan selesai kalau memenuhi semua poin berikut.

Untuk daftar requirement, setiap baris terukur dan bebas kata sifat kabur seperti cepat, mudah, atau ramah pengguna. Kalau menyangkut waktu, harus ada angka dan kondisi jaringannya. Setiap FR punya minimal satu use case dan satu kriteria penerimaan.

Untuk use case scenario, ada pra kondisi, pasca kondisi, skenario utama, dan minimal dua skenario alternatif atau eksepsi. Skenario yang hanya berisi happy path dianggap belum selesai.

Untuk ERD, semua entitas punya kunci primer, semua relasi punya kardinalitas eksplisit, dan setiap kolom uang punya tipe data serta satuan yang jelas. Tidak ada kolom bertipe `float` untuk nominal rupiah.

Untuk kontrak API, ada method, path, header, contoh request, contoh response sukses, dan minimal tiga contoh response gagal dengan kode status yang berbeda.

## 7. Alur kerja mingguan dan pemetaan ke alur SA

Alur kerja SA sembilan langkah dari materi lab dipetakan ke rencana eksekusi seperti berikut.

| Langkah | Aktivitas | Output workspace | Target |
| --- | --- | --- | --- |
| 1 | Kick-off dan identifikasi stakeholder | `01_stakeholder-register-glossary.md` | Selesai |
| 2 | Requirement elicitation | catatan diskusi tim, hasil review prototipe | Selesai |
| 3 | Analysis dan validation | `04_dual-role-transaction-analysis.md` | Selesai |
| 4 | Requirement documentation | `02_requirement-master-list.md` v0.3, SRS bab 1 sampai 3 | Master list v0.3 dan SRS v0.3 selesai 19 Okt 2026, menunggu persetujuan Mentor |
| 5 | Verifikasi ke stakeholder | `10_notulen-sinkronisasi-erd-final.md`, `14_perubahan-model-marketplace.md` | Selesai 28 Sep 2026, perubahan lingkup final diterima 19 Okt 2026 |
| 6 | Modelling | `03_use-case-inventory.md`, `05_erd-draft.md` v0.4, `12_class-diagram.md` v0.2, `13_use-case-dan-activity-diagram.md` v0.2 | Seluruh diagram disesuaikan dengan model marketplace 19 Okt 2026 |
| 7 | Handover ke Design dan Dev | `06_api-contract-draft.md` v0.3, SDD v0.1 bab 1 sampai 4 | SDD v0.1 draf 19 Okt 2026, bab 4 menunggu QA |
| 8 | Monitoring development | log perubahan requirement dan dampaknya | Berjalan |
| 9 | Support UAT | `07_acceptance-criteria-gherkin.md` | Menjelang UAT |

## 8. Matriks handover antar divisi

| Penerima | Yang SA berikan | Yang SA butuhkan balik |
| --- | --- | --- |
| UI/UX Designer | daftar layar, aturan bisnis per layar, kondisi kosong dan kondisi error, aturan validasi form | konfirmasi bahwa semua state punya desain, termasuk state gagal |
| Backend Developer | ERD, kamus data, kontrak API, aturan invarian transaksi | konfirmasi kelayakan teknis dan estimasi, penyesuaian nama field |
| Mobile Developer | alur navigasi, state machine pesanan, daftar kode error dan pesan yang harus ditampilkan | konfirmasi bahwa semua transisi state bisa dipicu dari UI yang ada |
| QA | kriteria penerimaan format Gherkin, daftar edge case, data uji | daftar skenario yang belum tercakup |
| Mentor | SRS, SDD, progres mingguan | persetujuan lingkup dan koreksi format |

## 9. Decision log

Keputusan yang mengubah arah desain dicatat di sini supaya tidak diulang perdebatannya.

| ID | Keputusan | Status | Alasan singkat | Tanggal |
| --- | --- | --- | --- | --- |
| DEC-01 | Satu akun dengan dua kapabilitas, bukan dua akun terpisah | Disetujui | Menghindari duplikasi identitas dan mempermudah rekonsiliasi saldo. Detail di `04_dual-role-transaction-analysis.md` | 12 Agu 2026 |
| DEC-02 | Uang ditahan sistem (escrow) sejak helper menerima pesanan sampai pesanan dikonfirmasi selesai | Disetujui | Ini jawaban atas risiko transaksi dua arah | 12 Agu 2026 |
| DEC-03 | Penilaian dua arah bersifat tertutup sampai kedua pihak mengisi atau tenggat lewat | Digantikan oleh DEC-09 | Dibahas ulang bersama UI/UX 28 Agustus, tim memilih menghapus penilaian dari sisi Helper sama sekali alih alih membangun mekanisme tunda dan sembunyikan | 12 Agu 2026 |
| DEC-04 | Dompet berjalan dengan saldo simulasi, integrasi payment gateway sungguhan dicatat sebagai arah pengembangan lanjutan di luar lingkup lab | Disetujui | Hasil sesi sinkronisasi 19 Agustus, menjawab pertanyaan terbuka nomor satu di `04_dual-role-transaction-analysis.md` | 19 Agu 2026 |
| DEC-05 | Pemesanan memakai model tawar harga dua arah, client mengajukan estimasi harga, beberapa helper dapat mengajukan harga masing masing, client memilih satu | Digantikan oleh DEC-26 | Hasil sesi sinkronisasi 19 Agustus, menggantikan model penerimaan tawaran sederhana yang diusulkan sebelumnya. Detail dampak desain di `09_dokumentasi-keputusan-minggu-1.md` | 19 Agu 2026 |
| DEC-06 | Pelacakan pesanan memakai perubahan status, bukan lokasi GPS langsung, untuk saat ini | Disetujui, dikonfirmasi ulang 28 Agustus tanpa perubahan | Hasil sesi sinkronisasi 19 Agustus, menurunkan beban kerja Mobile dan BE. Layar pelacakan tetap perlu digambar ulang oleh UI/UX sebagai visualisasi tahapan status | 19 Agu 2026 |
| DEC-07 | Mentor lab berperan sebagai admin operasional untuk verifikasi identitas helper dan penyelesaian sengketa, mengakses lewat panel admin sungguhan di dalam sistem | Disetujui | Hasil sesi sinkronisasi 19 Agustus, mekanisme akses dikonfirmasi 28 Agustus. Konsekuensi: modul ADM baru ditambahkan ke `02_requirement-master-list.md`, `05_erd-draft.md`, dan `06_api-contract-draft.md` | 19 Agu 2026 |
| DEC-08 | Besaran potongan platform ditetapkan 10 persen dari nominal yang diterima helper | Disetujui | Dikonfirmasi 28 Agustus. Rekomendasi dan alasannya ada di `09_dokumentasi-keputusan-minggu-1.md` | 19 Agu 2026 |
| DEC-09 | Penilaian dihapus dari dua arah menjadi satu arah, hanya Client menilai Helper. FR-RTG-002 dinyatakan DEPRECATED | Disetujui | Hasil diskusi SA dengan UI/UX 28 Agustus. Menggantikan DEC-03. Konsekuensi: argumen SDG 8 soal "perlindungan dari penilaian sepihak" pada `01_stakeholder-register-glossary.md` bagian 5 ikut direvisi karena mekanisme yang mendasarinya sudah tidak ada | 28 Agu 2026 |
| DEC-10 | Kanal sengketa (FR-ORD-018) dibatasi hanya untuk pesanan berkategori Delivery | Disetujui. Inkonsistensi Food run ditutup lewat DEC-20, lalu gugur bersama Food run lewat DEC-25 | Hasil diskusi SA dengan UI/UX 28 Agustus. Jalur `PRICE_ADJUSTMENT --> DISPUTED` pada Food run dihapus 28 September | 28 Agu 2026 |
| DEC-11 | Teknologi ditetapkan: Backend Laravel (PHP), Mobile Flutter, Database Supabase (lapis PostgreSQL saja, bukan Auth atau REST bawaan), Autentikasi Laravel Sanctum, Storage foto Cloudflare R2 dengan bucket terpisah publik dan privat | Dikonfirmasi (keputusan teknis BE) | Dilaporkan Kai (Backend) lewat `tolongindong-erd-review.md`. Menjawab pertanyaan lama soal stack yang belum ditetapkan | 6 Sep 2026 |
| DEC-12 | Enam dari dua belas pertanyaan terbuka ERD terjawab lewat sesi BE: enkripsi foto KTP tidak perlu, enkripsi nomor telepon tidak perlu, voucher dipotong dari komisi platform, voucher boleh ditumpuk, cooldown verifikasi 2 menit, admin butuh fitur suspend akun | Sebagian digantikan oleh DEC-14 | Poin voucher bertumpuk dan cooldown 2 menit dibatalkan Mentor. Poin lain tetap berlaku dengan penyempurnaan di DEC-14 | 6 Sep 2026 |
| DEC-13 | Enam pertanyaan ERD sisanya (itemisasi order, kardinalitas penyesuaian harga, definisi kategori, rumus tarif jasa, batas alamat, chat sebelum pesanan) mendapat rekomendasi awal SA | Terjawab oleh DEC-14 | Seluruh rekomendasi SA disetujui Mentor, dengan tambahan aturan satu penyesuaian pending dan direct booking tetap memakai quote | 6 Sep 2026 |
| DEC-14 | Mentor menjawab dua belas pertanyaan terbuka ERD: tanpa `ORDER_ITEM`, penyesuaian harga 0..N dengan maksimal satu pending, definisi kategori, `service_fee` flat dan direct booking memakai quote, subsidi voucher maksimal sebesar komisi, satu voucher per order, cooldown verifikasi dihapus, foto KTP tanpa enkripsi tambahan, batas 10 alamat sebagai business rule, chat hanya setelah order, nomor telepon tanpa enkripsi dengan masking, suspend diblokir order aktif | Disetujui. Nomor 10 digantikan DEC-27, nomor 2 gugur bersama talangan (DEC-25) | Rincian di `10_notulen-sinkronisasi-erd-final.md` bagian 2. Menggantikan sebagian DEC-12 dan menjawab DEC-13 | 28 Sep 2026 |
| DEC-15 | Empat konsep uang dipisah: `total_amount`, `service_client_charge`, `held_amount`, `advance_actual`. Voucher hanya mengurangi tagihan client dan komisi bersih, tidak pernah upah helper. Komisi dibulatkan ke bawah | Disetujui (BE, diperluas tim). Sejak DEC-25 `advance_actual` dihapus dan `held_amount` selalu sama dengan `service_client_charge` | Memperbaiki rumus `erd_final.md` yang mencairkan dana melebihi dana tertahan. Notulen T-01 | 28 Sep 2026 |
| DEC-16 | Penyelarasan skema: enum final termasuk `pending_confirmation` dan `awaiting_quote`, tabel `ORDER_ATTACHMENT`, tabel fisik `orders`, pivot kategori helper, status suspend hanya di `user`, label lokasi pengerjaan | Disetujui (BE). Nilai enum broadcast dan talangan dihapus mengikuti DEC-25 dan DEC-26, rincian di `05` bagian 4 | Notulen T-02 sampai T-04 | 28 Sep 2026 |
| DEC-17 | Food run memisahkan upah jasa dari uang belanja. Client menyediakan batas talangan, komisi hanya dari upah, helper dengan `max_advance_limit` di bawah batas pesanan tidak melihat pesanan | Tidak berlaku sejak DEC-25 | Notulen T-05. `scenario.md` ditulis ulang menjadi `11_skenario-pengguna.md` | 28 Sep 2026 |
| DEC-18 | Laundry adalah jasa jemput dan antar ke penyedia laundry dengan talangan. Personal mendapat lima larangan termasuk pekerjaan berbahaya | Sebagian tidak berlaku. Bagian Laundry gugur lewat DEC-25, bagian larangan Personal tetap dan larangan ketiga diperluas | Notulen T-06 | 28 Sep 2026 |
| DEC-19 | Aturan pembatalan final: gratis sampai 120 detik setelah diterima, lalu Rp 5.000, lalu 25 persen harga jasa saat helper berangkat, tidak bisa batal sepihak saat dikerjakan. Voucher kembali kalau gratis, hangus kalau kena penalti. Kompensasi tanpa komisi. Talangan hanya dijamin kalau sah dan dalam batas | Disetujui. Status `searching` dan `pending_confirmation` serta klausul talangan gugur sejak DEC-25 dan DEC-26 | Notulen T-07 | 28 Sep 2026 |
| DEC-20 | Jalur sengketa Food run dihapus. Penolakan penyesuaian talangan mengembalikan pesanan ke `in_progress`, belanja di atas batas tidak dijamin tanpa persetujuan client | Tidak berlaku sejak DEC-25 | Notulen T-08. Menutup inkonsistensi DEC-10 | 28 Sep 2026 |
| DEC-21 | Direct booking: helper memberi quote dalam 300 detik, quote berlaku 300 detik, dana ditahan saat client setuju tanpa konfirmasi 60 detik, tanpa counter-offer, gagal berarti tombol siarkan ke helper lain | Disetujui, diperluas DEC-26 menjadi satu satunya jalur. Tombol siarkan diganti Pilih helper lain | Notulen T-09 | 28 Sep 2026 |
| DEC-22 | Suspend diblokir oleh pesanan `pending_confirmation` sampai `awaiting_confirmation` dan `disputed`. Admin boleh membatalkan pesanan aktif selain `disputed`, dana jasa kembali ke client, talangan sah tetap diganti ke helper, tanpa split | Disetujui. Sejak DEC-25 dan DEC-26 batas bawahnya `accepted` dan klausul talangan gugur | Notulen T-10 | 28 Sep 2026 |
| DEC-23 | Voucher divalidasi keras saat dibuat admin, dicek terhadap quote terpilih, manajemen voucher berprioritas Should dan demo boleh memakai data seed | Disetujui | Notulen T-11 | 28 Sep 2026 |
| DEC-24 | Keputusan sengketa split membagi dana tertahan 50:50 tanpa komisi platform. Foto penjemputan wajib untuk Delivery | Disetujui | Notulen T-12 | 28 Sep 2026 |
| DEC-25 | Kategori Moving, Food run, dan Laundry dihapus. Kategori yang tersisa Delivery, Personal, dan Household | Disetujui, final tanpa diskusi tim lanjutan | Seluruh model talangan ikut hilang. DEC-17 dan DEC-20 tidak berlaku, DEC-18 bagian Laundry gugur. Rincian di `14_perubahan-model-marketplace.md` | 19 Okt 2026 |
| DEC-26 | Pemesanan memakai model marketplace: client memilih langsung helper yang relevan dari daftar helper lalu mengirim permintaan pesanan ke helper itu, tanpa broadcast. Harga tetap dari quote helper sesuai DEC-21 | Disetujui, final tanpa diskusi tim lanjutan. Mekanisme harga quote dipilih 19 Okt 2026 | Menggantikan DEC-05. 11 FR DEPRECATED, endpoint dan status broadcast dihapus. Rincian di `14` | 19 Okt 2026 |
| DEC-27 | Client bisa chat helper sebelum memesan | Disetujui, final tanpa diskusi tim lanjutan | Menggantikan DEC-14 nomor 10. Ruang chat menjadi milik pasangan client dan helper (turunan SA nomor 12) | 19 Okt 2026 |

## 10. Risiko yang sedang dipantau

| ID | Risiko | Dampak | Mitigasi |
| --- | --- | --- | --- |
| RSK-01 | Lingkup terlalu besar untuk durasi lab, terutama fitur peta waktu nyata dan dompet | Tinggi | Tetapkan MoSCoW sejak awal, jadikan pelacakan peta sebagai simulasi status, bukan GPS penuh |
| RSK-02 | Prototipe UI belum punya layar untuk sebagian alur penting seperti pembuatan pesanan dan sengketa, dan sejak 19 Oktober 2026 masih menampilkan enam kategori serta belum punya chat sebelum pesanan | Sedang | Daftar gap sudah disiapkan di `03_use-case-inventory.md` bagian 5 untuk dibahas dengan UI/UX |
| RSK-03 | Uang sungguhan tidak boleh dipakai di lingkungan lab | Sedang | Dompet berjalan dalam mode simulasi dengan saldo dummy, dinyatakan eksplisit di batasan SRS |
| RSK-04 | Turunan SA di `05_erd-draft.md` bagian 7 (delapan dari nomor 1 sampai 11 yang masih berlaku, ditambah nomor 12 sampai 21) belum dikonfirmasi BE, dan BE bisa sudah memulai migrasi dari `erd_final.md` atau v0.3 | Sedang | Kai meninjau bagian 7 dan tabel migrasi di bagian 8 sebelum migrasi berikutnya. `erd_final.md` dan v0.3 dianggap digantikan oleh `05_erd-draft.md` v0.4 |
| RSK-05 | Chat sebelum pesanan membuka peluang client dan helper bertransaksi di luar aplikasi, sehingga platform kehilangan komisi dan client kehilangan perlindungan escrow | Sedang | Nomor telepon tetap tersamar (NFR-SEC-04) dan ruang chat menampilkan pemberitahuan tetap (FR-CHT-009). Deteksi nomor telepon di isi pesan dicatat sebagai pengembangan lanjutan |
| RSK-06 | Helper dibanjiri chat dari client yang tidak berniat memesan | Rendah | Hanya client yang bisa membuka ruang baru, maksimal 20 per hari (FR-CHT-007, FR-CHT-010). Fitur blokir pengguna dicatat sebagai pengembangan lanjutan |
