# Blameless Postmortem - Insiden Kegagalan Deployment Manual

# Ringkasan Insiden

[Pada simulasi serah-terima aplikasi antara peran Development dan Operations, proses deployment manual mengalami kegagalan sebanyak 2 kali dan memakan waktu 15 menit tetapi aplikasi tida berhasil dijalankan oleh ops. Insiden ini terjadi karena dokumen serah-terima manual (HANDOVER.md) tidak mencantumkan informasi teknis yang lengkap.]

## Kronologi (Timeline)
10:56 Tim Operations menerima artefak (folder src/ dan HANDOVER.md) dari Development.
10:58 Percobaan pertama menjalankan python3 src/app.py gagal dengan pesan error ModuleNotFoundError
11:10  Tim Operations mencoba  pip install flask , namun mengalami kegagalan kedua dengan pesan error externally-managet-environment
11:12 Tim Operations gagal menjalankan aplikasi.

## Dampak (Waktu Terbuang, Jumlah Kegagalan)
Total waktu terbuang: 15 menit (Lead Time manual dari tabel JOB 2)
Jumlah kegagalan: 2 kali percobaan gagal
Jumlah pertanyaan yang seharusnya bisa diajukan ke Dev namun terhalang aturan simulasi: 3 pertanyaan.

## Akar Masalah pada SISTEM (bukan pada orang)

1. Insiden ini berakhir dengan status GAGAL — tim Operations tidak berhasil menjalankan aplikasi dalam batas waktu 15 menit yang ditentukan. Akar masalahnya BUKAN terletak pada kompetensi atau ketelitian individu yang berperan sebagai Operations, melainkan pada kegagalan SISTEM KERJA itu sendiri, dengan rincian sebagai berikut:

2. Tidak ada standar dokumentasi serah-terima (handover) yang baku. Dokumen HANDOVER.md hanya berisi instruksi generik ("Install dependensi", "Jalankan aplikasi") tanpa detail teknis spesifik (nama library, versi Python minimum, kebutuhan virtual environment), sehingga Operations tidak memiliki informasi yang cukup untuk menyelesaikan tugasnya sendiri, seberapa pun kompeten atau teliti orang tersebut.
3. Tidak ada mekanisme verifikasi otomatis sebelum serah-terima. Artefak yang diserahkan sengaja tidak menyertakan requirements.txt, dan tidak ada proses/checklist yang memvalidasi kelengkapan artefak sebelum diserahkan ke Operations. Kegagalan seperti ini seharusnya terdeteksi SEBELUM sampai ke tangan Operations, bukan ditemukan secara reaktif di tengah proses deployment.
4. Tidak ada jalur komunikasi yang memungkinkan validasi cepat. Aturan simulasi (Operations dilarang bertanya langsung ke Developer) merepresentasikan kondisi nyata di banyak organisasi tradisional, di mana Dev dan Ops bekerja dalam silo terpisah tanpa umpan balik cepat (fast feedback loop). Akibatnya, kesalahpahaman atau informasi yang hilang tidak dapat segera diklarifikasi, dan kegagalan terus berlanjut hingga waktu habis.
5. Tidak ada otomasi yang menggantikan langkah manual berulang. Proses seperti pembuatan virtual environment dan instalasi dependensi dikerjakan secara manual oleh manusia, padahal proses ini bersifat repetitif dan rentan terlewat/keliru — pekerjaan semacam ini idealnya diotomasi (baru diterapkan kemudian melalui setup.sh pada JOB 4).

## Tindakan Perbaikan (Action Items) + Penanggung Jawab Peran
Tindakan Perbaikan	Penanggung Jawab Peran
Membuat skrip otomasi (setup.sh) yang menggantikan instruksi manual	Development
Menetapkan requirements.txt sebagai artefak wajib yang harus selalu disertakan	Development
Membuat checklist standar serah-terima (definition of done) sebelum artefak diserahkan	Development & Operations
Menambahkan health check otomatis untuk verifikasi cepat pasca-deployment	Development

## pelajaran yang Diambil

Kegagalan total yang dialami Operations pada JOB 2 — bukan sekadar lambat, tapi benar-benar tidak berhasil dalam batas waktu yang ditentukan — menjadi bukti paling jelas bahwa masalah utama dalam kolaborasi Development dan Operations bukan terletak pada kompetensi individu, melainkan pada RANCANGAN SISTEM KERJA yang tidak mendukung serah-terima yang aman dan dapat diandalkan. Selama proses deployment masih bergantung pada dokumentasi manual yang tidak terverifikasi dan komunikasi yang tidak lancar, kegagalan serupa berpotensi terus berulang, siapa pun personel yang menjalankan peran tersebut.

Fenomena ini adalah contoh nyata dari "Wall of Confusion" yang dibahas dalam teori DevOps: kode "dilempar" melewati batas organisasi tanpa jaminan bahwa pihak penerima memiliki informasi dan lingkungan yang memadai untuk menjalankannya. Insiden ini menegaskan pentingnya prinsip The First Way (Flow) — bahwa setiap langkah manual yang berulang dan rawan galat harus diotomasi, bukan didokumentasikan lalu diserahkan sebagai tanggung jawab manusia untuk menafsirkannya.

Setelah otomasi diterapkan pada JOB 4 melalui skrip setup.sh, seluruh proses yang sebelumnya gagal total kini dapat diselesaikan dalam satu perintah, tanpa perlu pengetahuan teknis mendalam dari pihak Operations mengenai detail dependensi aplikasi. Ini membuktikan bahwa solusi atas kegagalan semacam ini bukan "melatih individu agar lebih teliti", tapi "membangun sistem yang tidak memungkinkan kegagalan akibat informasi yang hilang terjadi sejak awal". Ke depannya, standar otomasi dan verifikasi seperti ini perlu diterapkan sejak awal siklus pengembangan, bukan sebagai perbaikan reaktif setelah insiden serupa terjadi.